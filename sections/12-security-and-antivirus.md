# 12 — Security and Antivirus

## 1. Purpose

Make sure **no uploaded file can harm Officetul, its users or other companies** (Part A, file pipeline), and that **one company never sees another company's Documents, while people inside a company see only what they should** (Part B, access and isolation). Activity log protection is in [Section 09](09-audit-trails-and-integrity.md).

## 2. Status

- **Part A:** owner agreed (Round 9, D-047, D-048, D-051), 2026-10-20. Research: [R13](../research/R13-upload-security-and-scanning.md), with tool risks from [R11](../research/R11-file-formats-and-previews.md).
- **Part B:** owner agreed (Round 11, D-052 to D-059), 2026-10-20. Research: [R15](../research/R15-tenant-isolation-and-permissions.md). Designed without the Officetul code; checked against it later (D-013).

## 3. Executive Summary

- **Fail closed.** A file is visible to nobody (not even in the Inbox) until it has a **clean virus-scan verdict**. If the scanner is down, files wait. **Never release on a timer.** *(Recommendation; OWASP ASVS basis)*
- **ClamAV with hardened settings is the MVP scanner**, in its own container. Its default settings silently skip files over 100 MB and can report them "clean", so its limits must be raised and over-limit files flagged. It needs about **3–4 GB of RAM**, from a 2020 figure (Estimate). *(Verified fact for the defaults)*
- **Antivirus is not enough.** It misses many malicious PDFs and Office files. So the tools that read files run **isolated, with no internet, and with time and memory limits**. Downloads come from a **separate domain** as forced downloads. *(Recommendation)*
- **Never send client files to VirusTotal**: their contents can be shared with VirusTotal's paying customers. If hosted on AWS, GuardDuty's S3 malware scanning (same-region) can be a second engine later. *(Verified fact)*
- **Company isolation is fail-closed.** Documents reuses Officetul's tenancy model, but any query or job that doesn't know its company **stops with an error**. Two-company isolation tests block every release. *(D-052)*
- **Folders decide access** at three Access levels (View, Contribute, Manage), with Restricted and Confidential folders; downloading originals is a separate role ability. *(D-053 to D-055)*
- **No hints** about hidden Documents anywhere: search, counts, panels, duplicates. *(D-056)*
- **Officetul staff need the client's approval** to look at Documents: read-only, 4 hours, logged. *(D-057)*
- **Infected files** are blocked, the uploader and company admins are notified, and the file is kept locked for 30 days for appeals, then deleted. The fingerprint and verdict stay in the log. *(Recommendation; the 30-day period is for the lawyer to confirm)*

## 4. Business Context

Clients email and upload files from anywhere. A single malicious file that reached another user, or exposed another company's documents, would destroy trust and could be a reportable breach under the Data Privacy Act (D-032).

## 5. Verified Research Findings

| Finding | Source |
|---|---|
| ClamAV defaults: `MaxFileSize` 100 MB, `StreamMaxLength` 100 MB; over-limit files may pass unless `AlertExceedsMax` is on; hard ceiling 2 GB | R13 §3 |
| ClamAV needs about 3–4 GiB RAM, plus about 2.4 GiB briefly during daily signature reloads (2020 figure) | R13 §3 |
| ClamAV has had PDF and OLE2 parser bugs, so it runs isolated too | R13 §3 |
| VirusTotal shares submitted files with premium customers | R13 §3 |
| GuardDuty Malware Protection for S3: $0.09/GB + $0.215 per 1,000 files (US East); Singapore availability unverified | R13 §3 |
| Object Lock storage can't delete an infected file, so quarantine must be a separate bucket | R13 §4 |
| ImageMagick, Ghostscript, LibreOffice, Poppler and pdf.js all have serious CVE histories | R11, R13 §5 |

## 6. Technical Analysis

### 6.1 Pipeline *(Recommendation, R13 §9)*

