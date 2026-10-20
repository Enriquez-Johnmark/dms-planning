# R20 — MVP scope synthesis: inventory, consistency check, effort and release gates

- **Status:** First pass for Phase 11 (Section 17, MVP scope). Synthesis only: no code, no new market research.
- **Prepared:** 2026-10-21.
- **Question:** What do the confirmed decisions (D-001 to D-088) put in the first version of Officetul Documents? Do they agree with each other? Can 1–2 developers build it in the time D-005 allows, and what must be true before each pilot starts?
- **Inputs:** [DECISION-LOG.md](../DECISION-LOG.md) (the source of truth), [GLOSSARY.md](../GLOSSARY.md), [OPEN-QUESTIONS.md](../OPEN-QUESTIONS.md), [DISCUSSION-TRACKER.md](../DISCUSSION-TRACKER.md), [MASTER-PLAN.md](../MASTER-PLAN.md), [PROJECT-CONTEXT.md](../PROJECT-CONTEXT.md), Sections 01–07, 09–16 and 18, the [document-type catalogue](../appendices/document-type-catalogue.md), the [ERP technical questionnaire](../questionnaires/erp-technical-questionnaire.md), and R04, R11, R15, R17 and R19 where a section points to them.
- **Labels:** **Verified fact** (stated in the decision log or an agreed section), **Estimate**, **Open question**, **Recommendation** (proposed, not approved). Decision IDs `D-0xx` and open-question IDs `OQ-0xx` are the citations; no external source was consulted for this note.

**Terms** follow [GLOSSARY.md](../GLOSSARY.md) exactly. "Officetul record" means a record owned by another Officetul app (an expense, a customer, a project). Two release tiers are used throughout:

| Code | Meaning |
|---|---|
| **M-own** | Must exist before our **own-company pilot** starts (D-017) |
| **M-ext** | Must exist before the **first external client** stores Documents (D-017, D-032) |
| **S** | Should: decided or recommended for the MVP, but can follow the first external client without breaking what the decision protects |
| **C** | Could: cheap extra, only if time allows |

---

## 0. Summary

- **The decided MVP is about 99 Must items in 10 epics.** Of these, 74 are needed for the own-company pilot and 25 more before the first external client. Another 12 items are recommended as Should and 2 as Could. *(Count from §1; MoSCoW is a Recommendation)*
- **It does not fit "4–6 months with 1–2 developers".** That figure was set in Round 1 (D-005) and restated for an 8-bullet MVP in Round 4 (D-014). About 70 later decisions (D-019 to D-088) added security, integrity, backup and operations work.
  - Must work: about **57–86 developer-weeks**, or **65–98 with 15% contingency**.
  - Capacity: two developers over 4–6 months give about **28–42 productive developer-weeks**. *(Estimate, §4)*
- **Realistic dates with two full-time developers** *(Estimate)*:
  - own-company pilot about **7–10 months** after scope approval;
  - first external client about **3 months later**.
  - With one developer, roughly double, and several two-person duties can't be met (§3, C-12).
- **Biggest conflicts and gaps** (§3):
  - nobody decided **who sees an Inbox item** before it is filed;
  - **link-granted access** (D-053) depends on OQ-025, which is unanswered;
  - **scan-to-email** depends on OQ-018, which was deferred to Phase 7 and never decided;
  - a searchable PDF/A from OCRmyPDF (D-049) may need Ghostscript, which D-051 bans;
  - several decisions change **all of Officetul** (MFA, sessions, database users, MySQL upgrade), and no decision says who does that work.
- **Ten decisions for the owner** are in §7. Each has a recommended answer worded so that "yes" accepts it.

---

## 1. MVP feature inventory

**Verified fact:** each item's decision IDs. **Recommendation:** the MoSCoW tier. "↓Q#" marks an item the decisions put in the MVP that this note proposes to move down; the owner accepts or rejects that in §7.

### 1.1 Epic E1 — Capture and Inbox

| # | Capability | From | What it means | Tier |
|---|---|---|---|---|
| 1.1 | Browser upload | D-015, D-026, D-045, D-046 | Resumable upload of files up to 1 GB, straight to quarantine storage; our worker computes the SHA-256 | M-own |
| 1.2 | Phone capture | D-015, D-016, D-044 | The phone's own scanner app (iOS Notes, Google Drive scan), then a mobile-friendly upload; HEIC accepted | M-own |
| 1.3 | Review Inbox | D-014, D-015 | List of Inbox items; file one into a Folder with a Document type and details, or reject it | M-own |
| 1.4 | Filing choice | D-036 | The user picks Official record or Draft; the default comes from the Document type | M-own |
| 1.5 | Capture record | D-031 | Who, when, which channel, which device; full email headers kept | M-own |
| 1.6 | Duplicate warning | D-042, D-056, D-070 | Same fingerprint → "link to existing" or "file anyway"; a duplicate of a hidden Document is flagged only to that Folder's Manage holders | M-own |
| 1.7 | Inbox visibility and clean-up | **Not decided** (§3 C-04) | Who inside a company sees an Inbox item before filing; what happens to rejected items | M-own once decided (Q4) |
| 1.8 | Scan-to-email | D-015, D-045, Section 12 §6.2 | A per-company address, approved senders only, DMARC failures held, about 35 MB per message | S ↓Q6 |
| 1.9 | Scanner guide | D-016 | One-page scanner compatibility guide | M-ext |

### 1.2 Epic E2 — Documents, Versions and lifecycle

| # | Capability | From | What it means | Tier |
|---|---|---|---|---|
| 2.1 | Core model | D-026, D-035, D-038, D-075 | Document, Version, Original file, Derived file. Versions and Original files are write-once; the database allows at most one Official version per Document | M-own |
| 2.2 | Version states | D-038 | Draft → Official → Superseded, plus Withdrawn (reason required); a Superseded version can be restored as Official | M-own |
| 2.3 | Official marking | D-037 | Only Manage holders mark official, withdraw or restore; optional note; logged | M-own |
| 2.4 | New uploads are Drafts | D-036, D-039 | A later upload to a Document is always a Draft and shows "based on vN" | M-own |
| 2.5 | Stale-draft warning | D-039 | "Based on v1, but v2 is now official" | S ↓Q2 |
| 2.6 | Preset Document types | D-021, D-023, D-024 | 22 general presets, including Invoice (purchase), Sales Invoice (issued) and Payment Receipt with the "OR" alias; companies can rename them | M-own |
| 2.7 | Add Document types and fields | D-024, D-071 | Company admins add types and type-specific fields | M-ext |
| 2.8 | Accounting-firm pack | D-024 | 8 optional types (setup data) | C ↓Q2 |
| 2.9 | Details | D-026, D-071 | Typed standard details (document date, reference number, amount, expiry date, event date) with ranges and sorting; type-specific details filter by exact value | M-own |
| 2.10 | Details history | D-043 | Changes logged with old and new values; details are not versioned | M-own |
| 2.11 | Paper original | D-008, D-026 | Kept? Where? Custodian? on every type | M-own |
| 2.12 | Revision label and Issue status | D-026, D-035 | Company labels ("Rev C", "For Construction") kept separate from the Official version | M-own |
| 2.13 | Expiry and event dates | D-026, D-038 | Captured now; "Expired" is a badge, not a state; reminders come later | M-own |
| 2.14 | Related documents | D-040 | An amendment is a separate Document that "amends" the original | S ↓Q2 |
| 2.15 | Trash | D-041 | Restorable; no automatic purge; an Official version can only be withdrawn, not deleted alone | M-own |
| 2.16 | Permanent delete | D-041 | Company admin only, with a logged reason (categories in 8.8) | M-own |
| 2.17 | Signed PDFs untouched | D-031, D-049 | Signed or e-notarized PDFs are never rewritten; OCR writes a text sidecar only | M-own |
| 2.18 | Sensitivity label | Catalogue §1 (D-024) | Public / Internal / Confidential / SPI per Document type | M-own (label only; see Q10) |

