# R15 — Tenant isolation and document permissions

- **Status:** First pass, design-level. Research only, no code. Defensive security research for Officetul Documents. **Written without access to the Officetul code**, so every recommendation adapts to whatever tenancy and permission model Officetul already has. §13 lists what to check when the code is available (D-013).
- **Researched:** 2026-10-20. **All sources accessed 2026-10-20.**
- **Question:** How do we guarantee that one company never sees another company's documents, and that inside a company people see only what they should, in a way a 1–2 person team (D-005) can build and test?
- **Labels:** **Verified fact** (read in the primary source), **Vendor claim**, **Secondary** (news, a third party or a search excerpt only), **Estimate**, **Open question**, **Recommendation** (proposed, not approved). Source IDs `[S#]` point to §15.

**Access caveats.**
- Several vendor pages were read through a fetch tool that summarises pages, so short quotes should be re-checked before they go into any client-facing text.
- OpenAI's incident post returned 403, so it is **Secondary**.
- Atlassian's support-access FAQ renders in the browser and came back empty; only its search excerpt was seen.
- The CSRB report PDF returned 404.
- No official MySQL statement that row-level security is absent was found. AWS states it [S30], and the MySQL privilege levels documented stop at columns, views and routines [S29].
- NPC texts come from R10, which read them from Internet Archive copies [S83].

**Terms** follow [GLOSSARY.md](../GLOSSARY.md): Document, Version, Original file, Derived file, Folder, Link, Inbox item, Trash. Other terms used here:
- **Company:** the tenant.
- **Company admin:** a client's own administrator.
- **Officetul staff:** the vendor's people.

---

## 1. Tenancy models and Laravel packages

### 1.1 The three models

| Model | How it isolates | Strengths | Weaknesses | Label |
|---|---|---|---|---|
| **Single database, `tenant_id` column + global scope** ("pool") | Every tenant table has `tenant_id`; an Eloquent global scope adds `WHERE tenant_id = ?` to model queries | One schema and one migration run; cheap; easy cross-tenant operations (billing, ops) | One missed scope leaks data. Scopes cover Eloquent only, not the query builder. Unique indexes and validation rules must include `tenant_id`. | Mechanism **Verified fact** [S15][S17]; trade-off **Estimate** |
| **Database (or schema) per tenant** ("silo") | Connection switched per request or job; each tenant's rows live in its own database | A forgotten `WHERE` cannot reach another tenant. Per-tenant backup, restore and deletion. A per-tenant DB user is possible. | Migrations × N tenants; connection management; harder reporting; more ops for a 1–2 person team. A wrong *connection* in a job or cache still leaks. | Mechanism **Verified fact** [S16]; trade-off **Estimate** |
| **Bridge / mixed** | Some resources pooled, some siloed (AWS uses "bridge") | Lets a large or regulated client be siloed later | Two code paths | **Verified fact** (term) [S65]; fit **Estimate** |

**Recommendation:** Documents **uses whatever Officetul already uses** (D-010, OQ-008). It does not add a second tenancy mechanism. If Officetul is single-database, Documents adds the fail-closed guard and database-level backstops in §1.3–1.4. If Officetul is database-per-tenant, the same leak paths in §2 still apply to cache, queues, storage and search.

### 1.2 Packages (versions as of 2026-10-20)

| Package | Current state | Laravel 13 | Relevant behaviour | Label |
|---|---|---|---|---|
| **stancl/tenancy** | Latest stable **v3.10.1** (2026-08-05). **v4 is unreleased**: no v4 tag or pre-release. The `master` branch is v4 work, requiring PHP ^8.4 and Laravel ^12\|^13, with docs at v4.tenancyforlaravel.com. Master adds **PostgreSQL** row-level security (RLS) bootstrappers, which are not usable on MySQL. | v3 supports `^13` since v3.10.0 (2026-03-18) | Single-DB mode: `BelongsToTenant` scopes "primary" models. When no tenant is current, models are "**not scoped at all**". Querying a secondary model directly "looks correct, but you might still accidentally access another tenant's comments". `DB` facade queries "won't be scoped". `unique`/`exists` validation is not scoped. | **Verified fact** [S14][S15]. Any v4 paid or sponsor-only early access: **not verified** |
| **spatie/laravel-multitenancy** | **4.2.2** (2026-10-19), PHP ^8.3 | `^12\|^13` | "Bare essentials". Switch tasks: `PrefixCacheTask` sets cache prefix `tenant_id_{key}`; `SwitchTenantDatabaseTask` switches DB. Queues: `queues_are_tenant_aware_by_default`; a tenant-aware job with a missing tenant fails with `CurrentTenantCouldNotBeDeterminedInTenantAwareJob`; a non-tenant-aware job "runs without a current tenant". **Ships no global scope**, so single-DB row scoping is yours to write. | **Verified fact** [S16]; "scoping is yours" is **Estimate** (from the source code and docs) |
| **Laravel itself** | 13.x released 2026-03-17, PHP 8.3–8.5 | — | Global scopes (`#[ScopedBy]`, `addGlobalScope`), removed by `withoutGlobalScope(s)` / `withoutGlobalScopesExcept`. `Context` is "dehydrated" into queued job payloads and "hydrated" when the job runs. `HasUuids` gives UUIDv7 by default; `HasUlids` exists. `scopeBindings()` scopes child route bindings to the parent. | **Verified fact** [S17][S18][S21][S22] |

**Danger in the package defaults (Estimate):** stancl's "not scoped at all when no tenant is current" is convenient for admin tools, but it means a job or command that forgot to set the tenant reads **every** tenant's rows. Documents should fail closed instead (§1.3).

### 1.3 Fail-closed tenant guard for Documents (**Recommendation**)

1. Every Documents model uses one trait, for example `BelongsToCompany`, whose global scope **throws** when no current company is set, rather than returning unscoped rows. Cross-tenant code (ops, billing, migrations) must call an explicit, greppable escape such as `Tenancy::runAsSystem(fn () => …)`, which is logged.
2. On create, the trait sets `tenant_id` from the current company and **refuses** a `tenant_id` supplied by user input.
3. Public identifiers are **ULIDs or UUIDv7**, never auto-increment IDs in URLs. Random IDs are defence in depth only. OWASP: masking or randomising IDs "is generally not sufficient by itself" [S6]; checks are "essential" "even with complex identifiers" [S7]. UUIDv7 embeds creation time, which is a negligible leak (**Estimate**).
4. Route model binding resolves through the scoped model, so another company's ID returns **404, not 403**. Documents routes use `scopeBindings()` where a child (Version) belongs to a parent (Document) [S21].
5. Every model lookup is followed by a policy check (the in-company permission). Tenant scoping and permissions are two separate layers, and both always run.

### 1.4 Database backstops (MySQL has no row-level security)

| Option | What it gives | Cost | Label |
|---|---|---|---|
| **Composite tenant keys**: unique `(tenant_id, id)` on parents; child foreign keys on `(tenant_id, parent_id)` | The database itself refuses a Link, Version or folder grant that points at another company's row | Low; a schema convention | **Recommendation** (standard SQL multi-column foreign keys) |
| **Separate DB users** | The web app's DB user cannot touch ops-only tables; reporting uses a read-only user | Low | Privileges exist at global, database and object (table, column, view, routine) level [S29]. **Verified fact**; use **Recommendation** |
| **Views per tenant** | `CREATE VIEW … WHERE tenant_id = 42` for a reporting user, with `WITH CHECK OPTION` | A view "cannot refer to system variables or user-defined variables" [S28], so one view cannot follow the session's tenant. That means one view per tenant, or a stored-function workaround. Not worth it for the app; possible for a client's BI export later. | **Verified fact** [S28]; fit **Estimate** |
| **Row-level security** | Database-enforced policies | **Not available in MySQL.** AWS: MySQL "doesn't natively support row-level security on tables" [S30]. PostgreSQL has `CREATE POLICY`, but "table owners normally bypass row security" unless `FORCE ROW LEVEL SECURITY` is set [S31]. | **Secondary** [S30] / **Verified fact** [S31] |
| **DB per tenant + per-tenant DB user** | Strongest | High ops cost; only if Officetul already does it | **Estimate** |

**Recommendation:** composite tenant keys and separate DB users in the MVP. No views, no RLS, and no database engine change for Documents alone.

---

## 2. Leak paths and how to close each one