1. **Sign:** check login, company quota, size (≤ 1 GB), type allowlist; issue short-lived upload URLs.
2. **Upload:** resumable pieces into **quarantine** storage.
3. **Validate and hash:** isolated worker with no network computes SHA-256, checks the real type against the extension, test-opens the file, and applies pixel and archive limits.
4. **Scan:** ClamAV (hardened). Clean → continue. Infected → block. Error → retry.
5. **Promote:** move to permanent storage under a random name; record the fingerprint.
6. **Process:** previews and OCR in isolated queues. A failure here does not block the original.
7. **Inbox:** the item appears for review (D-014), visible only to that company.
8. **Serve:** permission check every time; short-lived link on a separate domain; forced download; no content sniffing.

### 6.2 Failure handling *(Recommendation, R13 §10)*

| Scenario | Policy |
|---|---|
| Scanner down | Files stay in quarantine; retry; alert ops at 15 min; **never auto-release** |
| Scan timeout | Retry up to 3 times, then held for review by our ops; the company sees "security check pending" |
| Infected | Blocked, never shown; uploader and company admins notified; locked 30 days for appeal, then deleted; fingerprint and verdict kept in the log |
| False positive claimed | Our ops review; release with a per-fingerprint exception and a log entry |
| Password-protected file (can't be scanned inside) | Held; a **company admin can accept the risk** (logged); stored with an "Encrypted" badge; no preview |
| Corrupt or unreadable | Rejected with a reason shown to the uploader; not filed |
| Converter crash | Original stays downloadable without a preview; alert if the same file keeps failing |
| Same file, same company | Duplicate warning (D-042) |
| Same file, different company | Processed separately, so no one can learn what another company holds |
| Abandoned upload | Cleaned up after 1–7 days; quota released |
| Virus signatures out of date | Alert at 24 h; keep scanning with the last good version and record the version on each verdict |
| Email from an unapproved sender or failing DMARC | Held, visible to company admins only, not processed until released |

### 6.3 Isolation of file-reading tools

Each tool runs in its own container, as an unprivileged user, with **no network access** and with CPU, memory and time limits:
- Gotenberg/LibreOffice: one job at a time, outbound URLs blocked.
- Tika: internal network only, version ≥ 3.2.2.
- Poppler, Tesseract, libvips: pixel limits, untrusted-input mode.
- ClamAV: its own container.

The worker pool is separate from the web servers.

## 7. Alternatives and Trade-offs

| Question | Options | Recommendation |
|---|---|---|
| Scanner | ClamAV / GuardDuty (AWS only) / commercial API | **ClamAV now**; GuardDuty as a second engine later if on AWS; no third-party API that shares files |
| When the scanner is down | Release after a timeout / **hold** | Hold: availability matters less than safety here |
| Encrypted files | Reject / **hold, admin may accept** / ops only | Admin acceptance: bank statements are commonly password-protected |
| Infected retention | Delete at once / **30 days locked** / forever | 30 days for appeals; lawyer to confirm proportionality |

## 8. Recommendation

Adopt §6 as the MVP file-security design. Later: content disarm and reconstruction for high-risk companies, a second scan engine, stronger sandboxing (gVisor), rescanning the Inbox on new signatures, archive unpacking with bomb checks, SIEM export. *(Recommendation, R13 §11)*

## 9. Cost and Operational Implications

- ClamAV RAM (≈3–4 GB) plus converter containers: Phase 10 sizing.
- **Someone must watch alerts** (scanner down, held files, stale signatures). This is an ops duty for the company (with D-032's incident procedure).

## 10. Risks and Limitations

| ID | Risk | Mitigation |
|---|---|---|
| RK-35 | **Malware that AV misses** reaches a user | Isolated processing; forced downloads from a separate domain; page-image previews |
| RK-36 | **Held files pile up** when the scanner fails unnoticed | Alerts; ops duty; status shown to users |
| RK-37 | **Admins accept a malicious encrypted file** | Warning text; logged acceptance; never previewed |

## 11–12. Questions and Decisions

Grilling Round 9, Q4–Q5 and Q8 (Part A); Round 11, Q1–Q8 (Part B).

## 13. Confirmed Decisions

| ID | Decision |
|---|---|
| D-047 | Fail-closed scanning and failure policies |
| D-048 | Password-protected files: admin may accept risk; Encrypted badge |
| D-051 | Tool isolation; page-image previews; separate download domain; no AGPL tools |
| D-052 | Fail-closed company isolation; isolation tests block releases |
| D-053 | Folder Access levels, inheritance, Restricted folders; Links give View only |
| D-054 | Download, export and admin are role abilities |
| D-055 | Confidential folders |
| D-056 | No hints about hidden Documents; duplicates flagged to the folder's managers; report limited to roles seeing all finance folders |
| D-057 | Client-approved, time-limited Support access; break-glass rules |
| D-058 | MFA and session limits |
| D-059 | Provider-key encryption; no per-company keys in the MVP |

## 14. Open Questions

R13 §12: hosting (AWS Singapore vs local data centre); ClamAV RAM at today's database size; GuardDuty in Singapore; Postmark DMARC verdicts; disabling LibreOffice macros; 30-day infected retention (lawyer).

## 15. Dependencies

Depends on D-014, D-015, D-020, D-032, D-033, D-042. Feeds Section 09 (logging), 13 (database and search design), 14 (sizing), 15 (backup excludes quarantine), 18 (cost).

## 16. Sources

R13 (54 sources), R11 and R15 (85 sources), all accessed 2026-10-20.

## Part B — Access and isolation (Phase 8)

### B1. Verified research findings

| Finding | Source |
|---|---|
| Laravel tenancy packages leave queries **unfiltered** when no tenant is set; global scopes apply only to Eloquent models, not raw `DB` queries | R15 §1–2 |
| MySQL has no row-level security (AWS documentation); a view can't follow the session's company | R15 §1.4 |
| Real cross-company leaks came from cached logged-in pages (Steam 2015), state left over between requests (GitHub 2021), guessable IDs (First American) and over-broad storage links (Microsoft 38 TB) | R15 §3 |
| SharePoint advises "minimize the number of unique permission scopes"; Google lets you restrict access only at folder level, not per file | R15 §5.1 |
| OpenFGA and SpiceDB have no maintained PHP client, and keeping them in sync is the app's job | R15 §5.1 |
| Microsoft Customer Lockbox: staff access needs customer approval; it lasts at most 4 hours | R15 §7 |
| Laravel Fortify supports TOTP two-factor and, since v1.37.0, passkeys | R15 §8.2 |
| NIST SP 800-63B-4 (AAL2): idle timeout no more than 1 hour; overall re-login within 24 hours | R15 §8.3 |
| AWS KMS: $1 per key per month, up to $3 with rotations | R15 §8.1 |
| NPC Circular 2023-06 s.16: multi-factor authentication for access to sensitive personal information; s.24: limit copying | R15 §9 |

### B2. Design *(Recommendation → agreed)*

**Layer 0: company boundary (D-052).**
- Every Documents model uses one guard that **throws** when no company is set.
- Cross-company code (ops, billing) uses an explicit, logged "run as system" escape.
- Public IDs are ULIDs or UUIDv7.
- Another company's ID returns **404**.
- Composite `(tenant_id, id)` keys stop a record pointing at another company's row.
- Separate database users.
- All 21 leak paths in R15 §2 have a named control: jobs, scheduled commands, cache keys, HTTP caching, search, storage links, Derived files, broadcasting, exports, logs and emails.

**Layer 1: role abilities (D-054).** `use`, `download` (original files), `export` (bulk or folder), `admin`. Full company export is company admin only.

**Layer 2: folder grants (D-053, D-055).**

| Access level | Allows |
|---|---|
| View | Official version, details, preview |
| Contribute | + Drafts and history, file from the Inbox, upload new Versions as Draft, move own Drafts to Trash |
| Manage | + mark Official, withdraw, restore, Trash anything, change the folder's access |

- Subfolders inherit their parent's access unless **Restricted**.
- A **Confidential folder** hides its contents from company admins. An admin can add themselves, which is logged and announced.
- No per-Document rules.
- "Why can I see this?" and "Who can see this?" panels.

**Layer 3: Links (D-020, D-053).** A switched-on link type gives **View** to anyone who may open the linked Officetul record.

**Search and previews (D-056).**
- Laravel proxies every search, with the company and access filter **inside** the query.
- Counts, snippets, suggestions and empty states come only from the permitted set.
- Previews and thumbnails pass the same check, are served `private, no-store` and are keyed by Version, never by file fingerprint.

**Support access (D-057):**
1. **Request:** Officetul staff give the ticket, the reason and the scope.
2. **Approval:** the company admin approves with MFA.
3. **Access:** read-only, 4 hours by default (24 hours at most), then revoked automatically.
4. **Logging:** everything is logged in the client's Activity log.

Break-glass access needs two Officetul staff and lasts at most 1 hour; the client is told within 24 hours.

**Login and sessions (D-058).**
- Authenticator app or passkey; no SMS.
- Required for company admins and Officetul staff; a company switch requires it for everyone.
- Sessions end after 60 minutes idle or 12 hours in total.
- Users re-enter their password before deleting permanently, changing access, exporting, or approving support access.

**Encryption (D-059).** Provider-key encryption for storage, the database, backups and exports; TLS everywhere; no per-company keys in the MVP.

### B3. Testing that blocks releases *(D-052; R15 §4.2)*

1. A two-company test setup with mirrored data (same folder names, same file, same words).
2. A route sweep: every Documents route, called as company A with company B's IDs, must return 404. A new route without coverage fails the sweep.
3. Architecture tests: every Documents model uses the guard, and raw `DB` queries are banned outside an allowlist.
4. Static analysis bans `withoutGlobalScopes` and raw queries.
5. A "list equals detail" test: the documents a user sees in lists and search equal the documents they may open.
6. Job, search, preview and storage tests.
7. A manual two-company pass (Burp Autorize or OWASP ZAP) before each pilot. An external penetration test every year.

### B4. Alternatives and trade-offs

| Question | Options | Choice |
|---|---|---|
| Permission model | Roles only / **folder grants + role abilities** / separate permission service (OpenFGA, SpiceDB) | Folder grants: roles alone can't say "only HR sees HR"; a separate service is too much for a small team |
| Per-Document permissions | Yes / **no, use Restricted folders** | No: Microsoft and Google both steer users away from per-file rules |
| Search filtering | Filter after search / access lists stored in the index / **filter inside the query** | Inside the query: no leaks through counts, no stale lists |
| Per-company encryption keys | **Later, on request** / MVP | Later: about $1–3 per company per month (Estimate) and no help against app bugs |

### B5. Risks

| ID | Risk | Mitigation |
|---|---|---|
| RK-42 | **Cross-company leak** through a path the tests miss | Fail-closed guard; route sweep; alert on any cross-company ID hit; 24-hour client notice (D-032) |
| RK-43 | **Officetul's existing tenancy can't fail closed** without changes outside Documents | Code review first (D-013); the guard lives in the Documents models only |
| RK-44 | **Confidential folders get orphaned** when their only manager leaves | Admin sees that the folder exists and can add themselves (logged, announced) |
| RK-45 | **Support access is bypassed** by an old "log in as" tool or direct database access | Remove or wrap it (D-057); database audit plugin next release |
| RK-46 | **Link visibility surprises** a client (a receipt seen by an expense approver) | Off by default; "Why can I see this?" panel |

### B6. Open questions

- **Officetul code (D-013):** the R15 §13 checklist, now in the [ERP technical questionnaire](../questionnaires/erp-technical-questionnaire.md) (Q18–Q24).
- **How Officetul decides who may open a record** (an expense, a project). Link visibility depends on it.
- Can one user belong to several companies? Accounting firms make this likely.
- **Lawyer:** should the support-access terms go into the data processing addendum, and does break-glass need advance consent in the contract? (A22)

## 17. Next Steps

- [x] Fail-closed pipeline agreed (D-047)
- [x] Failure policies agreed (D-047, D-048)
- [x] Isolation approach agreed (D-051)
- [x] Company isolation, permissions, support access, login and encryption agreed (D-052 to D-059)
- [ ] Code review against Officetul (D-013)
- [ ] Lawyer question A22 answered