### 1.3 Epic E3 — Folders and permissions

| # | Capability | From | What it means | Tier |
|---|---|---|---|---|
| 3.1 | Company boundary | D-052, D-075 | Fail-closed guard (a query with no known company stops); another company's IDs return 404; composite company keys; ULIDs | M-own |
| 3.2 | Folder access | D-053 | Grants to roles or users at View, Contribute or Manage; subfolders inherit unless Restricted; no per-Document rules | M-own |
| 3.3 | Role abilities | D-033, D-054 | `use`, `download`, `export`, `admin`; without `download` a user sees page-image previews only | M-own |
| 3.4 | No hints | D-026, D-056 | Search, counts, snippets, panels, previews, the Activity log and empty states only reflect what the viewer may open | M-own |
| 3.5 | "Why can I see this?" | D-020, D-053 | Panel on each Document | M-own |
| 3.6 | Confidential folders | D-055 | Admins don't see contents; an admin can add themselves, which is logged and announced | M-ext |
| 3.7 | "Who can see this?" | D-053 | Panel on each Folder and Document | M-ext |
| 3.8 | Password re-entry | Section 12 B2 (with D-058) | Before permanent delete, access changes, export and approving Support access | M-ext |
| 3.9 | Link-granted View | D-020, D-053 | A company admin can switch on a link type so that people who can open the Officetul record get View | S ↓Q5 |

### 1.4 Epic E4 — Links and the ERP contract