| # | Leak path | What goes wrong | Control | Label |
|---|---|---|---|---|
| 1 | **Raw queries**: `DB::table`, `DB::select`, query builder joins | Global scopes are an Eloquent feature, and the docs document them only for models. stancl says `DB` facade queries "won't be scoped". | Ban them in the Documents namespace (static rule, §4.3). Any allowed exception includes an explicit `tenant_id` and has a test. | **Verified fact** [S15][S17]; "DB::table bypasses scopes" for plain Laravel is **Estimate** (inferred: scopes are model-only) |
| 2 | **`withoutGlobalScopes()`** | Removes the tenant scope too | Banned outside an allowlisted system namespace. Use `withoutGlobalScope(SpecificScope::class)`. | **Verified fact** (method exists) [S17]; ban **Recommendation** |
| 3 | **Relationships and secondary models** | A child model queried directly skips the parent's scope | Every Documents table carries its own `tenant_id` and the trait. Composite foreign keys (§1.4). | **Verified fact** [S15] |
| 4 | **Validation rules** | `unique` / `exists` look across all tenants. `exists:documents,id` lets a user link another company's document if they know its ID. | Always add `->where('tenant_id', …)`, or use a custom `ExistsInCompany` rule | **Verified fact** [S15] |
| 5 | **Queued jobs** | The job runs in a worker with no request. The tenant is missing, or left over from the previous job. | The job carries `tenant_id` (or uses Laravel `Context`, which is dehydrated into the payload [S18], or spatie `TenantAware` [S16]). Its middleware sets the company **and resets it after**. A missing tenant means the job **fails**. | **Verified fact** (mechanisms) [S16][S18]; policy **Recommendation** |
| 6 | **Permission checks inside jobs** | A notification or export job enqueued while the user still had access, and run after access was removed | Re-check permission **when the job runs**, not when it was queued. Matches ASVS 8.3.2 (changes "applied immediately"). | **Verified fact** [S1]; **Recommendation** |
| 7 | **Scheduled commands** (fixity checks D-031, reminders, cleanup) | They loop over all data with no tenant | Iterate companies explicitly: `foreach company → set context → work → forget`. Never run unscoped. | **Recommendation** |
| 8 | **Long-running workers / Octane** | State held in static properties or singletons survives between requests or jobs. GitHub 2021 is the real-world case (§3). | No tenant state in statics. Reset per request and per job. Test with two consecutive jobs for different tenants. | **Verified fact** (incident) [S72]; **Recommendation** |
| 9 | **Cache keys** | The default Laravel prefix is one app-wide string. "Flushing the cache does not respect your configured cache 'prefix'." | All Documents cache keys go through one helper that prefixes `t{tenant}:` (spatie's `PrefixCacheTask` does the same). Never `Cache::flush()` from tenant code. Cache permission results per request only. | **Verified fact** [S19][S16] |
| 10 | **HTTP / CDN caching of logged-in responses** | Steam 2015 and Railway 2026 both cached authenticated pages | Middleware sends `Cache-Control: private, no-store` on every authenticated response. Any CDN is configured to bypass authenticated paths, and the config change is tested before rollout. | **Verified fact** (incidents) [S73][S80]; **Recommendation** |
| 11 | **Session and company switching** | A user in several companies switches company. Stale role or permission relations stay loaded. | On switch: re-check membership, regenerate the session, clear loaded relations. spatie teams mode says you "need to `unset`" the cached `roles`/`permissions` relations after switching. | **Verified fact** [S37]; **Recommendation** |
| 12 | **Search index** | One shared index; a filter is forgotten, or applied after the search | Tenant and permission filter **inside** the query (§6). Meilisearch tenant tokens bake filters in and "only restrict the search endpoint" [S25]. Typesense scoped keys embed `filter_by` that "users will not be able to override" [S27]. | **Verified fact** [S25][S27]; **Recommendation** |
| 13 | **Object storage keys and presigned URLs** | Guessable keys; long-lived URLs. The Microsoft 38 TB case: a SAS token over a whole storage account, valid until 2051. | Keys `tenants/{tenant}/…/{uuid}` (R13 §2). URLs signed **only after** a policy check, valid for minutes, one object each. Never logged. | **Verified fact** [S81]; R13 §6 [S84] |
| 14 | **Derived files** (previews, thumbnails, OCR text) | Same keys and URLs, but often served from a "faster" path that skips the policy | Same controller and policy as the original. Keyed by the Version's UUID, never by the file's SHA-256, because a hash-keyed path reveals that two companies hold the same file (R13 §4.2). | **Recommendation** |
| 15 | **Broadcasting** (live Inbox updates) | A channel name without the tenant; an authorisation callback that returns true | Private channels such as `company.{tenant}.user.{user}`. The callback checks membership. Private channels require the user to be "authenticated and authorized". | **Verified fact** [S20]; naming **Recommendation** |
| 16 | **Exports** (full tenant export D-033, folder export) | Built by a job without tenant context; the file is left at a guessable path | Export job is tenant-aware and writes under the tenant prefix. Short-lived download. Company admin only for full export. A folder export applies the requesting user's permissions **at run time**. | **Recommendation** |
| 17 | **Logs and error tracking** | Titles, OCR text, URLs or other tenants' data in logs or error reports | Log IDs, not content. Never log presigned URLs (R13 §8). Every log line carries `tenant_id`. The log viewer is ops-only. `Context::addHidden()` keeps a value out of logs. | **Verified fact** [S18]; **Recommendation** |
| 18 | **Email and notifications** | A "document filed" email sent to someone who lost access, or to the wrong company's address | Recipients computed and checked at send time. Emails carry a link, not the file. | **Recommendation** |
| 19 | **IDOR** | Changing an ID in a URL or form opens another record (First American, §3) | §1.3 items 3–5 plus the two-tenant tests (§4). ASVS 8.2.2 and CWE-639. | **Verified fact** [S1][S13][S74] |
| 20 | **Support and impersonation tools** | A "log in as" feature or direct DB access by Officetul staff | Replace with approved, time-limited support access (§7). | **Recommendation** |
| 21 | **Integrations and AI features added later** | A new endpoint that skips the tenant layer (Asana's MCP feature, 2025) | Every new entry point (API, webhook, MCP, mobile) goes through the same guard and the same two-tenant test suite | **Secondary** [S79]; **Recommendation** |

---

## 3. Real cross-tenant leaks and lessons

| Incident | What leaked | Root cause class | Label |
|---|---|---|---|
| **First American Financial, 2019** | Krebs reported "approximately 885 million files". Changing a digit in a document URL opened other customers' files without login. An SEC settlement (2021, $487,616) is reported in press. | IDOR, sequential IDs | **Verified fact** [S74]; SEC figure **Secondary** |
| **Steam, Christmas 2015** | "a second caching configuration was deployed that incorrectly cached web traffic for authenticated users"; about 34k users' pages, with billing address and partial card data | Shared cache of logged-in pages | **Verified fact** [S73] |
| **GitHub, March 2021** | A background thread kept a reference to a shared request environment, so a session cookie could go to another user. GitHub "revoked all active user sessions". | Request state reused across users | **Verified fact** [S72] |
| **OpenAI ChatGPT, 20 March 2023** | A redis-py bug returned other users' data. Chat titles were seen by others, and some payment details were exposed (reported as 1.2% of Plus subscribers active in a nine-hour window). | Shared cache / connection state | **Secondary** [S71] (page returned 403) |
| **Microsoft Power Apps portals, 2021** | 38 million records. "If those configurations are not set and the OData feed is enabled, anonymous users can access list data freely." Microsoft then made table permissions on by default. | Public by default | **Verified fact** [S75] |
| **Salesforce Experience Cloud guest users, 2023** | Misconfiguration "allows an unauthenticated user to access records that should only be available after logging in" | Guest or link access set wrong | **Secondary** [S76] |
| **Wiz ChaosDB (Azure Cosmos DB), 2021** | "complete unrestricted access to the accounts and databases of several thousand Microsoft Azure customers" | Cross-tenant bug in a platform feature that was on by default | **Verified fact** [S77] |
| **Microsoft Storm-0558, 2023** | Tokens forged with a consumer signing key were accepted for enterprise mail, "made possible by a validation error in Microsoft code" | Token claims not fully validated | **Verified fact** [S78]; CSRB counts **Secondary** |
| **Microsoft AI researchers' storage token, 2023** | 38 TB. The token granted "full control" over "the entire storage account", valid to 2051. | Over-scoped, long-lived share link | **Verified fact** [S81] |
| **Asana MCP feature, 2025** | A "logic flaw" "exposed data from Asana instances to other MCP users", about 1,000 customers per a spokesperson | New entry point missing the tenant check | **Secondary** [S79] |
| **Railway CDN, 30 March 2026** | A change "accidentally enabled caching on domains that had it disabled"; "most GET responses without explicit cache headers were cached by default" | Cache config change | **Verified fact** [S80] |
| **Cloudflare Cloudbleed, 2017** | Other customers' cookies and tokens leaked, and "some of that data had been cached by search engines" | Leaked data spreads into third-party caches and indexes | **Verified fact** [S82] |

**Not found:** no credible post-mortem of a cross-tenant leak through an app's **own** search index, thumbnails, or a background job running as the wrong tenant. No NPC decision on a SaaS cross-customer exposure was found either. The absence of public reports is **not** evidence that these paths are safe (**Estimate**).

**Lessons (Recommendation):**
1. **Scope by default, fail closed.** Most app-level cases are a missing check (First American, Asana).
2. **IDs are not secrets.** Always authorise after lookup.
3. **Never let a shared cache hold a logged-in response** (Steam, Railway, ChatGPT). Test cache and CDN changes.
4. **Per-request and per-job state only** (GitHub).
5. **Private by default; short, narrow links** (Power Apps, Salesforce, 38 TB token).
6. **Validate every claim in a token or signed link** (Storm-0558): tenant, document, expiry, purpose.
7. **New entry points inherit the same guard and tests** (Asana).
8. **Plan the breach path now.** Our discovery can start the client's 72-hour NPC clock; we committed to tell the client within 24 hours (D-032) [S83].

---

## 4. Testing tenant isolation

### 4.1 What the standards require

| Source | Requirement | Label |
|---|---|---|
| ASVS **5.0.0** (May 2025; still the current stable release) | **8.4.1 (L2)**: "multi-tenant applications use cross-tenant controls to ensure consumer operations will never affect tenants with which they do not have permissions to interact". This is the only tenant requirement in ASVS 5.0.0. | **Verified fact** [S1] |
| ASVS 8.2.2 (L1) | "data-specific access is restricted to consumers with explicit permissions to specific data items to mitigate insecure direct object reference (IDOR) and broken object level authorization (BOLA)" | **Verified fact** [S1] |
| ASVS 8.2.1 (L1), 8.3.1 (L1) | Function-level access needs explicit permissions. Rules are enforced "at a trusted service layer", never in client-side JavaScript; for us that means Inertia props are never the control. | **Verified fact** [S1] |
| ASVS 8.1.1 (L1), 8.1.2 (L2) | **Authorization documentation** defines function-level, data-specific and field-level rules. This note's §5 is the start of that document. | **Verified fact** [S1] |
| ASVS 8.3.2 (L3) | Changes to authorization data are "applied immediately". Otherwise alert and revert, which "would not mitigate information leakage". | **Verified fact** [S1] |
| ASVS 16.3.2 (L2) | "failed authorization attempts are logged" | **Verified fact** [S4] |
| OWASP Top 10:**2025** A01 Broken Access Control | Released (the release-candidate label was removed from the repo on 2025-12-24). "100% of the applications tested were found to have some form of broken access control." Advice: "deny by default"; "Implement access control mechanisms once and reuse them"; "Model access controls should enforce record ownership"; "Log access control failures"; "include functional access control in their unit and integration tests". | **Verified fact** [S5]; release date inferred from the repo history |
| OWASP API Security 2023 API1 (BOLA) | "Write tests to evaluate the vulnerability of the authorization mechanism. Do not deploy changes that make the tests fail." | **Verified fact** [S8] |
| OWASP IDOR cheat sheet / WSTG-ATHZ-04 | Create User A and User B and try B's objects "regardless of whether the object identifier is predictable or unguessable", for read, create, update, delete, export and admin actions. WSTG: "at least two (often more) users". | **Verified fact** [S7][S9] |

### 4.2 Minimal test strategy for a 1–2 person team (**Recommendation**)

Ordered by value for effort. Items 1–5 run in CI and block merges, following the API1 advice [S8].

1. **Two-company fixture.** A seeder creates Company A and Company B with mirrored data: the same folder names, the same document titles, the same file (same SHA-256), the same OCR words, and users with every role. Every isolation test uses it.
2. **Route sweep test.**
   - The test walks `Route::getRoutes()` for the Documents app. For every route with a model parameter, it calls the route as a Company A user with Company B's IDs.
   - It expects **404** and no B data in the body.
   - A new route without coverage fails the test, so coverage cannot silently fall behind.
   - This is the automated form of WSTG-ATHZ-04 [S9].
3. **Architecture tests** (Pest `arch()`):
   - every class in `App\Documents\Models` uses `BelongsToCompany` (`toUseTrait`);
   - the `DB` facade is `->not->toBeUsed()` in Documents, except in an allowlist;
   - jobs implement the tenant-aware interface.

   Pest 5 is current (v5.0.0, 2026-07-24) but needs **PHP ^8.4**. Pest 4 also supports Laravel 13 on PHP 8.3 [S35].
4. **Static analysis** (Larastan v3.13.0 supports Laravel 13 [S34]):
   - `spaze/phpstan-disallowed-calls` (v4.15.0) bans `withoutGlobalScopes`, `DB::table`, `DB::select` and `Model::query()->toBase()` outside allowlisted paths, each with a message explaining why [S33].
   - Whether it catches facade calls like `DB::table()` is **not verified**. If it does not, write a ~50-line custom PHPStan rule [S32].
5. **Policy = list agreement test.**
   - The in-company permission is one module with two faces: `DocumentPolicy::view($user, $doc)` for one item, and `Document::visibleTo($user)` for lists and search.
   - A test generates random folder trees, grants and links. For each user it asserts that the IDs returned by `visibleTo` equal all documents filtered by `can('view')`.
   - This catches the classic bug where the list and the detail page disagree.
6. **Job and command tests.**
   - Dispatch under Company A, run with the context cleared, and assert the job touched only A.
   - A job with no tenant fails.
   - Two jobs for A then B run in one worker process and do not bleed into each other.
7. **Search, preview and storage tests.**
   - Searching as A for a word that exists only in B returns zero, with the same empty state as a word that exists nowhere.
   - Snippets, facet counts and suggestions contain nothing from inaccessible documents.
   - Every stored path starts `tenants/{A}`.
   - Signed URLs expire within the set minutes.
8. **Manual pass before each pilot and major release.**
   - Burp **Autorize** replays a user's traffic with another user's cookie [S10]. Run it with A's admin traffic replayed as B's user and as A's least-privileged user.
   - The OWASP ZAP Access Control add-on is a free alternative [S12]. AuthMatrix [S11] has been inactive since 2024.
9. **Annual external penetration test** with a two-tenant scope. This also serves NPC 2023-06 s.32 vulnerability management [S83].
10. **Production signal.** Log every cross-tenant ID hit (a 404 on an ID that exists in another tenant) and alert on any (R13 §8; ASVS 16.3.2).

---

## 5. Permissions inside a company

### 5.1 How others do it

| Product / model | Folder vs file | "Broken inheritance" | Label |
|---|---|---|---|
| **Odoo Documents 18/19** | "Access is determined by the access rights defined for the folder and file". Viewer, Editor or None for internal users and for "access through link". Per-contact expiry. A shared folder shows its files "excluding any with restricted access". | A file can be stricter than its folder. How inheritance works is not in the official docs; a non-staff forum answer says file-level `access_ids` stop inheritance. How record-linked documents get access is not documented. | **Verified fact** [S38]; inheritance **Secondary** [S39] |
| **SharePoint / OneDrive** | "By default, a file or folder inherits the permissions of the library or folder that contains it". Sharing a file "breaks permission inheritance". | Supported limit **50,000** unique permissions per list or library; recommended **5,000**. Microsoft: "Minimize the number of unique permission scopes", "Share folders instead of individual files". Moving files back does not restore inheritance. | **Verified fact** [S40][S41][S42] |
| **Google Drive** | Folder sharing flows down. "You cannot give someone less access to a specific file than they have to its parent folder." Since 2024 (beta) and 2025 (general availability), **limited-access folders** can be opened only by people added directly. "You can't apply limited access to an individual file". Google is moving items shared less broadly than their folder into limited-access folders. | **Restriction only at folder level** | **Verified fact** [S43][S44]; cut-over date unconfirmed |
| **M-Files** | Named access control lists on objects, plus **automatic permissions** by class or by metadata value | Changing a list that is in use recomputes permissions in the background | **Verified fact** [S45]; recompute **Secondary** |
| **RBAC (spatie/laravel-permission 8.3.2)** | Roles and permissions, with optional teams. The docs send per-record control to Laravel Policies. | n/a | **Verified fact** [S37]; "RBAC only" **Estimate** |
| **ReBAC / Zanzibar** (OpenFGA, SpiceDB) | Relationship tuples, which can model Drive-style folder inheritance (OpenFGA has a Google Drive sample) | OpenFGA is a CNCF **incubating** project (since 28 Oct 2025). SpiceDB is Apache-2.0, and its cloud tier is advertised "for $2/hr*" with conditions not seen. **No official OpenFGA PHP SDK** was found; the main community one was archived on 7 Nov 2025. Keeping tuples in sync is the app's job: "It is the application's responsibility to keep the relationships within SpiceDB up-to-date". | **Verified fact** [S49][S50][S51][S52]; price **Vendor claim** |
| **OWASP view** | "Prefer Attribute and Relationship Based Access Control over RBAC"; "RBAC is poorly suited" to multi-tenancy | — | **Verified fact** [S6] |

**Assessment (Estimate):**
- **ReBAC as a separate service** means another server, data kept in sync both ways (the "dual write" problem [S51]), and no maintained PHP client. That is too much for 1–2 developers and an SME product.
- **Pure RBAC** cannot express "only HR sees the HR folder".
- **Folder access lists with inheritance, evaluated in SQL by Laravel Policies**, is relationship-based access control in miniature. It covers D-020 and is what Odoo, Google and SharePoint users already understand.
- The market lesson from Google and SharePoint is to **restrict at folder level, not per file**.

### 5.2 Recommended model (**Recommendation**)

**Layer 0: Company boundary.** §1.3. Never configurable.

**Layer 1: Officetul role abilities** (whatever Officetul uses: spatie, custom tables or another package):
- `documents.use`: the app appears in the menu.
- `documents.download`: may download **original files**. Without it a user sees page-image previews only (D-033, NPC 2023-06 s.24).
- `documents.export`: bulk or folder export. Full tenant export is company admin only.
- `documents.admin`: settings, document types, link-type settings, permanent delete (D-041).

**Layer 2: Folder grants**, one table: `folder_id`, grantee (role or user), level.

| Level | Allows | Matches |
|---|---|---|
| **View** | See the document, its details, the Official version and the preview | Section 05 §6.4 |
| **Contribute** | View, plus see drafts and history, file from the Inbox, upload new versions as Draft, and move own drafts to Trash | Section 05 §6.4 |
| **Manage** | Contribute, plus mark official, withdraw, restore, Trash and restore anything, and change the folder's grants | D-037, D-041 |

- **Download** is not a folder level. It is the role ability above, intersected with folder access. A later per-folder "no download" switch can only remove, never add.
- **Inheritance:** a subfolder inherits its parent's grants unless it is marked **Restricted**. A restricted folder has its own grant list and ignores the parent's, like Google's limited-access folders.
- Each folder stores `acl_folder_id`, the nearest ancestor (or itself) that holds the grant list. Visibility is one SQL join: `documents → folders → folder_grants WHERE acl_folder_id IN (…)`. Moving a folder recomputes `acl_folder_id` for its subtree.
- **No per-document permissions in the MVP.** To restrict one document, move it into a restricted folder. This follows SharePoint's "minimize unique scopes" and Google's folder-only restriction.

**Layer 3: Link visibility (D-020).**
- Each link type (e.g. "receipt for", "contract of") has **Grants view** (off by default).
- When it is on, a user who may open the linked Officetul record gets **View** on the document. They never get Contribute or Manage, and they download only if their role has `documents.download`.
- Record access is decided by Officetul's **own** policy for that record (**Open question**: how Officetul decides who sees an expense, §13).

**"Why can I see this?" panel.** It lists each path that grants access. Examples:
- "Folder *Finance › Payables* gives the **Accounting** role *View*".
- "Linked as *receipt for* Expense #881, which you can open; link type *receipt for* shares visibility".

Company admins also get a **"Who can see this?"** view, built from the same function.

**Company admins and confidential folders.** See Decision 4.
- The proposal: a folder can be marked **Confidential**, a restricted folder whose contents company admins do **not** see by default.
- Admins still see that it exists, its name and its grant list, so it is never orphaned.
- An admin can add themselves; that is logged and notifies the folder's Manage holders.
- **Officetul staff never see content without approved support access (§7).**

**External sharing links** are next release (D-014), with these rules (**Recommendation**):
- one document per link;
- **expiry required** (default 7 days, maximum set by the company admin);
- optional password, stored hashed;
- "preview only" option;
- every open logged;
- revocable.

Comparable controls exist elsewhere:
- SharePoint lets admins "require all Anyone links to expire" [S53].
- Box offers passwords, expiry and download restriction on paid plans [S54].
- Dropbox offers password, expiry and disabled downloads on paid plans [S55].
- Google Drive can block download for viewers [S43].

**Watermarking:** none of the link documentation read describes a watermark control, so its value is unverified. Treat it as **later**, and possibly only on page-image previews.

### 5.3 Why not a separate authorization service

| Option | Build effort | Ops | Fit | Label |
|---|---|---|---|---|
| Laravel Policies + SQL access lists (above) | Low–medium | None new | Covers D-020; one code base | **Estimate** |
| spatie/laravel-permission alone | Low | None | Function-level only; still needs folder access lists | **Estimate** |
| OpenFGA / SpiceDB | High (modelling, sync, no maintained PHP client) | One more stateful service | Worth it only with many sharing patterns across many apps | **Estimate** [S51][S52] |

---

## 6. Permission-aware search and previews (D-026 item 6)

| Approach | How | Problem | Label |
|---|---|---|---|
| **Security trimming (the goal)** | Microsoft Search: "only the content that a user has permission to see can appear in search results" | — | **Verified fact** [S46] |
| **Filter after search** | Search everything, then drop rows the user can't see | Leaks through counts, facets, paging ("page 2 of 9") and timing; snippets already built | **Recommendation**: never |
| **Index-time access list per document** (user IDs in the index) | Each document carries its allowed users or roles | Goes stale on every permission change and forces re-indexing. SharePoint Server once showed wrong results "until a full crawl occurred", and now uses a 15-minute default for picking up security changes. | **Verified fact** [S42] |
| **Query-time filter on stable keys** (recommended) | The index stores `tenant_id`, `acl_folder_id` and `document_id`. Laravel computes the user's accessible `acl_folder_id` list plus link-granted document IDs, and passes them **inside** the query. | Long filter lists for users in very many folders (**Estimate**: fine at SME scale). Permission changes take effect on the next query (ASVS 8.3.2). | **Recommendation** |

**Engine notes:**
- **MySQL FULLTEXT:** add `tenant_id = ?` and the access-list join in the same `WHERE` as `MATCH … AGAINST` (**Estimate**, standard SQL).
- **Meilisearch:** tenant tokens are short-lived JWTs whose search rules "automatically apply to every search request". The PHP SDK's `generateTenantToken` signs them with the parent key and takes `expiresAt` [S25][S26].
- **Typesense:** scoped keys embed `filter_by` [S27].
- **Elastic and Algolia:** offer role query filters and secured keys [S47][S48].
- Laravel Scout 13.x docs do not mention tenant tokens [S24].
- **Recommendation:** in the MVP, the browser never talks to the search engine. Laravel proxies every search and adds the filter, with no tokens to leak and no window where a token outlives a revoked permission.

**Do not leak (Recommendation):**
1. **Titles and snippets** are built only from the filtered result set. Highlighting reads OCR text of permitted documents only.
2. **Counts, facets and "N results"** come from the same filtered query.
3. **Empty states are identical** for "nothing matches" and "matches exist but are hidden".
4. **Suggestions, typeahead and "recent"** come from the filtered set and are per user.
5. **The panel of linked documents on an Officetul record** (e.g. Expense #881) lists and counts only documents the viewer may see. See Decision 6 for hints.
6. **The duplicate warning (D-042)** names the existing document only if the uploader can see it (Decision 6).
7. **The activity log and the "Expenses without an invoice or receipt" report (D-023)** run with the viewer's permissions.
8. **Previews and thumbnails** go through the same policy as the original. They are served `private, no-store` from the user-content domain (D-051), keyed by Version UUID, with no public bucket and no CDN cache shared across users.
9. **Tenant-wide statistics** (storage used, document count) are shown to company admins only.

---

## 7. Officetul staff support access (D-033)

| Vendor | Pattern | Time limit | Label |
|---|---|---|---|
| **Microsoft Customer Lockbox** | "Microsoft can't access your content to do service operations without your explicit approval". Engineer actions are audited as "Microsoft Operator". | A request expires if not approved "within 12 hours". Access lasts a maximum of "4 hours", "after which the access is automatically revoked". Included in E5 or available as an add-on. | **Verified fact** [S57] |
| **Google Access Transparency / Access Approval** | Transparency logs staff access with "the reason for the action". Approval requires "approval from an authorized customer administrator". Approval needs Transparency enabled. Outage requests can be auto-approved and are logged as such. | The approver can set an expiry. An unapproved request is dismissed after 14 days. Revocable "at any time". | **Verified fact** [S56] |
| **Atlassian Cloud** | Support "won't access your data unless you explicitly grant us that permission". Grants are approved on the ticket or in the admin console. | Duration not found in official docs. A community article says "within 24 hours after ticket closure". A public bug says grants are missing from the org audit log. | **Verified fact** (principle) [S58]; rest **Secondary** |
| **Salesforce** | The user grants login access from an "Access Duration" picklist | "the maximum period for granting access is 1 year" | **Verified fact** [S59] |
| **AWS break-glass** | Emergency access process. "Break-glass procedures can be considered a backdoor", so alert on use and rotate the credentials after. | — | **Verified fact** [S60] |

**Recommendation: support access in the MVP.**
1. **Request.** An Officetul staff member requests access in the ops console, giving the ticket number, a reason and a scope: settings and metadata only, one folder, or the whole company (read-only).
2. **Approve.** A company admin approves in-app (re-authentication plus MFA). Unanswered requests expire after **12 h**.
3. **Grant.**
   - Read-only by default, with **no original downloads** unless the scope says so.
   - Default **4 h**, maximum 24 h, auto-revoked, and revocable by the admin at any time.
   - The staff member must use MFA, preferably a passkey.
4. **Log.** Every staff action is logged as "Officetul Support – {name}" in the company's activity log, which the company admin can see. This avoids Atlassian's audit gap.
5. **Break-glass** is only for a security incident or data-corruption emergency when no company admin answers:
   - two Officetul staff approve;
   - access lasts at most 1 h;
   - an alert is raised;
   - the company admins are told within **24 h** (matching D-032), with the log extract;
   - the event is recorded in the incident register.
6. **No ad-hoc production database access.** Databases are reached only through approved, logged tooling (NPC 2023-06 s.12 [S83]). Staff hold security clearances under the contract (s.13–14).
7. Any existing "log in as user" feature in Officetul is either removed or wrapped in steps 1–4 (§13).

---

## 8. Encryption, MFA and sessions

### 8.1 Encryption at rest

| Layer | Fact | Label |
|---|---|---|
| S3 default | "Starting January 5, 2023, all new object uploads to Amazon S3 are automatically encrypted at no additional cost" (SSE-S3) | **Verified fact** [S62] |
| SSE-KMS + Bucket Keys | Bucket Keys "can reduce AWS KMS request costs by up to 99 percent". A Bucket Key is shared only among objects with the same KMS key. Audit events show the bucket, not the object. | **Verified fact** [S63] |
| KMS price | "$1/month (prorated hourly)" per key. The first and second rotations add $1/month each, then the price is capped (so at most $3/month per key). Free tier 20,000 requests a month. Symmetric requests $0.03 per 10,000, seen only in the page's worked example. No Singapore-specific price is shown. | **Verified fact** [S61]; request rate partly verified; Region price **Open question** |
| KMS quota | Up to 100,000 customer managed keys per Region per account (can be raised) | **Verified fact** [S64] |
| RDS MySQL | Encrypted with a KMS key (storage, logs, backups, snapshots), but "You can only encrypt an Amazon RDS DB instance when you create it" | **Verified fact** [S69] |
| Self-managed MySQL 8.4 | InnoDB data-at-rest encryption needs a keyring. Community has `component_keyring_file`, which is "not intended as a regulatory compliance solution". The KMS, KMIP and Vault keyrings are Enterprise. "If the master encryption key is lost, data … is unrecoverable." | **Verified fact** [S68] |
| Laravel app-level | `Crypt` uses AES-256-CBC with a MAC. `APP_PREVIOUS_KEYS` allows rotation. With `encrypted` casts "you will not be able to query or search encrypted attribute values". | **Verified fact** [S67] |
| AWS on per-tenant keys | 2025 Architecture Blog: "One KMS key per tenant, maintained in a central AWS account" when customers need dedicated keys or BYOK, warning of "growing operational overhead". The 2020 tenant-isolation whitepaper is now "for historical reference only". | **Verified fact** [S66][S65] |

**Per-tenant keys: worth it for the MVP? No (Recommendation).**
- **Cost (Estimate):** at 100 companies, 100 keys × $1–3 = **$100–300 a month** in key charges, plus requests. Bucket Keys cut the request share. At local-DC hosting there is no managed KMS, which means running something like Vault (more ops).
- **Protection:** per-tenant keys **do not stop the commonest leak class**. An app bug in §2 decrypts with whichever key the app thinks is current. They help mainly against a raw bucket or backup copy leaking, and they allow "crypto-shredding" at contract end and BYOK for a demanding client.
- **MVP:** encrypted storage (SSE-S3 or SSE-KMS with one key), an encrypted database and backups (RDS encryption must be chosen at creation), and TLS everywhere, including app to DB, Redis and search when they cross a network. Encrypted export bundles (NPC 2023-06 s.25) [S83]. App-level `encrypted` casts only for secrets such as integration tokens, never for searchable fields.
- **Later:** per-tenant KMS keys as an option for clients who ask, if hosting is AWS.

### 8.2 MFA

| Fact | Label |
|---|---|
| Laravel Fortify (v1.41.0, Laravel 13) supports TOTP two-factor and `password.confirm`, and since **v1.37.0 (2026-04-28) passkeys**: "Fortify supports passkey authentication using WebAuthn". The 13.x starter kits enable two-factor and passkeys. | **Verified fact** [S36] |
| ASVS 6.3.3 (L2): MFA (or a combination of factors) to access the application. Phishing-resistant hardware factor at L3. TOTP rules: 6.5.1 single use, 6.5.5 30-second lifetime. 6.6.1: SMS only alongside stronger methods. | **Verified fact** [S3] |
| NPC 2023-06 s.16: "multifactor authentication or secure encrypted links" for access to sensitive personal information | **Verified fact** [S83] |

**Recommendation:**
- Use Officetul's existing login. If it is Fortify or a starter kit, enable TOTP and passkeys.
- Add a **company setting "Require two-factor for everyone"** enforced by middleware. Whether Fortify enforces this per company was **not verified**; assume it must be built.
- **Mandatory MFA** for Officetul staff (D-033) and, proposed, for company admins (Decision 8).
- No SMS codes.
- Re-authenticate (`password.confirm`) before:
  - permanent delete;
  - permission changes;
  - exports;
  - approving support access;
  - changing MFA.

  This is ASVS 7.5.1 [S2].

### 8.3 Sessions

| Fact | Label |
|---|---|
| NIST SP 800-63B-4 (final, July 2025), AAL2: overall reauthentication timeout "SHOULD be no more than 24 hours"; inactivity "SHOULD be no more than 1 hour". AAL3: 12 hours / 15 minutes. | **Verified fact** [S70] |
| ASVS 7.3.1 and 7.3.2 (L2): an inactivity timeout and an absolute maximum lifetime, per documented risk decisions. 7.4.2: end sessions when an account is disabled. 7.4.3: offer to end other sessions after a credential change. 7.4.5: admins can end sessions. 7.5.2: users can view and end their sessions. | **Verified fact** [S2] |
| Laravel skeleton: `SESSION_LIFETIME` default **120** minutes idle; `expire_on_close` false. `Auth::logoutOtherDevices($password)`. An absolute lifetime is not a skeleton setting. | **Verified fact** [S23]; "no absolute lifetime" **Estimate** |

**Recommendation:**
- **Idle timeout 60 min** by default; a company can set 15–120.
- **Absolute lifetime 12 h**, one working day, stricter than AAL2's 24 h.
- Removing a user from a company or disabling them ends their sessions at once.
- A "your active sessions" page.

---

## 9. Philippine context (brief)

| Duty | Source | Design consequence | Label |
|---|---|---|---|
| Access control policy, "need to know"; security clearance; PIP contract carries an equivalent | NPC Circular 2023-06 s.13 | Folder grants (§5); staff clearances; support access (§7) | **Verified fact** [S83] |
| PIP access governed by "strict procedures contained in formal contracts" | 2023-06 s.14 | Support-access terms go in the data processing addendum (D-032) | **Verified fact** [S83] |
| Databases accessed or modified only through authorised software | 2023-06 s.12 | No ad-hoc production DB access | **Verified fact** [S83] |
| MFA or secure encrypted links for access to sensitive personal information, privileged information or high volumes | 2023-06 s.16 | §8.2 | **Verified fact** [S83] |
| Stop copying or printing into productivity software where reasonable | 2023-06 s.24 | `documents.download` / `documents.export` role abilities | **Verified fact** [S83] |
| Security and authentication logs kept longer; log backups | 2023-06 s.29 | Permission changes, support access and authz failures go to the security log | **Verified fact** [S83] |
| Encryption in storage and transit | IRR s.28(g); 16-03 s.7 | §8.1 | **Verified fact** [S83] |
| Breach: our discovery may start the client's 72-hour clock | 16-03 s.17(A); IRR s.38(a) | A cross-tenant leak is likely notifiable (scanned IDs, tax returns); 24-hour notice to clients (D-032) | **Verified fact** [S83] |
| Processor duties: act only on instructions; no sub-processor without instruction | IRR s.44 | Search, KMS and hosting vendors on the sub-processor list | **Verified fact** [S83] |

**ISO note (brief):**
- These controls map to ISO/IEC 27001:2022 Annex A as R03 lists them [S85]:
  - A.5.15 / A.5.18 / A.8.3 (access control, access rights, information access restriction);
  - A.8.5 (secure authentication);
  - A.8.15 (logging);
  - A.8.24 (cryptography);
  - A.8.12 (data leakage prevention: download and share limits).
- The ISO goal itself belongs to the senior partner (D-028).

---

## 10. Recommendations summary

| MVP | Next release / later |
|---|---|
| Reuse Officetul's tenancy model; fail-closed `BelongsToCompany` guard; explicit logged system escape | Silo option (DB per tenant) for a large or regulated client, if Officetul's model allows |
| ULID/UUIDv7 public IDs; 404 for other-tenant IDs; `scopeBindings` | — |
| Composite `(tenant_id, id)` keys and foreign keys; separate DB users | Per-tenant reporting views for client BI exports |
| Tenant-aware jobs (fail if missing), per-company scheduled loops, cache key helper, `private, no-store`, tenant-named broadcast channels | — |
| Two-company fixture; route sweep; Pest arch rules; disallowed-calls rules; policy = list agreement test; job, search and storage tests; manual Autorize/ZAP pass before pilot | Annual external penetration test (two-tenant scope); continuous anomaly alerts |
| Role abilities (`use`, `download`, `export`, `admin`) + folder grants View / Contribute / Manage; inheritance with Restricted folders; no per-document access lists | Per-folder "no download" switch; access review report for company admins |
| Link types with "Grants view" (off by default); "Why can I see this?" / "Who can see this?" | — |
| Confidential folders (Decision 4) | — |
| Search proxied through Laravel; query-time tenant + access filter; no hints about hidden documents | Meilisearch/Typesense with short-lived tenant tokens if search moves to the browser |
| Support access: request → company admin approval → 4 h read-only → logged → auto-revoke; break-glass with two-person rule and 24 h notice | Client-visible support-access report in the data processing addendum annex |
| SSE storage + encrypted DB/backups + TLS everywhere; encrypted exports | Per-tenant KMS keys / BYOK on request (AWS) |
| TOTP + passkeys; company "require 2FA"; mandatory for Officetul staff and company admins; 60 min idle / 12 h absolute | Phishing-resistant MFA required for admins |
| — | External share links: required expiry, optional password, preview-only, logged, revocable (D-014); watermark later |

---

## 11. Decisions for the owner

Each is worded so that "yes" accepts the recommendation.

1. **Use Officetul's existing tenancy, plus a fail-closed guard in Documents?**
   - *Recommended:* yes. Documents adopts whatever Officetul does, and adds a guard that **stops with an error** when the company is unknown, instead of showing everything.
   - *Example:* a nightly fixity-check job forgets to say which company it is working for. With the guard, the job fails and ops get an alert. Without it, the job could read every company's documents and write results into the wrong log.

2. **Folder grants with three levels (View, Contribute, Manage), subfolders inheriting, and "Restricted" folders, with no per-document permissions in the MVP?**
   - *Recommended:* yes.
   - *Example:* Maria in Accounting has View on *Finance*, so they see everything under *Finance › Payables*. The CFO marks *Finance › Payroll* as Restricted and grants only the Payroll role. Maria no longer sees payroll slips there.
   - If one sensitive contract must be hidden, someone with Manage moves it into a restricted folder, rather than setting a special rule on that one document that nobody remembers later.

3. **Downloading originals is a role ability, separate from seeing a document?**
   - *Recommended:* yes (D-033).
   - *Example:* Jun in Sales can open and read the preview of a signed customer contract in *Sales › Contracts*. Their role lacks **Download**, so they cannot save the original PDF to their laptop. Ana in Legal, whose role has Download, can.

4. **"Confidential" folders whose contents company admins don't see by default?**
   - *Recommended:* yes. The admin sees that the folder exists and who has access, and can add themselves, but that is **logged and the folder's managers are notified**.
   - *Example:* HR keeps *HR › Disciplinary cases* confidential. The company admin, who also runs IT, can't browse it. If HR leaves the company, the admin grants themselves access to reassign it, and the HR head gets a notice.

5. **Record links give visibility only for link types switched on, and only as View?**
   - *Recommended:* yes. All link types start off. The company admin may switch on chosen types such as "receipt for".
   - *Example:* when "receipt for" is on, Ben, who approves expenses, opens Expense #881 and sees its receipt even though the receipt sits in *Finance › Receipts*, a folder Ben has no grant on. The document says "You can see this because it is the receipt for Expense #881". Ben cannot edit it and cannot download it unless their role allows.

6. **No hints at all about documents a user can't open (strict D-026 item 6)?**
   - *Recommended:* yes. This covers search, counts, record panels and duplicate warnings. A duplicate that lands in a folder the uploader can't see is **flagged to that folder's managers** instead of being revealed to the uploader.
   - *Example:* Carla scans a resignation letter that HR already filed in a confidential folder. Carla gets no "already exists" warning naming HR's document. The HR head gets "a duplicate was filed in *General › Inbox filings*" and can tidy it up.

7. **Officetul staff see client documents only with approval from the client's company admin, time-limited and logged?**
   - *Recommended:* yes. Read-only by default, up to 4 hours, every action shown in the client's activity log. Emergency "break-glass" needs two Officetul staff, lasts 1 hour, and the client is told within 24 hours.
   - *Example:* a client reports that search can't find invoices. Our support person requests "metadata only, *Finance* folder, 4 hours". The client's admin approves on their phone. The client later sees exactly what support looked at.

8. **MFA required for company admins as well as Officetul staff, and offered to everyone, with a company switch to require it for all?**
   - *Recommended:* yes. Use authenticator-app codes or passkeys, not SMS.
   - *Example:* a law-firm client turns on "Require two-factor for everyone", so every associate must set up an authenticator app before opening Documents. A small trading company leaves it optional except for its admin.

9. **Sessions end after 60 minutes idle and 12 hours in total, with each company able to tighten the idle time?**
   - *Recommended:* yes.
   - *Example:* Maria leaves a payroll folder open at lunch. After an hour of no activity, the next click asks them to log in again. Even if active all day, they log in again the next morning.

10. **No per-company encryption keys in the MVP; offer them later only if a client asks?**
    - *Recommended:* yes. All storage, database and backups are encrypted anyway.
    - Separate keys cost money ($1–3 per company per month in AWS key charges alone) and **do not stop** the most common leak, a software mistake, because the app unlocks whichever key it thinks is current.
    - *Example:* a large law firm demands its own key. We can add it for that client later without changing the design.

11. **The two-company isolation tests and the code rules must pass before any change ships?**
    - *Recommended:* yes. A failed test blocks the release, and a manual two-company check runs before each pilot.
    - *Example:* a developer adds a "recent documents" widget using a quick database query. The automated check rejects the query because it bypasses the company filter, before any client sees it.

---

## 12. Open questions

1. **Officetul's tenancy model** (OQ-008, questionnaire Q1–Q3): single DB with `company_id`, DB per tenant, or a package? Can a user belong to several companies? Accounting firms serving several clients make this likely.
2. **How Officetul decides who may open a record** (an expense, a project), which Layer 3 link visibility depends on. Role-only, per-record ownership, or branch/department?
3. Does Officetul have **impersonation** ("log in as") or direct production DB habits that support access must replace?
4. **Search engine choice** (MySQL FULLTEXT vs Meilisearch/Typesense) is outside R15. The filter design in §6 works with any of them.
5. Does `spaze/phpstan-disallowed-calls` catch facade calls like `DB::table()`, or is a custom rule needed (§4.2 item 4)?
6. **Hosting:** AWS Singapore (KMS, RDS encryption at creation) or a local DC (no managed KMS; MySQL Community keyring limits)? This is the same open question as R13 §12.1.
7. KMS price in **ap-southeast-1**, and the standalone symmetric request price (only seen in a worked example).
8. Does Fortify, as configured in Officetul, allow **per-company enforcement** of 2FA, or must middleware be written?
9. The "Expenses without an invoice or receipt" report (D-023): if a receipt exists in a folder the viewer can't see, the report will say "missing" to that viewer. Acceptable, or restrict the report to roles that see all finance folders?
10. Lawyer (D-027): should support-access terms (approval, duration, break-glass notice) be written into the data processing addendum? Does break-glass access without prior approval need the client's advance consent in the contract?
11. Google's exact cut-over date for retiring per-file restrictions, and whether Odoo files inherit folder access, were not confirmed. Both only inform the design.

---

## 13. Check in the Officetul code (when access is granted, D-013)

1. **Tenancy:**
   - Which mechanism: package, version, column name (`tenant_id`/`company_id`).
   - What happens when no tenant is set: scoped, unscoped or exception?
   - How the tenant is identified per request: subdomain, session or user.
2. **Unscoped access:** grep for `DB::table`, `DB::select`, `DB::raw`, `withoutGlobalScope(s)`, `->toBase()`, raw `whereRaw` on tenant tables, and `Model::all()` in commands.
3. **Unique indexes and `unique`/`exists` validation rules** missing `tenant_id`.
4. **ID format in URLs:** auto-increment vs UUID/ULID. Route model binding, and whether bindings are tenant-scoped.
5. **Queues:**
   - Driver.
   - How jobs learn their tenant (payload, `Context`, package).
   - Whether workers run Octane or long-lived processes with static state.
   - The `retry_after` / timeout settings (R13 §4.2).
6. **Scheduled commands:** do they loop per tenant?
7. **Cache:** store, prefix, any per-tenant prefixing, any `Cache::flush()` calls. Any page or HTTP caching, CDN or reverse proxy in front of authenticated pages.
8. **Sessions:** driver, `SESSION_LIFETIME`, company switching logic, logout of other devices.
9. **Auth:**
   - Login package (Fortify, Breeze, Jetstream, custom) and version.
   - Whether 2FA or passkeys exist.
   - Password confirmation for sensitive actions.
10. **Roles and permissions:**
    - Package (spatie version, teams mode on?) or custom.
    - Whether client admins create roles.
    - Permission caching and its invalidation.
11. **Record-level access** in existing apps (expenses, projects): policies, ownership, departments.
12. **Storage:** disk config, key/path pattern, whether paths include the tenant, `temporaryUrl` usage and expiry, public disks.
13. **Broadcasting:** channels and their authorisation callbacks.
14. **Search:** Scout or engine in use, and how results are filtered.
15. **Logging and error tracking:** what is logged (bodies? URLs?), tenant tagging, who can read logs.
16. **Exports and reports:** how existing exports are built and where the files go.
17. **Impersonation or "login as"** packages; ops scripts that query production directly.
18. **Tests:** is there a test suite? Pest or PHPUnit version (Pest 5 needs PHP 8.4)? Any multi-tenant tests? CI?
19. **PHP version** (8.3 vs 8.4 affects Pest 5 and stancl v4).
20. **Database:** MySQL version, encryption at rest (RDS or self-managed keyring), DB users and their privileges.

---

## 14. Statement of limits

This note does not choose the search engine, the hosting or the data model. It sets the rules those choices must satisfy. Legal points are research, not legal advice; the lawyer reviews them under D-027.

---

## 15. Sources (all accessed 2026-10-20)

| ID | Source | URL | Type |
|---|---|---|---|
| S1 | OWASP ASVS 5.0.0, V8 Authorization (tag `v5.0.0_release`) | https://raw.githubusercontent.com/OWASP/ASVS/v5.0.0_release/5.0/en/0x17-V8-Authorization.md | Primary |
| S2 | OWASP ASVS 5.0.0, V7 Session Management | https://raw.githubusercontent.com/OWASP/ASVS/v5.0.0_release/5.0/en/0x16-V7-Session-Management.md | Primary |
| S3 | OWASP ASVS 5.0.0, V6 Authentication | https://raw.githubusercontent.com/OWASP/ASVS/v5.0.0_release/5.0/en/0x15-V6-Authentication.md | Primary |
| S4 | OWASP ASVS 5.0.0, V16 Security Logging and Error Handling | https://raw.githubusercontent.com/OWASP/ASVS/v5.0.0_release/5.0/en/0x25-V16-Security-Logging-and-Error-Handling.md | Primary |
| S5 | OWASP Top 10:2025 A01 Broken Access Control (repo history for release status) | https://owasp.org/Top10/2025/A01_2025-Broken_Access_Control/ ; https://raw.githubusercontent.com/OWASP/Top10/master/2025/docs/en/A01_2025-Broken_Access_Control.md | Primary |
| S6 | OWASP Authorization Cheat Sheet | https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html | Primary |
| S7 | OWASP IDOR Prevention Cheat Sheet | https://cheatsheetseries.owasp.org/cheatsheets/Insecure_Direct_Object_Reference_Prevention_Cheat_Sheet.html | Primary |
| S8 | OWASP API Security Top 10 2023, API1 BOLA | https://owasp.org/API-Security/editions/2023/en/0xa1-broken-object-level-authorization/ | Primary |
| S9 | OWASP WSTG-ATHZ-04 IDOR (master) | https://raw.githubusercontent.com/OWASP/wstg/master/document/4-Web_Application_Security_Testing/05-Authorization/04-Insecure_Direct_Object_References.md | Primary |
| S10 | Autorize (Burp extension) | https://github.com/Quitten/Autorize | Primary (project) |
| S11 | AuthMatrix (Burp extension) | https://github.com/SecurityInnovation/AuthMatrix | Primary (project) |
| S12 | ZAP Access Control add-on | https://github.com/zaproxy/zap-extensions/tree/main/addOns/accessControl | Primary (project) |
| S13 | CWE-639 and CWE-566 | https://cwe.mitre.org/data/definitions/639.html ; https://cwe.mitre.org/data/definitions/566.html | Primary |
| S14 | stancl/tenancy on Packagist and GitHub (releases, master composer.json, v4 docs link) | https://repo.packagist.org/p2/stancl/tenancy.json ; https://github.com/archtechx/tenancy ; https://v4.tenancyforlaravel.com | Primary |
| S15 | stancl/tenancy v3 single-database tenancy | https://tenancyforlaravel.com/docs/v3/single-database-tenancy | Primary |
| S16 | spatie/laravel-multitenancy (Packagist, queue docs, `src/Tasks`) | https://repo.packagist.org/p2/spatie/laravel-multitenancy.json ; https://spatie.be/docs/laravel-multitenancy/v4/basic-usage/making-queues-tenant-aware ; https://github.com/spatie/laravel-multitenancy | Primary |
| S17 | Laravel 13.x Eloquent (global scopes, `HasUuids`/`HasUlids`) | https://laravel.com/docs/13.x/eloquent | Primary |
| S18 | Laravel 13.x Context | https://laravel.com/docs/13.x/context | Primary |
| S19 | Laravel 13.x Cache | https://laravel.com/docs/13.x/cache | Primary |
| S20 | Laravel 13.x Broadcasting | https://laravel.com/docs/13.x/broadcasting | Primary |
| S21 | Laravel 13.x Routing (`scopeBindings`) | https://laravel.com/docs/13.x/routing | Primary |
| S22 | Laravel 13.x Release Notes | https://laravel.com/docs/13.x/releases | Primary |
| S23 | Laravel 13.x skeleton `config/session.php`; Authentication docs | https://github.com/laravel/laravel/blob/13.x/config/session.php ; https://laravel.com/docs/13.x/authentication | Primary |
| S24 | Laravel 13.x Scout | https://laravel.com/docs/13.x/scout | Primary |
| S25 | Meilisearch multitenancy and tenant tokens; SDK token guide | https://www.meilisearch.com/docs/learn/security/multitenancy_tenant_tokens ; https://www.meilisearch.com/docs/learn/security/generate_tenant_token_sdk | Primary |
| S26 | meilisearch-php `TenantToken.php` | https://github.com/meilisearch/meilisearch-php/blob/main/src/Endpoints/TenantToken.php | Primary (source) |
| S27 | Typesense API keys (scoped keys) | https://typesense.org/docs/latest/api/api-keys.html | Primary |
| S28 | MySQL 8.4 CREATE VIEW | https://dev.mysql.com/doc/refman/8.4/en/create-view.html | Primary |
| S29 | MySQL 8.4 Privileges Provided | https://dev.mysql.com/doc/refman/8.4/en/privileges-provided.html | Primary |
| S30 | AWS Database Blog, row-level security in Aurora MySQL / RDS for MySQL (2025-06-16) | https://aws.amazon.com/blogs/database/implement-row-level-security-in-amazon-aurora-mysql-and-amazon-rds-for-mysql | Secondary (vendor blog) |
| S31 | PostgreSQL row security policies | https://www.postgresql.org/docs/current/ddl-rowsecurity.html | Primary |
| S32 | PHPStan custom rules | https://phpstan.org/developing-extensions/rules | Primary |
| S33 | spaze/phpstan-disallowed-calls README | https://github.com/spaze/phpstan-disallowed-calls | Primary (project) |
| S34 | Larastan on Packagist | https://repo.packagist.org/p2/larastan/larastan.json | Primary |
| S35 | Pest architecture testing; Pest and pest-plugin-laravel on Packagist | https://pestphp.com/docs/arch-testing ; https://repo.packagist.org/p2/pestphp/pest.json | Primary |
| S36 | Laravel 13.x Fortify (TOTP, passkeys); Fortify CHANGELOG | https://laravel.com/docs/13.x/fortify ; https://github.com/laravel/fortify/blob/1.x/CHANGELOG.md | Primary |
| S37 | spatie/laravel-permission docs (v8 intro, teams, policies) and Packagist | https://spatie.be/docs/laravel-permission/v8/introduction ; https://github.com/spatie/laravel-permission/blob/main/docs/basic-usage/teams-permissions.md ; https://spatie.be/docs/laravel-permission/v6/best-practices/using-policies | Primary |
| S38 | Odoo Documents 18.0 and 19.0 | https://www.odoo.com/documentation/18.0/applications/productivity/documents.html ; https://www.odoo.com/documentation/19.0/applications/productivity/documents.html | Primary |
| S39 | Odoo forum, "Documents rights" (non-staff answer) | https://www.odoo.com/forum/help-1/documents-rights-281281 | Secondary |
| S40 | SharePoint Online limits | https://learn.microsoft.com/en-us/office365/servicedescriptions/sharepoint-online-service-description/sharepoint-online-limits | Primary |
| S41 | SharePoint, manage permission scope | https://learn.microsoft.com/en-us/sharepoint/manage-permission-scope | Primary |
| S42 | SharePoint Server, fine-grained permissions best practices | https://learn.microsoft.com/en-us/sharepoint/sites/best-practices-for-using-fine-grained-permissions-in-sharepoint-server | Primary |
| S43 | Google Drive Help: folder sharing, limited access, sharing settings | https://support.google.com/drive/answer/7166529 ; https://support.google.com/drive/answer/14254362 ; https://support.google.com/drive/answer/2494822 | Primary |
| S44 | Google Workspace Updates, Sept 2024 and Feb 2025 | https://workspaceupdates.googleblog.com/2024/09/google-drive-beta-to-restrict-folder-access.html ; https://workspaceupdates.googleblog.com/2025/02/updating-access-experience-in-google-drive.html | Primary |
| S45 | M-Files user guide: named ACLs, permissions | https://userguide.m-files.com/user-guide/latest/eng/Named_access_control_lists.html ; https://userguide.m-files.com/user-guide/latest/eng/Permissions_and_Default_permissions.html | Primary |
| S46 | Microsoft Search overview | https://learn.microsoft.com/en-us/microsoftsearch/overview-microsoft-search | Primary |
| S47 | Elastic document- and field-level security | https://www.elastic.co/docs/deploy-manage/users-roles/cluster-or-deployment-auth/controlling-access-at-document-field-level | Primary |
| S48 | Algolia user-restricted access (secured API keys) | https://www.algolia.com/doc/guides/security/api-keys/how-to/user-restricted-access-to-data/ | Primary |
| S49 | Zanzibar, USENIX ATC 2019 | https://www.usenix.org/conference/atc19/presentation/pang | Primary |
| S50 | OpenFGA at CNCF; OpenFGA Google Drive model | https://www.cncf.io/projects/openfga/ ; https://openfga.dev/docs/modeling/advanced/gdrive | Primary |
| S51 | AuthZed SpiceDB relationships; pricing | https://authzed.com/docs/spicedb/concepts/relationships ; https://authzed.com/pricing | Primary / Vendor |
| S52 | evansims/openfga-php (archived) | https://github.com/evansims/openfga-php | Primary (project) |
| S53 | SharePoint external sharing settings | https://learn.microsoft.com/en-us/sharepoint/turn-external-sharing-on-or-off | Primary |
| S54 | Box shared links | https://support.box.com/hc/en-us/articles/360043697094 | Primary |
| S55 | Dropbox link permissions | https://help.dropbox.com/files-folders/share/set-link-permissions | Primary |
| S56 | Google Cloud Access Transparency; Access Approval overview and approving requests | https://docs.cloud.google.com/assured-workloads/access-transparency/docs/overview ; https://docs.cloud.google.com/assured-workloads/access-approval/docs/overview ; https://docs.cloud.google.com/assured-workloads/access-approval/docs/approve-requests | Primary |
| S57 | Microsoft Purview Customer Lockbox | https://learn.microsoft.com/en-us/purview/customer-lockbox-requests | Primary |
| S58 | Atlassian support ticket eligibility; data access FAQ (search excerpt); AX-78 | https://confluence.atlassian.com/support/eligibility-for-support-tickets-for-atlassian-cloud-979419943.html ; https://support.atlassian.com/atlassian-cloud/kb/atlassian-cloud-support-data-access-request-faq/ ; https://jira.atlassian.com/browse/AX-78 | Primary / Secondary |
| S59 | Salesforce, grant login access | https://help.salesforce.com/s/articleView?id=sf.granting_login_access.htm&language=en_US&type=5 | Primary |
| S60 | AWS Well-Architected SEC03-BP03 emergency access | https://docs.aws.amazon.com/wellarchitected/latest/framework/sec_permissions_emergency_process.html | Primary |
| S61 | AWS KMS pricing | https://aws.amazon.com/kms/pricing/ | Primary |
| S62 | Amazon S3 default encryption FAQ | https://docs.aws.amazon.com/AmazonS3/latest/userguide/default-encryption-faq.html | Primary |
| S63 | Amazon S3 Bucket Keys | https://docs.aws.amazon.com/AmazonS3/latest/userguide/bucket-key.html | Primary |
| S64 | AWS KMS resource quotas | https://docs.aws.amazon.com/kms/latest/developerguide/resource-limits.html | Primary |
| S65 | AWS whitepaper, SaaS Tenant Isolation Strategies (bridge model) | https://docs.aws.amazon.com/whitepapers/latest/saas-tenant-isolation-strategies/the-bridge-model.html | Primary |
| S66 | AWS Architecture Blog, multi-tenant KMS key strategy (2025-08-21) | https://aws.amazon.com/blogs/architecture/simplify-multi-tenant-encryption-with-a-cost-conscious-aws-kms-key-strategy | Primary (vendor guidance) |
| S67 | Laravel 13.x Encryption; Eloquent mutators and casts | https://laravel.com/docs/13.x/encryption ; https://laravel.com/docs/13.x/eloquent-mutators | Primary |
| S68 | MySQL 8.4 InnoDB data-at-rest encryption | https://dev.mysql.com/doc/refman/8.4/en/innodb-data-encryption.html | Primary |
| S69 | Amazon RDS encryption | https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Overview.Encryption.html | Primary |
| S70 | NIST SP 800-63B-4 (final, July 2025) | https://csrc.nist.gov/pubs/sp/800/63/b/4/final ; https://pages.nist.gov/800-63-4/sp800-63b.html | Primary |
| S71 | OpenAI, March 20 ChatGPT outage (search excerpt; page returned 403) | https://openai.com/index/march-20-chatgpt-outage/ | Secondary (excerpt) |
| S72 | GitHub Blog, session-handling race condition (2021-03-18) | https://github.blog/2021-03-18-how-we-found-and-fixed-a-rare-race-condition-in-our-session-handling/ | Primary |
| S73 | Valve, Steam Store caching incident (2015) | https://store.steampowered.com/news/19852/ | Primary |
| S74 | KrebsOnSecurity, First American Financial (2019) | https://krebsonsecurity.com/2019/05/first-american-financial-corp-leaked-hundreds-of-millions-of-title-insurance-records/ | Secondary (reputable journalism; primary for the finding) |
| S75 | UpGuard, Power Apps portals (2021) | https://www.upguard.com/breaches/power-apps | Primary (researcher) |
| S76 | KrebsOnSecurity, public Salesforce sites leaking data (2023; excerpt) | https://krebsonsecurity.com/2023/04/many-public-salesforce-sites-are-leaking-private-data/ | Secondary (excerpt) |
| S77 | Wiz, ChaosDB (2021) | https://www.wiz.io/blog/chaosdb-how-we-hacked-thousands-of-azure-customers-databases | Primary (researcher) |
| S78 | Microsoft Security Blog, Storm-0558 (2023-07-14); CSRB review page | https://www.microsoft.com/en-us/security/blog/2023/07/14/analysis-of-storm-0558-techniques-for-unauthorized-email-access/ ; https://www.cisa.gov/resources-tools/resources/CSRB-Review-Summer-2023-MEO-Intrusion | Primary / Secondary |
| S79 | BleepingComputer, Asana MCP data exposure (2025) | https://www.bleepingcomputer.com/news/security/asana-warns-mcp-ai-feature-exposed-customer-data-to-other-orgs/ | Secondary |
| S80 | Railway incident report, 30 March 2026 | https://blog.railway.com/p/incident-report-march-30-2026-accidental-cdn-caching | Primary |
| S81 | Wiz, 38 TB exposed by Microsoft AI researchers (2023) | https://www.wiz.io/blog/38-terabytes-of-private-data-accidentally-exposed-by-microsoft-ai-researchers | Primary (researcher) |
| S82 | Cloudflare, Cloudbleed incident report (2017) | https://blog.cloudflare.com/incident-report-on-memory-leak-caused-by-cloudflare-parser-bug/ | Primary |
| S83 | R10, Data Privacy Act duties (NPC Circular 2023-06 ss.12–14, 16, 24, 25, 29, 32; Circular 16-03; IRR ss.28, 38, 44) | research/R10-data-privacy-for-saas-dms.md | Internal |
| S84 | R13, upload security (storage keys, serving, logging) | research/R13-upload-security-and-scanning.md | Internal |
| S85 | R03, ISO standards primer (Annex A mapping) | research/R03-iso-standards-primer.md | Internal |