| # | Capability | From | What it means | Tier |
|---|---|---|---|---|
| 4.1 | Labelled Links | D-019, D-075 | One Document linked to many Officetul records ("receipt for" Expense #881), no copies | M-own |
| 4.2 | Provider contract | D-075 | Each Officetul app registers a provider (record name, can-open, visibility subquery, link types); registry table; enforced morph map | M-own |
| 4.3 | Documents panel and attach | D-075, Section 16 §6.3 | Panel on record pages; "Add document" files automatically once the file is clean | M-own |
| 4.4 | Record deleted | D-069 | Documents stay; the Link is marked "record deleted", logged | M-own |
| 4.5 | Events and reconciliation | D-075 | After-commit events both ways; label refresh on rename; nightly consistency check | M-own |
| 4.6 | Withdrawal notice | D-072, Section 16 §6.3 | Withdrawing an Official version tells the owning app | M-own |
| 4.7 | "Expenses without an invoice or receipt" | D-021, D-023, D-056, D-072 | Covered only by an Official, non-Trash Document of the right type; only roles that see all finance Folders get it | M-own |
| 4.8 | First providers | D-019, D-021 | Expense (or Bill) and Supplier | M-own |
| 4.9 | Other providers | D-019, catalogue | Customer, Sales invoice, Project, as Officetul has them (owned by each app's maintainer, Section 16 §9) | M-ext |

### 1.5 Epic E5 — Search and OCR

| # | Capability | From | What it means | Tier |
|---|---|---|---|---|
| 5.1 | Search entry point | D-056, D-064 | One swappable interface, MySQL FULLTEXT adapter, access filter inside the query, counts and facets from the same query, shared contract tests | M-own |
| 5.2 | Coverage by Access level | D-065 | View: details and Official version text; Contribute and Manage: Drafts too; Encrypted files: details only | M-own |
| 5.3 | "Include history" and "Search Trash" | D-065 | Superseded and Withdrawn text on request; separate Trash search for Manage holders | S ↓Q2 |
| 5.4 | Identifiers and aliases | D-023, D-066 | TINs and invoice and receipt numbers found with or without dashes; "OR" and "2307" become type filters | M-own |
| 5.5 | Search privacy | D-067 | No suggestions; search terms never stored; daily counts only | M-own |
| 5.6 | Switch measurements | D-064, D-067 | p95 search time, zero-result rate, searchable-text size | M-own |
| 5.7 | OCR | D-049 | OCRmyPDF + Tesseract 5, `eng+fil`, optional `ceb`; text sidecar; no cloud OCR | M-own |
| 5.8 | OCR cap | D-065 | OCR text indexed up to a cap per Version; "partly searchable" badge beyond it | M-own |
| 5.9 | Snippets | Section 13 §6.1 | Snippets built in our code from per-page text | M-own |
| 5.10 | Searchable PDF/A copy | D-049 via Section 07 §6 | A searchable PDF/A Derived file for unsigned scans | S ↓Q2 (see C-06) |
| 5.11 | "OCR quality low" flag | Section 07 §6, §8 | Shown when page confidence is poor | S ↓Q2 |
| 5.12 | Taglish synonyms | Section 13 §6.1 | Product-wide list (*resibo* → receipt) | S ↓Q2 |

### 1.6 Epic E6 — File pipeline and security

| # | Capability | From | What it means | Tier |
|---|---|---|---|---|
| 6.1 | File-format matrix | D-044 | Full support, store-only, or reject, per the matrix | M-own |
| 6.2 | Fail-closed pipeline | D-046, D-047 | Sign → quarantine → validate and hash → ClamAV → promote → process → Inbox; nothing is visible before a clean verdict | M-own |
| 6.3 | Failure policies | D-047, Section 12 §6.2 | Scanner down (hold, alert at 15 min), timeouts, infected (blocked, 30 days, then deleted), corrupt, stale signatures, abandoned uploads | M-own |
| 6.4 | Password-protected files | D-048 | Held; a company admin may accept the risk (logged); "Encrypted" badge; no preview | M-own |
| 6.5 | Tool isolation | D-051 | One container per tool, unprivileged, no network, limits; worker pool apart from web servers; no AGPL tools | M-own |
| 6.6 | Previews | D-044, D-051 | Server-made page images for PDF, images (HEIC, multi-page TIFF), Office (via Gotenberg) and email | M-own |
| 6.7 | Downloads | D-051 | Separate domain, forced download, short-lived link, permission checked every time | M-own |
| 6.8 | Storage layout | D-073, D-075 | Buckets by class with random keys; versioning on Original files; lifecycle rules; "object before row, row before delete"; daily orphan sweep | M-own |
| 6.9 | Shared stored copy | D-070 | One stored Original file per company per fingerprint; never across companies | M-own |
| 6.10 | Encryption | D-059 | Provider keys at rest; TLS everywhere | M-own |

### 1.7 Epic E7 — Activity log and integrity

| # | Capability | From | What it means | Tier |
|---|---|---|---|---|
| 7.1 | Log writer and events | D-060, D-063 | Our own writer; every event in Section 09 §6.1 | M-own |
| 7.2 | Security log | D-060 | Logins, MFA, refused access, Support access | M-own |
| 7.3 | What is logged | D-060 | Downloads and exports one by one; views grouped per person, per Version, per 10 minutes; no content, no search terms | M-own |
| 7.4 | Who sees the log | D-056, D-060 | Changes for Contribute and above; views, IP and device only for Manage holders and company admins | M-own |
| 7.5 | No log, no access | D-060 | A download or preview is refused if its entry can't be written | M-own |
| 7.6 | Insert-only database user | D-061 | Insert-and-read grants, blocking triggers, a separate migration user | M-own |
| 7.7 | Hash chain | D-061 | Per-company chain plus a redactable payload for old/new values and personal details | M-own |
| 7.8 | Log digest | D-061 | Daily digest to compliance-locked storage; nightly chain verification | M-ext |
| 7.9 | Fixity on arrival and move | D-031, D-061 | Re-hash on promote, restore and migration | M-own |
| 7.10 | Scheduled Fixity checks | D-031, D-061 | Rolling 90-day pass plus nightly sample; "Integrity last verified"; mismatch procedure | M-ext |
| 7.11 | Activity page | Section 09 §6.4 | Company-wide page for admins; CSV export | M-own |
| 7.12 | Time | D-063 | UTC `DATETIME(6)`, Manila display, clock-drift alert above 1 s | M-own |
| 7.13 | Log retention | D-062 | Activity log for the account's life; Security log 2 years; app logs 30 days; digests 10 years | M-ext |
| 7.14 | Claim wording | D-029, D-061 | "Tamper-evident for changes after upload" only once 7.7 and 7.8 run in production | M-ext |

### 1.8 Epic E8 — Admin, privacy and exit

| # | Capability | From | What it means | Tier |
|---|---|---|---|---|
| 8.1 | MFA | D-033, D-058 | Authenticator app or passkey, no SMS; required for company admins and Officetul staff; a company switch requires it for everyone | M-ext |
| 8.2 | Session limits | D-058 | 60 min idle, 12 h total; companies may shorten the idle time | M-ext |
| 8.3 | Support access workflow | D-033, D-057 | Request → admin approves with MFA → read-only, 4 h default, auto-revoked, logged; break-glass with two approvers | S ↓Q7 |
| 8.4 | No back doors | D-057 | Any "log in as" feature removed or wrapped; no ad-hoc production database access | M-ext |
| 8.5 | Full company export | D-033, D-054, D-062 | Original files, details, Versions, Links and the Activity log; company admin only | M-ext (ops-run first; self-service S ↓Q2) |
| 8.6 | Exit | D-083 | 30 days read-only export → live deletion within 7 days → deletion certificate naming the last backup expiry | M-ext (as a runbook) |
| 8.7 | Purge ledger and restore marker | D-084 | Deletions recorded outside the database and replayed after any restore; a marker records any lost log period | M-ext |
| 8.8 | Delete categories | D-082 | Clean-up/mistake, disposal or data-subject erasure; only the first can be recovered from backup | M-ext |
| 8.9 | Company settings | D-015, D-049, D-053, D-058 | Cebuano OCR, link types, MFA-for-all switch (approved senders with 1.8) | M-own |
| 8.10 | No secondary use | Section 11 §6, §8 | Client Documents are used only to serve that client (no AI training, no analytics) | M-own (policy) |
| 8.11 | Plan meters | D-086 | Storage (GB) and OCR pages per month per company | S (show usage only; Q9) |

### 1.9 Epic E9 — Ops, backup and monitoring

| # | Capability | From | What it means | Tier |
|---|---|---|---|---|
| 9.1 | Hosting | D-076 | Singapore with Officetul's provider; AWS Singapore for files, worker and inbound email if that provider lacks the storage features | M-own |
| 9.2 | Processing server | D-077 | One dedicated 16 GB server for ClamAV, Gotenberg, Tika and OCR | M-own |
| 9.3 | Database | D-074, D-078 | Managed MySQL 8.4 LTS | M-own |
| 9.4 | First-line recovery | D-073, D-081 | 35-day point-in-time restore; versioning on Original files | M-own |
| 9.5 | Locked backups | D-080 | Separate account, locked vault, second region; daily database and Original files; Log digests replicated; Derived files, quarantine and exports not backed up | M-ext |
| 9.6 | Backup retention | D-081 | Daily 35 days; monthly 120 days | M-ext |
| 9.7 | Restore runbook and drill | D-084, D-085 | Restore order with ledger replay; monthly automatic restore check; first drill with a written record | M-ext |
| 9.8 | Published targets | D-079 | 5 min / 4 business hours; 24 h / 2 business days; confirmed by the first drill | M-ext |
| 9.9 | Alerts | D-047, D-061, D-063, D-087 | Scanner down, stale signatures, queues, Fixity check failures, chain and digest failures, disk, clock drift, backup health | M-own (basic set) |
| 9.10 | On-call | D-087 | A named on-call person and a named backup | M-own |
| 9.11 | Break-glass kit | D-087 | Runbook and break-glass kit held by the senior partner | M-ext |
| 9.12 | Infrastructure as code | Section 15 §9, RK-62 | Needed for a 2-day rebuild in the second region | S (see C-15) |

### 1.10 Epic E10 — Pilot tooling and quality

| # | Capability | From | What it means | Tier |
|---|---|---|---|---|
| 10.1 | Isolation test suite | D-052, Section 12 B3 | Two-company fixtures, route sweep, architecture tests, static-analysis bans, "list equals detail"; failures block every release | M-own |
| 10.2 | Manual two-company check | D-052 | Before each pilot (again before the external one) | M-own |
| 10.3 | Code review | D-013 | Design review against the real Officetul code before any build, then design fixes | M-own (before build) |
| 10.4 | Pilot measures | D-064, D-067, Sections 09 and 18 | Usage counts, event volume, ops hours, OCR backlog | M-own |
| 10.5 | OCR benchmark | D-050 | 240 pages, keyword-hit rate; owner sets the bar in a calibration session | M-ext |
| 10.6 | Search test | D-068 | Taglish, typos, identifier formats, leak checks on the benchmark set | M-ext |
| 10.7 | Pilot client choice | D-012, D-017 | Client document profile filled in; one client chosen | M-ext (owner task) |
| 10.8 | External penetration test | Section 12 B3 | Yearly | S |
| 10.9 | Load test | Section 13 §6.3 | Before about 1 million Documents | C |

### 1.11 Counts per epic

| Epic | M-own | M-ext | Must total | S | C |
|---|---|---|---|---|---|
| E1 Capture and Inbox | 7 | 1 | 8 | 1 | — |
| E2 Documents, Versions and lifecycle | 14 | 1 | 15 | 2 | 1 |
| E3 Folders and permissions | 5 | 3 | 8 | 1 | — |
| E4 Links and ERP contract | 8 | 1 | 9 | — | — |
| E5 Search and OCR | 8 | — | 8 | 4 | — |
| E6 File pipeline and security | 10 | — | 10 | — | — |
| E7 Activity log and integrity | 10 | 4 | 14 | — | — |
| E8 Admin, privacy and exit | 2 | 7 | 9 | 2 | — |
| E9 Ops, backup and monitoring | 6 | 5 | 11 | 1 | — |
| E10 Pilot tooling and quality | 4 | 3 | 7 | 1 | 1 |
| **Total** | **74** | **25** | **99** | **12** | **2** |

---

## 2. Out of scope (deferred by the decisions)

**Verified fact** unless marked. "Next release" is the release after the MVP; "later" has no date.

### 2.1 Next release

| Item | From |
|---|---|
| Approval chains on "mark official", formal rejection, urgent bypass | D-014, D-037, Section 05 §6.3 |
| Review and expiry reminders | D-014, D-021 |
| Retention rules, legal hold, "blocked" state, disposal log, certified deletion | D-014, D-033, D-041 |
| Governance-mode Object Lock with legal hold on Original files | D-073 |
| Person-level search and access report (data-subject requests) | D-033 |
| Share links | D-014 |
| Client portal | D-014, D-022 |
| Document requests (placeholder, assignee, reminders) | D-022 |
| Evidence pack export (Original file, hash, activity extract, draft custodian affidavit) | D-031 |
| RFC 3161 time-stamps on the Log digest; weekly digest email; database audit plugin | D-061, Section 09 §6.2 |
| PNPKI time-stamps, if DICT's service is live | OQ-027 |

### 2.2 Later

| Item | From |
|---|---|
| AI data extraction (metered) | D-014, Section 07 |
| E-signatures | D-014 |
| Law and construction packs | D-014, D-025 |
| "Copy issued from [Company] records", only after lawyer approval | D-030 |
| Password-unlock to a searchable Derived file | D-048 |
| Per-company encryption keys and crypto-shredding, only if a contract requires them | D-059, D-083 |
| Meilisearch, only when a D-064 trigger holds for two weeks | D-064 |
| PaddleOCR, then opt-in Azure Read, only if the benchmark requires them | D-049 |
| Per-user search suggestions | D-067, Section 13 §7 |
| Range filters on type-specific details | D-071 |
| Philippine hosting, only if the lawyer (A12) or a paying client requires it | D-076 |
| Separate OCR server at growth | D-077 |
| Cheaper S3 replication instead of AWS Backup for files at about 1 TB | D-080 |
| Watch-folder agent or browser scanning, only if a paying client insists | D-015 |
| Sandboxed in-browser PDF viewer | Section 06 §8 |
| Signature status display and the Supreme Court verification link | Section 10 §6 |
| Second scan engine, content disarm, gVisor, Inbox rescans, archive unpacking, SIEM export | Section 12 §8 |
| Per-company copy-back tool (manual runbook until then); third backup copy outside AWS | R19 §11 "Later" |

### 2.3 Never (by decision)

Supplying or maintaining scanners (D-016); advising clients to destroy paper originals (D-008); a "Certified True Copy" feature (D-030); SMS as a second factor (D-058); AWS Textract (D-049); AGPL tools such as Ghostscript and MuPDF (D-051); sending client files to VirusTotal (Section 12 §3); per-Document permissions in the MVP (D-053); check-out locking (D-039); automatic purge of Documents (D-041).

---

## 3. Consistency check

**Verified fact** for what each source says; the finding itself is this note's reading.

### 3.1 Contradictions

| # | Finding | Where | Suggested resolution |
|---|---|---|---|
| C-01 | **Scope has outgrown the timeline.** D-005 and D-014 set "4–6 months" for an 8-bullet MVP. D-019 to D-088 then added Confidential folders, a hash chain, locked digests, Fixity checks, Support access, locked cross-region backups, a purge ledger, exit handling and more, with no new timeline check. | D-005, D-014, D-022 rationale | Re-baseline (Q1) and trim (Q2) |
| C-06 | **OCRmyPDF may need Ghostscript.** Section 07 §6 has OCRmyPDF write a searchable PDF/A. D-051 bans AGPL tools, which include Ghostscript. R11 §1 #3 says the sources conflict on whether OCRmyPDF 17 runs without Ghostscript. | D-049, D-051, R11 §(d) | MVP writes the text sidecar only; the PDF/A copy becomes Should and depends on a clean-container test (Q2) |
| C-07 | **Sensitivity.** The catalogue says sensitivity "drives default folder access". D-053 says only Folders decide access. Section 04 cites "Sensitivity field per document (D-026)", but D-026 doesn't mention sensitivity. | Catalogue §1, D-053, Section 04 §6 | Sensitivity is a label that suggests a default Folder at filing; it never grants or removes access (Q10) |
| C-08 | **Access-level wording.** Section 05 §6.4 says View may see drafts "(configurable)"; D-053 gives View the Official version, details and preview only. Sections 05 and 12 let Contribute move its own Drafts to Trash; D-053's text lists Trash under Manage only. | Section 05 §6.4, Section 12 B2, D-053 | Drop "(configurable)"; add "Contribute may move own Drafts to Trash" to D-053 (Q10) |
| C-09 | **Versioning window.** D-073 says Original-file versioning keeps deleted copies "30–90 days". R19 §6.3 recommends 35 days to match D-081. D-083's "live deletion within 7 days" doesn't say that old object versions are deleted too. | D-073, D-081, D-083 | 35 days; exit and erasure delete old object versions as well (Q10) |
| C-10 | **"Recoverable within 120 days" overstates.** D-082 promises recovery within 120 days, but D-081 keeps daily copies only 35 days and then monthly copies. A Document added and permanently deleted between two monthly copies, more than 35 days ago, is in no backup. | D-081, D-082 | Dialog says "may be recoverable for up to 120 days" (Q10) |
| C-11 | **"Nothing in backups older than 120 days"** (D-081) vs Log digests kept 10 years (D-062) and replicated into the backup account (D-080). | D-062, D-080, D-081 | State that Log digests are not backups (Q10) |
| C-28 | **Attach from a record.** Section 05 §6.3 #2 files "as an Official record" every time; D-036 says the default comes from the Document type. A contract attached to a customer would skip Draft. | Section 05 §6.3, D-036 | Use the type default (Q10) |
| C-31 | **"Kept locked 30 days"** for infected files (D-047) reads like Object Lock, which D-073 keeps for Log digests only. | D-047, D-073 | Means "held in quarantine, unreachable"; reword (Q10) |

### 3.2 Gaps: referenced but never decided

| # | Gap | Where | Next step |
|---|---|---|---|
| C-02 | **Which pilot do "4–6 months" mean?** D-005 says "pilot"; D-017 has two pilots; D-032, D-050, D-068 and D-085 gate only the external one. | D-005, D-017 | Q1 |
| C-04 | **Inbox visibility inside a company.** No decision says who sees an Inbox item before it is filed. Section 12 §6.1 says only "visible only to that company". A confidential scan would be seen by everyone with Inbox access, despite D-055 and D-056. Rejected Inbox items have no clean-up rule (D-041 covers Documents only). | D-015, D-055, D-056, R17 `assigned_to` | Q4 |
| C-05 | **OQ-018 (our own mail-submission service) was "Deferred to Phase 7"**, but Round 9 decided nothing on it. Section 01 notes that Microsoft 365 turns off password-only device mail at the end of December 2026, so copiers that send through the client's account may stop working. | OQ-018, OQ-019, D-015 | Q6 |
| C-13 | **Officetul-wide changes have no owner.** Login is shared (D-007), so D-058's MFA and session limits change every Officetul app. Other examples: removing "log in as" (D-057), separate database users (D-061, RK-39), the MySQL 8.4 upgrade (D-074, D-078, OQ-028), an enforced morph map (D-075, RK-52) and server-wide full-text settings (RK-49). No decision says who does this work or whether it counts against the 1–2 developers. | D-007, D-057, D-058, D-061, D-074, D-075, D-078 | Q8 |
| C-16 | **What the full company export contains** is not defined. D-062 says the Activity log is handed over at exit, and D-083 relies on the export. | D-033, D-062, D-083 | Recommendation: Original files, details, Versions with states, Links, Related documents, Capture records and the Activity log with its chain values, as files plus CSV/JSON |
| C-17 | **"Finance Folders" is not a defined thing.** D-056 limits the expenses report to roles that see "all finance folders". | D-056, D-072 | A company-admin flag on Folders (Q10) |
| C-18 | **OCR text cap per Version** has no value. | D-065 | Set from benchmark throughput (D-050); tech default |
| C-26 | **Where the own-company pilot runs.** D-052 asks for a two-company check "before each pilot", but doesn't say whether our pilot runs on production Officetul next to clients' companies. | D-017, D-052 | Q3 |
| C-27 | **Form of phone capture.** D-015 says "phone-camera capture"; R04 §7 recommends the phone's own scanner app plus upload, with an in-browser capture SDK only as a nice-to-have. | D-015, R04 | Phone scanner app plus upload (Q10) |
| C-14 | **Plan meters are decided, enforcement isn't.** D-086 limits storage and OCR pages, but quota sizes wait for pricing (OQ-020), and nothing says whether a full quota blocks uploads. | D-086, OQ-020 | Q9 |
| C-15 | **The disaster target needs infrastructure as code**, which Section 15 §9 calls an Open question (RK-62, questionnaire Q31). Without it, D-079's "2 business days in another region" may not hold. | D-079, Section 15 §9 | Build it, or publish the disaster target only after the first full rebuild (gate item) |

### 3.3 Duplicated, amended or stale entries (housekeeping)

| # | Finding |
|---|---|
| C-19 | **Stale open questions:** OQ-026 (where the Log digest lives) is answered by D-076 and D-080 but still reads "Deferred to Phase 10". OQ-018 still reads "Deferred to Phase 7". |
| C-20 | **Amendment chains not shown on the original rows:** D-014 is amended or extended by D-021, D-023, D-024, D-026, D-031 and D-033, but its row has no pointer. D-033 ("MFA mandatory for vendor admins") is extended by D-058 (required for company admins too) without a note. D-020 is confirmed by D-053 (noted only in D-053). D-021→D-023, D-026→D-045 and D-074→D-078 are already annotated. |
| C-29 | **The Security log is effectively Officetul-wide**, because logins and MFA belong to the shared login (D-060 with D-007). Part of C-13. |

### 3.4 Decisions that depend on unanswered questions

| # | Dependency | Decisions affected | Blocks |
|---|---|---|---|
| C-03 | **OQ-025** (how Officetul decides who may open a record) | D-020, D-053 link-granted View; D-075 provider "can-open" and visibility subquery | Q5 |
| C-12 | **Two people are needed, D-005 allows one developer.** Break-glass needs two Officetul approvers (D-057). The yearly rebuild must be done by the person who didn't write the runbook (D-085). D-087 needs a named backup who also carries the 24-hour breach duty. | D-005, D-057, D-085, D-087 | Q1 |
| C-21 | **Lawyer and tax adviser (D-027) not booked; reviewers and budget not named.** Waiting on them: the claims list "before launch" (D-029); 30-day infected retention (D-047); Support-access terms (D-057, A22); log retention (D-062, A17–A21); Philippine hosting (D-076, A12); backups, exit and restore (D-080 to D-085, A23–A30 / L1–L8); Link labels and erasure (OQ-029); paper-original defaults and the TIN format for D-066 (OQ-021); law-firm questions (OQ-022). | Many | External-client gate |
| C-22 | **OQ-024:** NPC registration as a processor, a named DPO, and who owns the D-032 tasks. | D-032, D-087 | External-client gate |
| C-23 | **D-013 code review and OQ-008 questionnaire.** D-052, D-061, D-064 and D-075 are "subject to D-013 review". Repository access comes only after the senior partner approves the design, so the review falls inside the build window and can change estimates. | D-013, D-052, D-061, D-075 | Start of build |
| C-24 | **OQ-020 pricing** (senior partner). Needed for D-086 quota sizes and for the first client's terms. | D-018, D-086, D-088 | External-client gate (or a written free-pilot term) |
| C-25 | **Client document profile not filled in** (D-012), and our own pilot only runs "if it handles enough paper" (D-017). | D-012, D-017 | Pilot client choice |
| C-30 | **OQ-028** MySQL version and upgrade effort; **OQ-030** Tokyo or Malaysia (named in the data processing addendum, D-080); **Questionnaire Q12** Officetul's hosting (D-076). | D-074, D-076, D-078, D-080 | Own-pilot gate (MySQL, hosting); external gate (backup region) |

**Not blocking:** ISO certification (D-028, OQ-003, OQ-011, OQ-012) has no effect on MVP scope; D-011 only asks that the controls above exist.

---

## 4. Effort estimate

**Estimate** throughout. Units are **developer-weeks** of build, test and review by someone already fluent in Laravel, Inertia and React.

### 4.1 Assumptions

1. Developers are full-time on Documents and don't also maintain the rest of Officetul.
2. Officetul's tenancy, roles, queues and storage are close to what the questionnaire assumes (D-013). A poor fit adds weeks (RK-07, RK-43, RK-51).
3. Officetul already has a working login with Fortify-style two-factor support. If not, add 1–2 weeks to E8.
4. A productive week is about **80%** of a calendar week (meetings, Officetul support, Philippine holidays, the December season).
5. **Contingency: 15%** for integration, bug fixing and pilot feedback. It is shown separately.
6. No credit is taken for AI coding assistants. If the team uses them well, expect the low end.
7. Company tasks (D-032, lawyer, pricing) are listed separately in §4.4 and aren't developer time.

### 4.2 Per epic

"Should" is the extra effort if the items marked S in §1 are built too.

| Epic | M-own | M-ext | Must total | Should (extra) | Main drivers |
|---|---|---|---|---|---|
| E1 Capture and Inbox | 4.5–6.5 | 0–0.5 | 4.5–7 | 1.5–2.5 | Uppy multipart and signing endpoints; Inbox filing screen; scan-to-email (SES, sender lists, DMARC) is the Should part |
| E2 Documents, Versions and lifecycle | 6.5–9 | 1–1.5 | 7.5–10.5 | 0.75–1.5 | Model with write-once triggers; Document page; Document types with JSON-validated fields; presets |
| E3 Folders and permissions | 4–5.5 | 1.5–2 | 5.5–7.5 | 1–1.5 | Guard and composite keys; grant inheritance in one SQL check; panels; link-granted View is the Should part |
| E4 Links and ERP contract | 5–7.5 | 1–1.5 | 6–9 | — | Provider contract inside unseen code; panel and auto-filing; events and reconciliation; expenses report |
| E5 Search and OCR | 5–7.5 | — | 5–7.5 | 1.5–2.25 | Search table and access join; identifier extraction; OCR jobs; contract tests |
| E6 File pipeline and security | 6.5–9.5 | — | 6.5–9.5 | — | Containers and hardening; validate/scan/promote worker with failure states; previews for many formats; download domain |
| E7 Activity log and integrity | 4.5–6.5 | 2–2.5 | 6.5–9 | — | Writer with full event coverage and chain; database grants; digest, verifier, scheduled Fixity checks |
| E8 Admin, privacy and exit | 0.5–1 | 4–6.75 | 4.5–7.75 | 2.5–4 | MFA and sessions; export; exit job and certificate; purge ledger and restore marker; Support-access workflow is the Should part |
| E9 Ops, backup and monitoring | 2–3 | 2.5–4.5 | 4.5–7.5 | 1–2 | Buckets, lifecycle, deploys, alerts; backup account and locked vault; restore runbook and first drill; infrastructure as code is the Should part |
| E10 Pilot tooling and quality | 4–6.5 | 2–3.5 | 6–10 | — | Isolation suite; code review and design fixes; benchmark harness; search test; manual two-company passes |
| **Total (raw)** | **42.5–62.5** | **14–23** | **56.5–85.5** | **8.25–13.75** | |
| **With 15% contingency** | **49–72** | **16–26** | **65–98** | **9.5–16** | |

Could items (accounting-firm pack, load test) add about 0.5–1 more.

### 4.3 Non-feature work inside those figures

| Work | Epic | Developer-weeks | Notes |
|---|---|---|---|
| Two-company isolation test suite (D-052) | E10 | 2–3 | Route sweep, architecture tests, static analysis, "list equals detail"; then kept current every release |
| D-013 code review and design fixes | E10 | 1–2 | Needs repository access; may move other estimates |
| Manual two-company check (D-052) | E10 | 0.5 per pilot | Burp Autorize or OWASP ZAP |
| OCR benchmark (D-050) | E10 | 1–2 dev | Plus **5–8 staff days** for gathering pages and typing reference text, and a 1-hour calibration session with the owner |
| Search test (D-068) | E10 | 0.5–1 | Reuses the benchmark set; includes leak checks |
| Processing server setup (D-051, D-077) | E6 | 1.5–2 | Images, limits, no-network networking, patch routine |
| Restore runbook and first drill (D-084, D-085) | E9 | 1–1.5 | Includes ledger replay and the written drill record |
| Ops runbooks and alerts (D-047, D-087) | E9 | 0.5–1 | Scanner down, held files, Fixity check and chain failures |

**Outside the Documents budget** *(Estimate; Q8)*:

| Officetul platform work | Owner | Developer-weeks |
|---|---|---|
| MySQL 8.4 upgrade, if Officetul runs 8.0 or MariaDB (D-074, D-078, OQ-028) | Officetul maintainer | 0–4 |
| MFA and session limits for all Officetul logins, if missing (D-058) | Officetul maintainer | 0–2 |
| Removing or wrapping "log in as" (D-057) | Officetul maintainer | 0–0.5 |
| Separate migration and app database users (D-061) | Officetul maintainer | 0.25–0.5 |
| Morph-map migration for existing polymorphic columns (D-075, RK-52) | Officetul maintainer | 0–1 |
| **Total** | | **about 0.5–8** |

### 4.4 Company tasks (not developer time)

| Task | From | Who | Effort (Estimate) |
|---|---|---|---|
| NPC registration as a processor; name a DPO | D-032, OQ-024 | Senior partner / owner | 2–5 days of admin, plus NPC processing time (unknown) |
| Standard data processing addendum: 24-hour breach notice, Singapore hosting, backup region, sub-processors | D-032, D-080 | Owner with the lawyer | Lawyer time (not costed); 1–2 days of owner time |
| Publish the sub-processor list | D-032 | Owner | 0.5 day |
| Written incident procedure and one tabletop exercise | D-032, D-087 | Owner and on-call people | 1.5–2.5 days |
| Lawyer and tax adviser sessions (A1–A30, OQ-009, OQ-010, OQ-021, OQ-022) | D-027, D-034 | Senior partner names them; owner attends | 1–2 sessions |
| Client document profile; choose the pilot client | D-012, D-017 | Owner | 0.5–1 day |
| Pricing decision or written free-pilot terms | D-018, D-088, OQ-020 | Senior partner | One meeting (meeting guide 18) |

### 4.5 Fit against the timeline

| Item | Value | Label |
|---|---|---|
| D-005 timeline | Pilot about 4–6 months after scope approval | Verified fact |
| D-017 order | Own company for 2–4 weeks, then one client | Verified fact |
| Capacity, 1 developer, 4–6 months | about 14–21 productive developer-weeks | Estimate |
| Capacity, 2 developers, 4–6 months | about 28–42 productive developer-weeks | Estimate |
| M-own with contingency | 49–72 developer-weeks | Estimate |
| Own pilot start, 2 developers | about **7–10 months** after scope approval | Estimate |
| First external client, 2 developers | about **3–4 months** after that (M-ext is built during the own pilot) | Estimate |
| Own pilot start, 1 developer | about 14–21 months | Estimate |

**Verdict:** it doesn't fit. Even the low end of M-own (49 weeks) is above the high end of two-developer capacity in 6 months (42 weeks). *(Estimate)*

### 4.6 What to move, and what it costs

The S tier in §1 already takes out 8–14 developer-weeks. That is about one to two months for two developers. Proposed moves (Q2, Q5, Q6, Q7):

| Move | Saves (raw) | Trade-off |
|---|---|---|
| Scan-to-email → S (Q6) | 1.5–2.5 | Copiers scan to a USB stick or a PC folder, and someone uploads. Removes the OQ-018 risk from the critical path. |
| Support-access workflow → S; interim "no staff access to content" (Q7) | 1.5–2.5 | Support helps by screen-share only. No break-glass during an incident; the client's admin must act. |
| Link-granted View → S (Q5) | 1–1.5 | An approver needs Folder access to see a receipt from the expense page. Removes the OQ-025 dependency. |
| Self-service export → ops-run export for the first client (Q2) | 0.5–1 | The client asks us for an export instead of pressing a button. |
| Searchable PDF/A copy → S (Q2) | 0.5 | Downloads are the Original files; search still works from the text sidecar. Avoids the Ghostscript question. |
| Include history, Search Trash, Taglish synonyms, low-OCR flag, stale-draft warning, Related documents → S (Q2) | 1.75–3 | Fewer conveniences in the pilots. Trash is still listed and restorable; amendments are linked by a note in the details. |
| Accounting-firm pack → C (Q2) | 0.25 | Accounting firms add their types by hand in the pilot. |
| Plan meters display only; no enforcement (Q9) | 0.5 | No automatic limits until pricing exists. |

**A thinner own pilot** *(Recommendation, if the owner wants an earlier start)*: build Office-format previews, the activity-page filters, page numbers in snippets and the encrypted-file acceptance screen during the own pilot rather than before it. This moves about 3–4.5 weeks from M-own to M-ext and doesn't cut anything:
- M-own becomes about 45–67 weeks with contingency;
- with two developers that is about **6.5–9.5 months**.

Six months is reachable only at the low end of every estimate.

---

## 5. Release gates

### 5.1 Gate 0 — before the build starts

- [ ] Senior partner approves the MVP scope and release boundaries (Master plan Phase 11 exit). This starts the D-005 clock.
- [ ] Senior partner signs off Section 01 (pending since 2026-10-18).
- [ ] ERP technical questionnaire answered (D-013, OQ-008, OQ-025, OQ-028, Q12).
- [ ] Repository access granted; D-013 design review held; D-052, D-061, D-064 and D-075 defaults confirmed or changed.
- [ ] Number of developers confirmed (Q1).

### 5.2 Gate A — start of the own-company pilot

**Technical**
- [ ] All M-own items in §1 built.
- [ ] Automated two-company isolation suite green on the release build (D-052).
- [ ] Manual two-company check done and recorded (D-052).
- [ ] Documents switched on for our company only, if Q3 is accepted.
- [ ] Shared database on MySQL 8.4 LTS (D-074, D-078, OQ-028).
- [ ] Hosting confirmed (D-076, questionnaire Q12): storage with versioning and lifecycle rules; processing server running with tool isolation (D-051, D-077).
- [ ] Point-in-time restore on; Original-file versioning on (D-073, D-081).
- [ ] Basic alerts reach a named on-call person and backup (D-047, D-087).

**Company**
- [ ] Our company has enough paper to make the pilot worthwhile (D-017 revisit condition).
- [ ] Staff know the pilot is internal: no client-facing claims yet (D-029).

**Not required yet** (by decision): D-032 company privacy tasks, the OCR benchmark (D-050), the search test (D-068) and the first restore drill (D-085).

### 5.3 Gate B — first external client

**Technical**
- [ ] All M-ext items in §1 built; blocking bugs from the 2–4-week own pilot fixed (D-017).
- [ ] Isolation suite green; second manual two-company check done (D-052).
- [ ] Hash chain, daily Log digest in locked storage and nightly verification running (D-061). Only then may anyone say "tamper-evident for changes after upload" (D-029).
- [ ] Scheduled Fixity checks running (D-061).
- [ ] Locked backups in a separate account and second region (D-080); OQ-030 settled (Tokyo or Malaysia).
- [ ] Purge ledger and restore marker working (D-084).
- [ ] First restore drill done, with a written record; D-079 targets confirmed or corrected (D-085). The disaster target is published only if infrastructure as code exists (C-15).
- [ ] MFA required for company admins and Officetul staff; session limits on (D-058).
- [ ] "Log in as" removed or wrapped; Support access per D-057, or the interim policy (Q7).
- [ ] Export and exit runbook tested on our own company (D-033, D-083).

**Testing**
- [ ] OCR benchmark run on the real server class; the owner has set and met the bar per document set (D-050).
- [ ] Search test passed, including leak checks: a word that exists only in a Confidential folder looks exactly like a word that exists nowhere (D-068).

**Legal and privacy** (D-032, OQ-024)
- [ ] NPC registration as a processor confirmed; DPO named.
- [ ] Data processing addendum signed with the pilot client: 24-hour breach notice, Singapore hosting, backup region, sub-processors, exit terms.
- [ ] Sub-processor list published.
- [ ] Written incident procedure; named on-call person and backup; runbook and break-glass kit with the senior partner (D-087).
- [ ] Lawyer has reviewed the claims list (D-029) and the addendum. Otherwise the owner records which lawyer items (A7–A10, A17–A30) are accepted as open risks for this pilot.

**Approvals**
- [ ] Pilot client chosen from the client document profile (D-012, D-017).
- [ ] Senior partner: pricing decided, or written free-pilot terms (D-018, OQ-020).
- [ ] D-013 code-review findings closed.

---

## 6. Risks this note adds

| ID | Risk | Mitigation |
|---|---|---|
| RK-68 | **Timeline slips silently** because D-005 still reads "4–6 months" | Re-baseline in Q1; track M-own burn-down weekly |
| RK-69 | **Officetul platform work** (Q8) lands on the Documents developers unplanned | Name the owner and schedule it before Gate A |
| RK-70 | **A confidential scan is seen in a shared Inbox** before filing | Decide Inbox visibility (Q4) before Gate A |
| RK-71 | **The code review (D-013) invalidates estimates** late, because repository access comes after scope approval | Hold the review in the first two weeks of the build; re-estimate right after |

---

## 7. Decisions for the owner

Ten questions, grilling style: each has a recommended answer worded so that "yes" accepts it. **Recommendation** throughout.

### Q1. Re-baseline the timeline

**Recommended:** Yes. Commit **two full-time developers**. Read D-005's "4–6 months" as a target for the **own-company pilot**, re-estimated at **about 8 months after scope approval** (range 7–10), with the first external client about 3 months later.

- **Why:** the Must work (65–98 developer-weeks with contingency) is more than two developers can do in 6 months (about 28–42). One developer also can't meet the two-person duties in D-057, D-085 and D-087 (C-12).
- **Example:** if the senior partner approves the scope on 1 December 2026, our bookkeeper starts filing real receipts around August 2027, and the first client goes live around November 2027.

### Q2. Accept the Should list

**Recommended:** Yes. Move these decided items to Should, built after the first external client unless time allows:
- the stale-draft warning (D-039);
- Related documents (D-040);
- "Include history" and "Search Trash" (D-065);
- Taglish synonyms;
- the "OCR quality low" flag;
- the searchable PDF/A copy (D-049; also avoids the Ghostscript question, C-06);
- a self-service export button (an ops-run export covers the first client).

The accounting-firm pack becomes Could.
- **Example:** Ana in Finance uploads a corrected lease as v3 while v2 is Official. They don't get the "based on an older version" warning in the pilot, but v3 still arrives as a Draft and nothing replaces v2 until a Manage holder marks v3 official.

### Q3. Run our own pilot on production, switched on only for us

**Recommended:** Yes. Our own-company pilot runs on **production Officetul**, with Documents switched on **only for our company**, after the isolation suite passes and one manual two-company check.
- **Example:** our staff file Meralco bills in the real Officetul. ABC Trading, an existing client on the same Officetul, sees no Documents menu, and a request using one of our Document IDs from their login returns "not found".

### Q4. Who sees the Inbox

**Recommended:** Yes. An Inbox item is visible only to:
- the person who captured it;
- people with a new **"Inbox reviewer"** role ability.

Scan-to-email items, when they come, go to the reviewers only. Rejected Inbox items are deleted after **30 days**, and the deletion is logged.
- **Why:** nothing decides this yet (C-04), and a shared Inbox would show confidential scans to everyone before filing.
- **Example:** Rosa in HR photographs a staff member's medical certificate. Until Rosa files it into HR › Confidential, only Rosa and the two Inbox reviewers can see it, not the whole office.

### Q5. Folders alone decide access in the MVP

**Recommended:** Yes. Build Links without the "grants view" switch (D-020, D-053). In the MVP, access comes **only from Folders**. Add the switch after OQ-025 (how Officetul decides who may open a record) is answered.
- **Why:** the switch can't be designed without OQ-025 (C-03), and it carries the leak risk in RK-13 and RK-46.
- **Example:** Marco approves Expense #881 but has no access to Finance › Receipts. On the expense page Marco sees no Document at all (no hint that one exists) until the admin gives Marco's role View on that Folder.

### Q6. No mail service of our own; scan-to-email after the pilots

**Recommended:** Yes. Close OQ-018 for the MVP: **we don't run our own mail-submission service**. Scan-to-email moves to Should. We build the per-company address once the pilot client's copier check (OQ-019) shows their devices can actually send to it.
- **Why:** Microsoft 365 turns off password-only device mail at the end of December 2026, so copiers that send through a client's account may stop working (C-05).
- **Example:** the pilot client's office copier sends through their Microsoft 365 account and stops working in January 2027. Their staff scan to a USB stick and drag the PDFs into the Inbox, and nothing we built is wasted.

### Q7. First client: no staff access to content

**Recommended:** Yes. For the first external client, **Officetul staff have no access to Documents content at all**. Help is by screen-share with the client's admin. Any "log in as" feature is disabled for Documents. The full D-057 Support-access workflow is built before the **second** external client.
- **Trade-off:** no break-glass access during an incident; the client's admin must act.
- **Example:** Carlo at the client can't find a filed contract. Our support person joins a screen-share and guides Carlo; they never open the file themselves.

### Q8. Officetul-wide changes belong to the Officetul maintainer

**Recommended:** Yes. Treat these as **Officetul platform work**, owned and estimated by the Officetul maintainer (about 0.5–8 developer-weeks, §4.3), scheduled before Gate A:
- MFA and session limits (D-058);
- removing "log in as" (D-057);
- separate database users (D-061);
- the MySQL 8.4 upgrade (D-074, D-078);
- the morph-map migration (D-075).

They apply to **every** Officetul company, not only Documents users.
- **Example:** when the MFA rule lands, the admin of a client that never uses Documents must also set up an authenticator app at their next login.

### Q9. Plan meters: show usage, don't block

**Recommended:** Yes. In the MVP, **measure and show** storage and OCR pages per company, on the admin page and in our ops report. The only hard limit is the 1 GB file cap (D-045). **No plan enforcement or billing** until the senior partner decides pricing (OQ-020).
- **Example:** Ana's firm uploads 40 GB of old files in the first week. The admin page shows "40 GB used", nothing is blocked, and we offer back-file scanning as a separate project (D-086).

### Q10. Housekeeping (one yes accepts all)

**Recommended:** Yes to every item below:
- (a) **Sensitivity** is a label that suggests a default Folder at filing; it never grants or removes access (C-07).
- (b) Drop "View: drafts (configurable)" from Section 05; add "Contribute may move own Drafts to Trash" to D-053 (C-08).
- (c) Original-file **versioning window 35 days**; exit and erasure also delete old object versions (C-09).
- (d) The permanent-delete dialog says "**may** be recoverable for up to 120 days" (C-10).
- (e) Log digests are **not backups**, so the 120-day cap doesn't apply to them (C-11).
- (f) "**Finance Folders**" for the expenses report is a flag a company admin sets on Folders (C-17).
- (g) **Phone capture** = the phone's own scanner app plus upload; no in-browser capture SDK (C-27).
- (h) Attaching from a record uses the **Document type's default** (Official record or Draft), not always Official (C-28).
- (i) In D-047, "kept locked" means held in quarantine, not Object Lock (C-31).
- (j) Close OQ-026 (answered by D-076, D-080); record the Q6 outcome on OQ-018; add amendment pointers on D-014 (D-021, D-023, D-024, D-026, D-031, D-033) and D-033 (D-058) (C-19, C-20).
- (k) The OCR text cap per Version is set from benchmark throughput (C-18).

**Example:** HR files a BIR 2316. The SPI label suggests HR › Confidential, but if someone files it in General › Admin, the Folder's access applies. The label alone hides nothing, so the "Who can see this?" panel tells the truth.

---

## 8. Sources

All internal, read 2026-10-21:
- [DECISION-LOG.md](../DECISION-LOG.md) D-001 to D-088;
- [OPEN-QUESTIONS.md](../OPEN-QUESTIONS.md) OQ-001 to OQ-031;
- [DISCUSSION-TRACKER.md](../DISCUSSION-TRACKER.md);
- [MASTER-PLAN.md](../MASTER-PLAN.md);
- [PROJECT-CONTEXT.md](../PROJECT-CONTEXT.md);
- [GLOSSARY.md](../GLOSSARY.md);
- Sections [01](../sections/01-business-problem-and-opportunity.md), [02](../sections/02-market-and-competitor-research.md), [03](../sections/03-philippine-sme-workflows.md), [04](../sections/04-iso-certification-strategy.md), [05+08](../sections/05-document-lifecycle-and-workflows.md), [06](../sections/06-upload-and-file-format-support.md), [07](../sections/07-ocr-and-document-understanding.md), [09](../sections/09-audit-trails-and-integrity.md), [10](../sections/10-authenticity-and-certified-true-copies.md), [11](../sections/11-philippine-legal-and-regulatory-research.md), [12](../sections/12-security-and-antivirus.md), [13](../sections/13-database-and-search-architecture.md), [14](../sections/14-storage-and-server-requirements.md), [15](../sections/15-backup-retention-and-disaster-recovery.md), [16](../sections/16-erp-integration-and-data-model.md), [18](../sections/18-cost-estimates-and-operating-model.md);
- [document-type catalogue](../appendices/document-type-catalogue.md);
- [ERP technical questionnaire](../questionnaires/erp-technical-questionnaire.md);
- research notes R04 §6–§7, R11 §1 and §(d), R15 §4, R17 §2–§4, R19 §5–§6 and §11.

No external sources were consulted. Effort figures are this note's own Estimates, except the Meilisearch switch cost (2–4 developer-weeks, R16, D-064) and the per-company copy-back tool (1–2 developer-weeks, R19 §5.1), which are excluded from the MVP.
