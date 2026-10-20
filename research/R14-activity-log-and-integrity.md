# R14 — Activity log and file integrity

- **Status:** First pass, design-level. Research only, no code. Defensive design research for Officetul Documents. We have **no access to the Officetul codebase yet**, so every recommendation is written to work with whatever tenancy and logging Officetul already has; §10 lists what to check in the code.
- **Researched:** 2026-10-20. **All sources accessed 2026-10-20.**
- **Question:** How should Officetul Documents keep an activity log that a client, an auditor or a court can trust, and show that files were not altered, at SME cost?
- **Labels:** **Verified fact** (read in the primary source, or tested directly on 2026-10-20), **Vendor claim**, **Secondary** (news, law-firm article or search excerpt only), **Estimate**, **Open question**, **Recommendation** (proposed, not approved). Source IDs `[S#]` point to §14.

**Terminology.** GLOSSARY.md does not yet define "activity log" or "audit trail". D-014, Section 05 §6.5 and Section 10 all say **activity log**, so this note uses:

- **activity log** for the tenant-facing record of what happened to Documents;
- **security log** for the vendor-side record of logins, MFA, failed access and support access;
- "audit trail" only when quoting BIR or vendor texts.

Glossary entries are proposed in §13 (decision 1).

**Access caveats.**

- privacy.gov.ph blocks automated access, so NPC texts were downloaded from Internet Archive copies and their PDF text extracted.
- AWS's own QLDB pages now return 404 or redirect, so the QLDB end date is **Secondary**.
- S3 Singapore prices come from the AWS Price List API JSON (S3 list published 2026-10-15; data-transfer list 2026-09-16), not the rendered pricing page.
- Wasabi's page defining its lock modes returned 404.
- The PNPKI time-stamp endpoint, FreeTSA, DigiCert, `time.aws.com` and PAGASA's NTP host were tested with live requests on 2026-10-20.
- SHA-256 speed was measured on an Apple M1 development machine, not a server.

---

## 1. What to log

### 1.1 Baseline guidance

| Source | What it says | Label |
|---|---|---|
| OWASP Logging Cheat Sheet | Record "when, where, who and what": log time and event time, interaction ID, application ID and address, source address, user identity, type, severity, action, object, result status. Always log authentication successes and failures, authorisation failures, and "higher-risk functionality" such as "Data import and export", file uploads and admin actions. | **Verified fact** [S1] |
| OWASP: exclusions | Session IDs, access tokens, passwords, connection strings, keys and payment data should be "removed, masked, sanitized, hashed, or encrypted". "Never log data unless it is legally sanctioned." | **Verified fact** [S1] |
| OWASP: protection | "Build in tamper detection so you know if a record has been modified or deleted." "Store or copy log data to read-only media as soon as possible." "Synchronize time across all servers and devices." | **Verified fact** [S1] |
| ASVS 5.0 V16 (all L2) | 16.1.1: keep an inventory of logging (event types, storage, access, retention). 16.2.1: when/where/who/what. 16.2.2: synchronised time sources; "use UTC or include an explicit time zone offset". 16.3.1: log all authentication, success and failure. 16.3.2: log failed authorisation. 16.4.2: logs "cannot be modified". 16.4.3: send logs to "a logically separate system". 16.2.5: log sensitive data only as its protection level allows. | **Verified fact** [S2] |
| NDSA Levels v2.0, Control row | L3: "Maintain logs and identify the human and software agents that performed actions on content". L4: "Perform periodic review of actions/access logs". | **Verified fact** [S70] |

### 1.2 Event catalogue (**Recommendation**)

This builds on the Section 05 §6.5 list and adds the events needed by D-033, D-041, D-047 and D-048.

| Group | Events | Extra fields | Log |
|---|---|---|---|
| Capture | Inbox item captured; scan verdict; held (encrypted, D-048) and risk accepted; rejected; blocked as infected (D-047) | Channel, sender, email Message-ID, SHA-256, size, scan engine and signature version | Activity |
| Filing and versions | Filed (Official or Draft, D-036); version added (with "based on" version, D-039); marked official, withdrawn (reason), restored (D-037, D-038); duplicate warning overridden (D-042) | Version ID, SHA-256, note or reason | Activity |
| Details and connections | Details changed (old → new, D-043); link added or removed (D-019); related document added or removed (D-040); moved between folders | Field name, old and new value, target record | Activity |
| Access | Viewed or previewed (coalesced, §1.4); downloaded; exported (single, bulk or full tenant, D-033); share link created, revoked or used (next release) | Version ID; for exports, the item count and the export file's SHA-256 | Activity |
| Permissions | Folder permission changed; role or user changes affecting Documents; link-visibility setting changed (D-020); download or export permission changed (D-033) | Before and after | Activity |
| Deletion and blocking | To Trash; restored from Trash; deleted permanently (reason, D-041); blocked or unblocked, legal hold set or released (next release) | Title snapshot and hashes on permanent delete (it becomes the disposal record) | Activity |
| Integrity | Fixity check failed; fixity run summary (§6) | Expected and found hash, storage version ID | Activity (failures and summaries only) |
| Authentication | Login success or failure; MFA challenge passed or failed; MFA enrolled or reset; password reset; session revoked; MFA enforcement setting changed | Factor type, failure reason (never the password) | Security |
| Failed access | Authorisation denied on a document; cross-tenant ID attempt; expired-link reuse | Requested object ID, rule that denied it | Security; cross-tenant attempts alert vendor ops (R13 §8) |
| Vendor support | Support access granted, started, ended or expired; **every action during a support session** | Grant ID, who granted it, scope, expiry, staff member | Both: the tenant sees it in the activity log and the vendor keeps it in the security log |
| The log itself | Log exported; log verification run and its result | Range, hashes | Activity |

**Not logged:** browser printing (the browser can't report it reliably). In-app "print" is logged as a download.

### 1.3 Fields on every event (**Recommendation**, built on [S1][S2])

| Field | Notes |
|---|---|
| `event_id` | ULID or UUID |
| `tenant_id`, `tenant_seq` | Per-tenant order. The hash chain (§2) follows `tenant_seq`, not the clock. |
| `occurred_at` | Server time in UTC, microseconds (§7). **Never client-supplied.** A phone's own capture time (EXIF) is stored separately and labelled as such. |
| `actor_type` | `user`, `vendor_support`, `system` (scanner, OCR, fixity job), `api`, `share_link_guest` (later) |
| `actor_id` and display-name snapshot | The snapshot keeps the history readable after a user is renamed or removed |
| `support_grant_id` | Set on every action taken under support access (D-033) |
| `auth_context` | MFA used yes/no; a hash of the session ID (never the raw ID, [S1]) |
| `ip`, `user_agent` | User agent truncated (owen-it uses 1023 characters [S56]). IP is personal data, so it is shown only to company admins (§5). |
| `request_id`, `channel` | Web, email, mobile, API; ties events to server logs |
| `action`, `object_type`, `object_id`, `document_id`, `version_id` | IDs, not names, except the snapshots noted above |
| `file_sha256` | For any event about a specific file version |
| `result` | `success`, `denied`, `failed` |
| `reason` | Required for withdraw, permanent delete and risk acceptance |
| `payload_ref`, `payload_hash` | Old and new values and other detail live in a separate payload table (§1.4) |
| `prev_hash`, `hash` | Hash chain (§2), written by the sealing job |

### 1.4 Data minimisation (**Recommendation**)

1. **Never log document content, OCR text, previews, presigned URLs, tokens or passwords** [S1].
2. **Search:** log that a search ran and how many results came back, **not the query text**, which often contains people's names. M-Files logs "Search initiated" only in its paid User Action Log [S77].
3. **Old and new values (D-043)** may contain personal data such as a TIN or a name. Store them in a separate `activity_payloads` table. The event row keeps only `payload_hash = SHA-256(random salt ‖ payload)`.
   - **Why:** under an erasure, blocking or exit instruction, the payload can be erased and replaced with a "redacted on [date] by [actor], reason" marker without breaking the hash chain.
   - The salt stops anyone guessing short values (such as a TIN) from the hash.
   - The redaction is itself an event. **Estimate:** the same pattern as Certificate Transparency, where the tree commits to hashes, not data [S41].
4. **Coalesce views and previews:** one "viewed" event per user per version per 10 minutes, with a count. Downloads and exports are always logged one by one. Microsoft drops repeat FileAccessed events for the same user and file for 5 minutes, and warns that preview versus access "isn't a guarantee of user intent" [S79]. **Estimate:** without coalescing, preview events would dominate log volume.
5. **No log, no access.**
   - A download or preview URL is issued **only after** its event is written. If the write fails, the request is refused (fail closed, as in D-047).
   - State-changing events are written **in the same database transaction** as the change, so there is never a change without a log entry or a log entry without a change.

---

## 2. Tamper-evidence options

### 2.1 Who could alter the log, and what stops them (**Estimate**)

| Threat | Grants (A) | Triggers (B) | Hash chain (C) | Locked external digest (D/E) | Third-party timestamp (F) |
|---|---|---|---|---|---|
| App bug or SQL injection through the app account | **Stops** | Stops | Detects | Detects | Detects |
| Tenant admin (even a malicious one) | Stops (no UI or API path) | — | Detects | Detects | Detects |
| Vendor developer or DBA with database access | No | Weak (can drop the trigger) | Detects edits in the middle, **not** a full rewrite | **Detects a rewrite** of anything already anchored | Detects |
| Attacker or insider with cloud root | No | No | No | Compliance mode: object can't be deleted, even by root [S20] | **Detects** (the token is held outside our control) |

The conclusion: **A and C are cheap and catch most problems.** Only an external anchor (D/E/F) protects against the vendor itself, and that is the protection a court or auditor will ask about.

### 2.2 Options compared

| # | Option | How it works | Limits | Cost / complexity | Label |
|---|---|---|---|---|---|
| A | **Append-only by DB grants** | The app's MySQL user gets `SELECT, INSERT` on the log tables and nothing else. MySQL supports table-level grants such as `GRANT SELECT, INSERT ON mydb.mytbl` [S24][S25]. TRUNCATE "requires the DROP privilege" [S28], so withhold DROP, ALTER and TRIGGER too. Migrations run as a **separate** user. | No protection against whoever holds the migration or admin user. On RDS the master user has DROP and TRIGGER [S36]. | Free. Needs a second DB user and a deploy change. | Mechanism **Verified fact** [S24][S28][S36]; design **Recommendation** |
| B | **Blocking triggers** | `BEFORE UPDATE` / `BEFORE DELETE` trigger that runs `SIGNAL SQLSTATE '45000'`. The statement fails and transactional tables roll back [S26][S27]. | Anyone with TRIGGER can drop it [S26]. TRUNCATE "does not invoke ON DELETE triggers" [S28]. | Free, low effort. Belt and braces only. | **Verified fact** [S26][S27][S28] |
| C | **Per-tenant hash chain** | `hash = SHA-256(prev_hash ‖ fixed-order event fields ‖ payload_hash)` in `tenant_seq` order (Schneier & Kelsey 1999 [S43]; Crosby & Wallach 2009 [S42]). **Sealed asynchronously:** one queue job per tenant reads new events in order and **inserts** chain rows into an insert-only `activity_seals` table, so event rows are never updated and requests aren't slowed by a chain lock. | Edits or deletions after sealing are detected. Events deleted in the seconds before sealing are not. MySQL auto-increment gaps are normal, so the chain, not the ID, is the record. A full rewrite of the chain by someone with database access goes undetected without D/E/F. | Low: one table, one job, a verifier. | **Recommendation**; window **Estimate** |
| C+ | **Per-document link** | Each event also stores the hash of the previous event **for the same document**. A one-document extract (Evidence pack, D-031) can then be checked on its own, without the whole tenant log. | Adds one column | Very low; awkward to add later | **Recommendation** |
| D | **Periodic digest (Merkle root)** | Daily, compute a Merkle tree over all tenants' chain heads (RFC 9162 tree hashing, with leaf and node prefixes for second-preimage resistance [S41]). One root per day for the whole platform; each tenant gets an inclusion proof. | Needs somewhere trusted to put the root (E or F) | Medium. Simpler MVP form: a daily file listing each tenant's latest `tenant_seq` and `hash`. | **Verified fact** (RFC) [S41]; design **Recommendation** |
| E | **Copies to Object Lock storage** | (1) Daily digest file to a **compliance-mode** bucket (it holds only hashes and tenant UUIDs). (2) Daily per-tenant log export (gzipped JSONL, encrypted with a per-tenant key) to a locked bucket, which is also a recoverable copy. | Compliance mode: "can't be overwritten or deleted by any user, including the root user", and the period can't be shortened [S20]. That clashes with deleting personal data at exit (§5), hence the split: digests in compliance mode; personal-data exports in governance mode, or compliance mode with a per-tenant key you can destroy ("crypto-shredding"). | Very low storage cost (§9) | Lock behaviour **Verified fact** [S20][S21]; split **Recommendation** |
| F | **RFC 3161 timestamp on the daily root** | Send the root's hash to a time-stamp authority (TSA). The token shows "a datum existed before a particular time" [S44]. Use two TSAs for redundancy. | Proves the time a hash existed, not who wrote it or that the log is complete. Free TSAs offer no SLA (§2.4). | Very low: one HTTP POST a day plus `openssl ts -verify` [S49] | **Verified fact** [S44][S49]; **Recommendation** |
| G | **OpenTimestamps (Bitcoin)** | Free calendar servers aggregate hashes into Bitcoin. Confirmation "takes a few hours". To verify "you need a local Bitcoin Core node" [S54]. | No Philippine or eIDAS legal standing; hard to explain to an SME or a judge | Free; odd to operate | **Vendor claim** / **Verified fact** (README) [S54] |
| H | **MySQL binary log** | On by default in 8.4; expires after 30 days by default [S29] | **Not a tamper record.** A privileged session can `SET sql_log_bin=OFF`; BINLOG_ADMIN can purge it; SELECTs are not logged [S29][S30]. | Keep it for point-in-time recovery only (on RDS it is on when backup retention > 0 [S35]) | **Verified fact** [S29][S30][S35] |
| I | **AWS QLDB** | Managed ledger database | **Ended 31 July 2025.** AWS pointed users to Aurora PostgreSQL. Not an option. | — | **Secondary** [S23] |
| J | **Database-level audit plugin** | Records DBA and vendor-staff queries on production. MySQL Enterprise Audit is commercial [S31]. RDS offers the MariaDB Audit Plugin as an option for 5.7, 8.0 and 8.4 [S34]. Percona has an Audit Log Filter component for 8.4 [S32]; neither its page nor the MariaDB plugin page [S33] states a licence. | Watches people, not the log's integrity. Fits NPC 2023-06 s.12 (database access only through "authorised software"). | Low on RDS; next release | **Verified fact** [S31][S32][S33][S34] |

### 2.3 Object Lock stores in or near the Philippines

| Store | Region near PH | Object Lock / WORM | Price (published) | Label |
|---|---|---|---|---|
| **AWS S3** | Singapore (ap-southeast-1) | Compliance and governance modes plus legal hold. Needs versioning. Can be enabled on existing buckets; can't be disabled afterwards [S20][S21]. | Standard $0.025/GB-month (first 50 TB); GET $0.0004 per 1,000; PUT $0.005 per 1,000; internet egress $0.12/GB (first 10 TB) [S38][S39] | **Verified fact** |
| **Wasabi** | Singapore | Object Lock "only during bucket creation"; versioning required. Compliance and governance modes are not confirmed on a primary page. Not confirmed for the Singapore region. | $7.99/TB-month; 90-day minimum; "No fees for egress or API requests" | Lock **Verified fact**; modes **Secondary**; price **Vendor claim** [S60] |
| **Akamai / Linode** | Singapore exists; lock there **not confirmed** | GOVERNANCE and COMPLIANCE modes; set at bucket creation; S3 API only | Not checked | **Verified fact** [S62]; region **Open question** |
| **Alibaba Cloud OSS** | **Manila, ap-southeast-6** | Bucket WORM retention: no one, "including the bucket owner", can modify or delete; once locked it can only be extended | Not checked | **Verified fact** [S63]; Manila WORM **Open question** |
| **Huawei Cloud OBS** | Philippines region launched 3 Dec 2024 | WORM (compliance-only per help excerpts) | Not checked | Region **Verified fact** [S65]; WORM **Secondary**; PH availability **Open question** |
| **Google Cloud Storage** | No PH region (Singapore, Jakarta, Bangkok) | Bucket Lock: can't remove or shorten once locked | Not checked | **Verified fact** [S64] |
| **Backblaze B2** | **No Asia region** | Compliance, governance, legal hold | From $6.95/TB-month | **Verified fact** / **Vendor claim** [S61] |
| **Cloudflare R2** | Location hint only ("best effort") | **Not true WORM:** anyone with a token that can edit bucket config can remove the lock rules | — | **Verified fact** [S66] |
| **DigitalOcean Spaces** | Singapore (SGP1) | Probably unsupported; the docs disagree | — | **Open question** [S67] |
| **Philippine providers** (ePLDT/VITRO, Converge, Globe, PLDT) | — | No published S3-compatible Object Lock found. Converge announced a "sovereign cloud" in June 2026 with no object-storage detail. | — | **Open question** / **Secondary** [S68] |

**Recommendation:**

- **If hosting is AWS Singapore,** use S3 Object Lock.
- **If hosting is a local data centre without Object Lock,** keep the daily digest in a small compliance-mode S3 Singapore bucket anyway, plus RFC 3161 tokens.
  - The digest holds only hashes, so the cross-border personal-data question barely arises (**Estimate**; lawyer to confirm, §12).

### 2.4 Time-stamp authorities

| TSA | Facts | Label |
|---|---|---|
| **FreeTSA.org** | "a free Time Stamp Authority"; "No Service Level Agreement (SLA) is offered"; at most one request per second per IP. Live test granted (Würzburg, DE). | **Verified fact** [S46] |
| **DigiCert** (`timestamp.digicert.com`) | Offers an RFC 3161 server; TSA certificates rotated at least every 15 months. **No free-use terms or rate limits stated.** Live test granted. | **Vendor claim**; terms **Open question** [S47] |
| **Sectigo** | Asks scripted users to wait 15 s or more between requests; terms not stated | **Vendor claim** [S48] |
| **PNPKI (DICT) TSA** | A "PNPKI Time-Stamping Authority – Time-Stamp Policy/Practice Statement" v1.0 exists, effective 4 Sep 2015. It follows RFC 3161 and RFC 3628, with "accuracy of ± 1 second", clocks synchronised "with UTC or the Philippine Standard Time (Republic Act No. 10535)", and an expected token validity of 10 years. It disclaims warranties of availability or accuracy and lists no fees. The adopting ICTO circular's number is blank ("No. 2015 - ____"). **A test request to the published endpoint returned HTTP 404 on 2026-10-20.** | Policy **Verified fact** [S50][S51]; endpoint 404 **Verified fact** (test) [S52]; whether it is live and open to private users **Open question** |
| **Private Philippine CAs** | No DTI-accredited CA offering timestamping was found. This is "not found", not "proven absent". | **Open question** |
| **SC eNotarization rules** | A.M. No. 24-10-14-SC does not mention timestamping or PNPKI. It requires the notarial book to record "the date and time" and facilities to be "tamper-evident". | **Verified fact** [S53] |

**Recommendation:** in the next release, timestamp the daily root with **FreeTSA plus one commercial endpoint** after reading its terms. Retry with DICT once the PNPKI TSA is confirmed live, because a Philippine government TSA would be the easiest to explain in a local court (**Estimate**).

---

## 3. Laravel packages

| Package | Latest | Laravel 13 | PHP | Licence | Tamper-evidence | Notes | Label |
|---|---|---|---|---|---|---|---|
| **spatie/laravel-activitylog** | 5.1.3 (2026-10-19) | Yes (`^12.0 \|\| ^13.0`) | ^8.4 | MIT | None found | v5 table: `log_name, description, subject, causer, event, attribute_changes, properties`. **`batch_uuid` and batches were removed in v5.** | **Verified fact** [S55] |
| **owen-it/laravel-auditing** | v14.0.6 (2026-06-20) | Yes (`^11\|^12\|^13`) | ^8.2 | MIT | None | `audits`: user, event, auditable, `old_values`, `new_values`, `url`, `ip_address`, `user_agent`, `tags`. Pluggable drivers; redactors for sensitive attributes. | **Verified fact** [S56] |
| **laravel-chronicle/core** | v1.15.0 (2026-10-11) | Yes | ^8.2 | MIT | SHA-256 chain, signed checkpoints, signed exports | About 19k installs, 126 stars | Version **Verified fact** [S57]; features **Secondary** |
| cboxdk/laravel-audit-chain, yammi-audit-log | v0.1.0 (2026-09), 0 stars / 68 stars | Yes | — | MIT | Chain (yammi's is off by default) | Too new or too small to depend on | **Verified fact** / **Vendor claim** [S57] |

**Key limit for all model-event packages:** "When issuing a mass update via Eloquent, the `saving`, `saved`, `updating`, and `updated` model events will not be fired". Mass deletes likewise do not dispatch `deleting`/`deleted` [S58]. Model-change auditing therefore silently misses bulk operations. These packages also don't log **reads** (views, downloads), which are half of a DMS log. **Verified fact** [S58]; consequence **Estimate**.

**Recommendation:**

- **If Officetul already uses one of these packages,** keep it for ERP-wide record auditing.
- **Documents gets its own small, explicit event writer.** Each controller and job calls `Activity::record(...)` inside its transaction, writing to insert-only tables (`document_activity`, `activity_payloads`, `activity_seals`).
  - **Estimate:** the writer, sealer and verifier are small. Chronicle's approach is worth reading before building.
- **Laravel 13** (17 Mar 2026; PHP ≥ 8.3; security fixes until 17 Mar 2028) [S59].

---

## 4. Philippine requirements and expectations

**This is research, not legal advice.** Lawyer questions are in §12.

### 4.1 Data privacy

| Rule | Text | What it means for the log | Label |
|---|---|---|---|
| **NPC Circular 2023-06 s.29, "Logs Retention"** | A PIC or PIP "shall retain logs as long as deemed necessary and appropriate based on best practices and industry standards". The retention period depends on "the type of log and its corresponding use". "Security logs that record information about authentication attempts and security incidents shall be retained for longer periods than general system logs". Incident logs are kept "for a specified period required by the NPC, even if it exceeds their retention policies". "PICs shall implement backup and archive mechanisms for their logs." **No number of days or years is given.** | We set and document the periods ourselves (§5). The security log is kept longer than app logs. Log backup or archive is required: the locked export (E). | **Verified fact** [S3] |
| 2023-06 s.13 | Access control "on a 'need to know' basis… authentication methods and regular monitoring" | Per-folder access plus a reviewable log | **Verified fact** [S3] |
| 2023-06 s.19 | For **physical** files, a log of "which file was accessed, including when, where, and by whom", and whether copies were made | Applies to paper originals. The optional paper-original check-out log could reuse the activity log (later). | **Verified fact** [S3] |
| 2023-06 s.12 | Databases accessed or changed only through authorised software | Vendor database access through logged tooling (option J) | **Verified fact** (via R10) [S3] |
| NPC Circular 16-03 s.7(B) | Security measures include "Access control and secure log files" | — | **Verified fact** [S4] |
| 16-03 ss.9, 22 | Breach actions "properly documented". All incidents documented in written reports, made available on request; "a summary of all reports shall be submitted to the Commission annually". No retention period stated. Breach-notification procedure later supplemented by NPC Advisory 2026-02 [S19]. | The security log feeds incident reports | **Verified fact** [S4]; advisory **Secondary** [S19] |
| IRR ss.26(e)(3), 28(d) | Policies for "access management, system monitoring"; "Regular monitoring for security breaches". The word "logs" does not appear in ss.25–29. | — | **Verified fact** [S5] |
| IRR s.44 | The PIP must "allow for and contribute to audits, including inspections" by the PIC or its auditor | The log export is how we let clients audit us | **Verified fact** [S5] |
| **RA 10173 s.16(c)(7)** | Data subjects may have access to the "Date when his or her personal information concerning the data subject were last accessed and modified" (mirrored in IRR s.34(c)(7)) | The client (PIC) needs per-document access and change dates to answer requests. The activity log provides them; a person-level report is next release (D-033). | **Verified fact** [S7][S5] |
| NPC Circular 2022-04 | Nothing on logs; only that PICs and PIPs keep "a record of their data processing activities" | — | **Verified fact** [S6] |
| NPC CCTV guidance (Circular 2024-02, replacing Advisory 2020-04) | Reportedly "only as long as necessary", with no fixed days | Shows the NPC's principle-based style | **Secondary** [S18] |

### 4.2 Tax (context only: Officetul Documents is **not** a CAS)

| Rule | Text | Relevance | Label |
|---|---|---|---|
| **RR 9-2009**, "Maintenance, Retention, and Submission of Electronic Records" | s.5.3.2: "The electronic records must show an audit trail from the source document(s)… to the summarized financial accounts"; web logs and emails "could be an important part of the audit trail". s.5.4.2(c): documentation of "internal controls used to prevent unauthorized addition, alteration, or deletion of retained records". s.5.6.1: "an accurate chronological record of the changes". No per-field audit-trail list is prescribed. | Source documents linked to ERP entries (D-019), with logged link and unlink events, help a client **show** that trail. We never claim "BIR-compliant" (D-029). | **Verified fact** [S12] |
| NIRC s.235 as amended by EOPT (RA 11976) | Books and records kept "for a period of five (5) years" from the day after the filing deadline | Log entries about tax documents should outlive the documents' 5-year minimum (§5) | **Verified fact** [S11] |
| CAS registration changes | RMC 10-2020 suspended permits; RMC 5-2021 replaced the Permit to Use with registration; RMO 9-2021 simplified the guidelines | Not ours to meet | **Secondary** [S14][S15] |
| E-invoicing | RMC 98-2026 (22 Sep 2026) on e-invoices under RR 8-2022 and RR 11-2025 as amended by RR 26-2025. It says nothing on audit trails. | Watch only | **Verified fact** (existence) [S13]; RR contents **Secondary** [S16][S17] |

### 4.3 Electronic evidence

| Rule | Text | Feature | Label |
|---|---|---|---|
| REE Rule 5 s.2 | A private e-document's authenticity is proved by a digital signature, or by "other appropriate security procedures or devices as may be authorized by the Supreme Court or by law", or "(c) by other evidence showing its integrity and reliability to the satisfaction of the judge" | Hash, fixity results and the activity log are "other evidence" under (c) | **Verified fact** [S8] |
| REE Rule 7 s.1–2 | Weight factors: "the manner or method in which it was generated, stored or communicated, including… controls, tests and checks for accuracy and reliability"; "integrity of the information and communication system". s.2(a): system "operated in a manner that did not affect the integrity". s.2(c): recorded "in the usual and ordinary course of business by a person who is not a party". | Fixity checks are "tests and checks". The verified chain shows system integrity. An external anchor held by a non-party (AWS lock, TSA) **may** help under s.2(c) (**Estimate**; lawyer question). | **Verified fact** [S8] |
| REE Rule 8 s.1–2 | Business records made "at or near the time" in the regular course. **s.2** (not s.1): rebuttable by "the method or circumstances of the preparation, transmission or storage". | Server-side event time at commit; documented storage method | **Verified fact** [S8] |
| REE Rule 9 s.1–2 | Admissibility and weight "may be established by an affidavit" of personal knowledge; the affiant may be cross-examined | Evidence pack with a draft custodian affidavit (D-031, next release) | **Verified fact** [S8] |
| REE generally | **The words "audit trail" and "log" don't appear in the Rules** | No prescribed log format: courts weigh reliability | **Verified fact** [S8] |
| RA 8792 s.13 | Electronic retention meets a keep-the-original rule if the record stays accessible, keeps its format (or one that "accurately represent[s]" it), and "Enables the identification of its originator and addressee, as well as the determination of the date and the time it was sent or received". s.13(b): a third party may do this. | Capture record plus activity log cover (iii). D-008 still applies: we never advise destroying paper. | **Verified fact** [S9] |
| RA 10535 (PST Act) | Binds government offices (display PST, sync with PAGASA at least monthly) and penalises only private TV and radio stations. **No duty on private businesses.** | Optional good practice (§7) | **Verified fact** [S10] |

---

## 5. Retention, visibility and privacy of the log

| Log | Proposed retention | Basis | Label |
|---|---|---|---|
| **Tenant activity log** | For the life of the tenant's account, with **no automatic purge** (as D-041). Entries about a permanently deleted document stay as its disposal record. At exit: included in the export, then deleted with the tenant's data. | s.29 leaves the choice to us. It must outlive the 5-year tax and 10-year contract periods the documents themselves serve (Section 11 §5.3). | **Recommendation** |
| **Payload values** (old and new) | Same as the activity log, but erasable under a client instruction (erasure, blocking, exit), leaving a redaction marker | RA s.16(e); keeps the chain verifiable | **Recommendation** |
| **Platform security log** (vendor) | **2 years**, then delete. Incident-related entries kept until the incident and any NPC matter are closed. | s.29: longer than general logs; NPC may require more. **No Philippine number exists; 2 years is our proposal.** | **Recommendation**; lawyer to confirm |
| **App and debug logs** | 30 days; IDs only, no content | s.29 "general system logs" | **Recommendation** |
| **Daily digests and TSA tokens** (hashes only) | 10 years, compliance mode | Matches the 10-year contract claim period (Civil Code art. 1144, Section 11) | **Recommendation**; whether hashes count as personal data is an **Open question** |
| **Exit record** (vendor keeps) | Tenant name, dates, export file hash, deletion certificate. Same period as the security log. | Shows we deleted on instruction (IRR s.44) | **Recommendation** |

**Who sees what (Recommendation):**

| Viewer | Sees |
|---|---|
| Users with history rights (Section 05 §6.4) | Changes, status and version events on documents they can see |
| Folder **Manage** and company admins | All of the above, plus views and downloads, IP and device |
| Company admins | Tenant-wide activity view; security events for their own users; support-access sessions highlighted; log export and verification |
| Vendor staff | Only through logged, time-limited support access (D-033). The security log is restricted to named ops staff. |
| Data subjects | Through the client (PIC). Person-level access report next release (D-033). |

Nobody, including the vendor, can edit or delete an event through the product. Erasing a payload is a separate, logged procedure run under a separate database user.

---

## 6. Fixity checks (D-031)

### 6.1 Practice

| Source | Finding | Label |
|---|---|---|
| NDSA Levels v2.0, Integrity row | L1 generate or verify on receipt (and virus check); L2 verify "when moving or copying content", "Back up integrity information and store copy in a separate location"; L3 verify "at fixed intervals", "Document integrity information verification processes and outcomes"; L4 verify "in response to specific events", "Replace or repair corrupted content" | **Verified fact** [S70] |
| NDSA 2014 guidance | Intervals "could be monthly, quarterly, or yearly". More often finds errors sooner but uses throughput. Storage-layer checks "will not log or help to fix issues"; keep your own fixity information. **Sampling is not recommended.** | **Verified fact** [S71] |
| NDSA 2017 survey | 74.3% check all content; 65 of 74 do not sample; large holders (>50 TB) check on a rolling basis; SHA-256 used by 25%; relying on provider checksums was rated "not at all important" most often | **Verified fact** [S72] |
| Archivematica | Storage Service API checks an AIP (a BagIt bag) against its checksums and the Payload-Oxum, which detects modified, deleted or added files. Results appear with "a full history of fixity checks". No built-in scheduler was found (cron likely). | **Verified fact**; scheduler **Open question** [S73] |
| AWS S3 | Designed for 99.999999999% durability [S74]. "Compute checksum" Batch Operations job (18 Aug 2025) computes full-object or composite SHA-256 and other algorithms "without downloading or restoring" data. Output is a CSV report to compare with our own hashes. No SSE-C. New uploads already get a CRC64NVME checksum by default [S22]. | **Verified fact** [S37][S22]; durability **Vendor claim** |

### 6.2 Cost for 1 TB per tenant (S3 Singapore, **Estimate** from [S38][S39][S40])

Assumes about 500,000 objects (2 MB average).

| Method | Per full pass | Notes |
|---|---|---|
| **S3 Batch compute checksum (SHA-256)** | about **$4.86**: 1,024 GB × $0.004 = $4.10, plus $0.25 per job, plus $0.50 per 0.5 M objects, plus about $0.01 for the object list | AWS only. Quarterly passes about $19/year; monthly about $58/year. |
| **Own worker** (stream and SHA-256) | about **$0.20** in GET requests (500k × $0.0004 / 1,000) plus worker time | Same-Region S3-to-EC2 transfer is free [S40]. SHA-256 ran at about 2.3 GB/s on one M1 core; one EPYC benchmark showed about 1.6 GB/s with SHA-NI [S76]. Hashing 1 TB is therefore roughly 8–17 core-minutes, and reading the data is the bottleneck. Works on **any** S3-compatible store. |
| For scale | Storing that 1 TB costs about $25.60/month in S3 Standard Singapore | Fixity is a few percent of storage cost |

### 6.3 Proposed schedule (**Recommendation**)

1. **On ingest:** SHA-256 by our worker (D-046).
2. **On every move:** re-verify on promotion from quarantine to originals, on restore from Trash or backup, and on any storage migration. Store the S3 `VersionId` with the hash.
3. **Rolling full pass every 90 days:** each night check 1/90 of each tenant's objects, oldest-verified first. 90 days sits inside NDSA's "monthly, quarterly, or yearly" range.
4. **Plus a small random sample nightly** (for example 50 files per tenant), so a failure doesn't wait up to 90 days.
5. **Results go to an insert-only `fixity_checks` table,** one row per file per check, kept as long as the activity log.
   - The activity log gets **failures** plus **one summary per tenant per run**. This avoids hundreds of thousands of routine events.
   - The document page shows "Integrity last verified: [date]".
6. **Hashes live in two places** (NDSA L2): in MySQL and in the daily locked log export (E).
7. **On a mismatch:**
   - Mark the version "Integrity check failed".
   - Alert vendor ops and company admins.
   - Restore from a previous S3 version or backup, re-verify, and log every step.
   - **Never silently replace** the file.
   - Treat it as a security incident until explained (16-03 s.22).
8. **Backups** are checked with the same job on restore tests (NPC 2023-06 s.21).

---

## 7. Clock and time

| Point | Detail | Label |
|---|---|---|
| Storage | Laravel 13's default `'timezone' => 'UTC'` [S75]. MySQL `TIMESTAMP` converts to UTC using the session time zone and ends in 2038; `DATETIME` does neither [S45]. | **Verified fact** |
| Proposal | Store `DATETIME(6)` in **UTC** with the DB session `time_zone = '+00:00'`. Display in **Asia/Manila** (UTC+8). Exports carry ISO 8601 with an offset (ASVS 16.2.2 [S2]). | **Recommendation** |
| AWS servers | Amazon Time Sync at `169.254.169.123` (IPv6 `fd00:ec2::123` on Nitro) through chrony; the default on AL2023. It **smears leap seconds**, so don't mix it with non-smeared sources. | **Verified fact** [S69] |
| Local data centre | chrony against several sources. PAGASA's `ntp.pagasa.dost.gov.ph` answered as **stratum 1** with the correct UTC time on 2026-10-20, but PAGASA does not document it as a public service. | Test **Verified fact** [S52]; status **Open question** |
| PAGASA's role | Official keeper of PST under RA 10535, which binds government, not private firms | **Verified fact** [S10] |
| Monitoring | Alert if the chrony offset exceeds 1 s. Record the time source in the system description. The PNPKI TSA policy uses ±1 s accuracy, a useful benchmark [S50]. | **Recommendation** |
| Ordering | Event order is `tenant_seq` and the chain, not timestamps. A clock step can't reorder history. | **Recommendation** |

---

## 8. How clients see the log

### 8.1 Competitors

| Product | What it shows | Retention | Label |
|---|---|---|---|
| **Odoo Documents 18/19** | The chatter "tracks changes to folders and files"; version history. **No built-in who-viewed or who-downloaded log found**; third-party apps fill the gap. | Not stated | **Verified fact** / **Secondary** [S80] |
| **M-Files** | Vault event log: logins, changes, deletions, state changes, "File downloaded", public link events. Opens, previews, exports and searches only in the separately sold "User Action Log". Mobile and API clients are not recorded. | Pruning reported | **Verified fact** [S77]; pruning **Secondary** |
| **Microsoft 365 / SharePoint (Purview)** | FileAccessed (5-minute de-duplication), FileDownloaded, FilePreviewed, FileModified, FileDeleted | **180 days** (Standard); 1 year (Premium); up to 10 years with an add-on licence | **Verified fact** [S79] |
| **Google Workspace Drive** | View, edit, download, print, upload, share-visibility changes, mainly for supported editions | **6 months** | **Verified fact** [S78] |
| **Box** | Access Stats: previews, edits, comments, downloads | Ranges up to "All Time" | **Verified fact** [S81] |

**Estimate:** keeping the log for the life of the documents, including views and downloads, with a verifiable export, is a real difference from Odoo (no access log) and from Microsoft and Google (180 days to 6 months on standard plans).

### 8.2 Proposed UI and export (**Recommendation**)

| Phase | Feature |
|---|---|
| MVP | **History tab per document:** a plain-language timeline ("Maria marked v3 Official: 'signed copy received'"), with filters for changes, access and integrity. "Integrity last verified" badge. Support-access actions shown with a distinct badge. |
| MVP | **Company admin "Activity" page:** filter by user, folder, action and date. CSV export for people and JSONL with hashes for checking. Each export is itself logged. |
| MVP (internal) | **Verifier job:** re-walks every tenant's chain nightly and checks it against the last digest. A failure is a security incident. |
| Next release | Downloadable verification report and documented algorithm, so an auditor can re-check an export; per-document extract with C+ links inside the **Evidence pack** (D-031); weekly "log fingerprint" email to company admins, so the client holds an anchor too; RFC 3161 tokens; person-level access report (D-033). |
| Claims | Say **"tamper-evident for changes after upload"** only once A, C and E are live. Never "tamper-proof", "immutable" or "court-admissible" (D-029). |

**ISO mapping note:** this design supplies evidence for Annex A 8.15 Logging, as already mapped in Section 04. Clock synchronisation is a neighbouring Annex A control; its number was not re-checked here. ISO remains the senior partner's decision (D-028).

---

## 9. Cost and effort summary (**Estimate**)

| Item | Running cost | Build effort |
|---|---|---|
| Event tables, writer, history UI, admin page, export | MySQL storage only. At about 1 KB per event and 4,000 events a day per tenant: about 1.5 GB a year per tenant. | Medium (this is the activity log D-014 already requires) |
| A: grants and separate migration user | Free | Low; depends on Officetul's deploy (§10) |
| B: triggers | Free | Very low |
| C and C+: chain, sealer, verifier | Small CPU | Low to medium |
| E: daily digest and locked exports | Pennies a month (1.5 GB × $0.025 ≈ $0.04/month per tenant after a year) | Low |
| F: RFC 3161 | Free endpoints (terms to check) | Low |
| Fixity, rolling 90 days, 1 TB tenant | About $1–20 a year (own worker vs Batch) | Low to medium |
| J: DB audit plugin | RDS option; Enterprise Audit needs a commercial licence | Low on RDS |

---

## 10. What to check in the Officetul code (D-013 design review)

1. Does Officetul already use spatie/laravel-activitylog, owen-it/laravel-auditing or its own table? Which version, and for what?
2. **Tenancy:** a single database with `tenant_id`, or a database per tenant? (The chain works either way; grants and backups differ.)
3. **Database users:** does the app connect as an admin or root user? Do migrations run with the same credentials? Is it RDS, and who holds the master user?
4. Are login, MFA and password-reset events already logged? Where, and for how long? Is MFA available (Section 11)?
5. `config/app.php` timezone, the DB session time zone, and whether existing columns are `TIMESTAMP` or `DATETIME`.
6. Is there an existing "impersonate user" or support-login feature? It must be routed through D-033 grants and logged.
7. Queue and scheduler setup (for sealing, digests and fixity), and which S3-compatible store is used today.
8. Do existing ERP modules perform mass updates or deletes (which bypass model events, §3)?

---

## 11. Recommendations summary

| MVP | Later (next release or after) |
|---|---|
| Activity log and security log with the §1 events and fields; payloads in a separate, salted, redactable table | Person-level access report; share-link and legal-hold events |
| "No log, no access"; state changes logged in the same transaction | Weekly log-fingerprint email to company admins |
| A (insert-only grants plus a separate migration user) plus B (blocking triggers) | J (DB audit plugin for vendor access to production) |
| C (per-tenant chain, sealed asynchronously) plus C+ (per-document link) | D as a full Merkle tree with inclusion proofs |
| E: daily digest to compliance-mode Object Lock; daily encrypted per-tenant export to locked storage | F: RFC 3161 on the daily root (FreeTSA plus one more; PNPKI if live) |
| Nightly chain verification; failure = incident | Evidence pack with verification report (D-031) |
| Fixity: on ingest, on every move, rolling 90-day full pass plus a nightly sample; results table; mismatch procedure | S3 Batch compute checksum as a second, independent check (if on AWS) |
| UTC `DATETIME(6)`, Asia/Manila display, chrony with drift alert | — |
| Per-document History tab; admin Activity page; CSV and JSONL export | Downloadable verifier and documented algorithm |
| Retention: activity log for the account's life; security log 2 years; app logs 30 days; digests 10 years | Retention rules per document type (with the retention release) |

---

## 12. Open questions

1. **Officetul code** (§10, all eight items).
2. **Hosting:** AWS Singapore or a local data centre? If local, which store, and does it offer Object Lock? Object Lock in the Wasabi and Linode Singapore regions, Alibaba Manila WORM and Huawei Philippines WORM are all unconfirmed. No Philippine provider with Object Lock was found.
3. **PNPKI TSA:** is it live? The endpoint returned 404. Who may use it, at what cost? Was the 2015 circular formally adopted? Ask DICT.
4. **TSA terms:** what are DigiCert's and Sectigo's terms for free automated use?
5. **Lawyer (add to the D-034 list):**
   - (a) Is 2 years right for the security log, and is keeping an exit record after deletion acceptable?
   - (b) Are salted hashes in a 10-year compliance-mode store personal data?
   - (c) Does an external anchor (AWS lock, foreign TSA) carry weight under REE Rule 7 s.2?
   - (d) Is coalescing views into 10-minute windows acceptable for law-firm clients?
   - (e) What is the "specified period required by the NPC" for incident logs (2023-06 s.29)?
6. **S3 Batch compute checksum:** does its full-object SHA-256 on multipart objects match our streamed SHA-256? Test on one object.
7. **Real event volume per tenant:** measure in the own-company pilot (D-017).
8. **Not verified:**
   - AWS's own QLDB text;
   - the Percona and MariaDB audit plugin licences;
   - the Archivematica scheduler;
   - NDSA v2.1 wording;
   - SharePoint's per-file activity UI;
   - Google's retention when logs are exported to BigQuery;
   - the size of the AWS free tier for data transfer.

---

## 13. Decisions for the owner

Each has a recommended answer, worded so "yes" accepts it.

| # | Decision | Recommended answer |
|---|---|---|
| 1 | Keep two logs: a tenant-facing **activity log** and a vendor **security log**, and add "Activity log", "Security log", "Fixity check" and "Log digest" to GLOSSARY.md? | **Yes.** "Audit trail" is used only when quoting others. |
| 2 | For the MVP, protect the log with insert-only grants, blocking triggers, a per-tenant hash chain with per-document links, and a daily digest in compliance-mode Object Lock storage, leaving RFC 3161 timestamps and client-held fingerprints to the next release? | **Yes** |
| 3 | Store old/new values and other personal details in a separate, salted payload table that can be redacted on the client's instruction without breaking the chain? | **Yes** |
| 4 | Refuse a download or preview when its log entry can't be written ("no log, no access")? | **Yes** |
| 5 | Log every download and export individually, but coalesce views and previews into one entry per user per version per 10 minutes? | **Yes** (lawyer to confirm for law firms) |
| 6 | Show views, downloads, IP and device only to folder Manage users and company admins, while everyone with history rights sees changes? | **Yes** |
| 7 | Retention: keep the activity log for the life of the account and hand it over at exit; keep the security log 2 years; app logs 30 days; hash-only digests 10 years? | **Yes** (lawyer to confirm the security-log and exit-record periods) |
| 8 | Fixity: verify on ingest and every move, plus a rolling full pass every 90 days and a nightly random sample, using our own worker so it works on any storage? | **Yes** |
| 9 | Store all times in UTC (`DATETIME(6)`), display them in Asia/Manila, and alert on clock drift above 1 second? | **Yes** |
| 10 | Build a small dedicated event writer for Documents rather than relying on spatie or owen-it model events, while keeping whichever package Officetul already uses for its other modules? | **Yes** |
| 11 | Allow the phrase "tamper-evident for changes after upload" in the product only once the chain and external digest are live, and never "tamper-proof" or "immutable"? | **Yes** (extends D-029) |

---

## 14. Sources (all accessed 2026-10-20)

| ID | Source | URL | Type |
|---|---|---|---|
| S1 | OWASP Logging Cheat Sheet | https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html | Primary |
| S2 | OWASP ASVS 5.0, V16 Security Logging and Error Handling | https://github.com/OWASP/ASVS/blob/master/5.0/en/0x25-V16-Security-Logging-and-Error-Handling.md | Primary |
| S3 | NPC Circular 2023-06, Security of Personal Data (Internet Archive) | https://web.archive.org/web/20261012063023/https://privacy.gov.ph/wp-content/uploads/2024/03/NPC-Circular-Repeal-16-01-Signed.pdf | Primary (archive) |
| S4 | NPC Circular 16-03, Personal Data Breach Management (Internet Archive) | https://web.archive.org/web/20261012063023/https://privacy.gov.ph/wp-content/uploads/2022/01/sgd-npc-circular-16-03-personal-data-breach-management.pdf | Primary (archive) |
| S5 | IRR of RA 10173 (Internet Archive) | https://web.archive.org/web/20241112161706/https://privacy.gov.ph/implementing-rules-regulations-data-privacy-act-2012/ | Primary (archive) |
| S6 | NPC Circular 2022-04 (Internet Archive) | https://web.archive.org/web/20261012063023/https://privacy.gov.ph/wp-content/uploads/2023/05/Circular-2022-04-2.pdf | Primary (archive) |
| S7 | RA 10173, Data Privacy Act (lawphil) | https://lawphil.net/statutes/repacts/ra2012/ra_10173_2012.html | Primary |
| S8 | A.M. No. 01-7-01-SC, Rules on Electronic Evidence (lawphil) | https://lawphil.net/courts/supreme/am/am_01-7-01_2001.html | Primary |
| S9 | RA 8792, Electronic Commerce Act (lawphil) | https://lawphil.net/statutes/repacts/ra2000/ra_8792_2000.html | Primary |
| S10 | RA 10535, Philippine Standard Time Act (lawphil) | https://lawphil.net/statutes/repacts/ra2013/ra_10535_2013.html | Primary |
| S11 | RA 11976, Ease of Paying Taxes Act (lawphil) | https://lawphil.net/statutes/repacts/ra2024/ra_11976_2024.html | Primary |
| S12 | BIR RR 9-2009 (SC E-Library) | https://elibrary.judiciary.gov.ph/thebookshelf/showdocs/10/48854 | Primary |
| S13 | BIR RMC 98-2026 | https://bir-cdn.bir.gov.ph/BIR/pdf/RMC%20No.%2098-2026%20redacted.pdf | Primary |
| S14 | PwC Tax Alert on RMC 10-2020 | https://www.pwc.com/ph/en/tax/tax-publications/tax-alerts/2020/tax-alert-8.html | Secondary |
| S15 | Forvis Mazars on RMO 9-2021 | https://forvismazars.com/ph/en/insights/tax-alerts/bir-rm-09-2021 | Secondary |
| S16 | PwC Tax Alert on RR 8-2022 | https://pwc.com/ph/en/tax/tax-publications/tax-alerts/2022/tax-alert-40.html | Secondary |
| S17 | Grant Thornton on RR 11-2025 | https://www.grantthornton.com.ph/technical-alerts/tax-alert/2025/implementation-of-the-electronic-invoicing-and-electronic-sales-reporting-system-under-the-create-more-act/ | Secondary |
| S18 | Conventus Law on NPC Circular 2024-02 (CCTV) | https://conventuslaw.com/report/philippines-updated-npc-guidelines-on-the-use-of-cctv-systems/ | Secondary |
| S19 | Baker McKenzie on NPC Advisory 2026-02 | https://www.bakermckenzie.com/en/insight/publications/2026/06/philippines-data-breach-notification-procedure-further-clarified | Secondary |
| S20 | Amazon S3 Object Lock | https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lock.html | Primary |
| S21 | Configuring S3 Object Lock | https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lock-configure.html | Primary |
| S22 | S3 Checking object integrity | https://docs.aws.amazon.com/AmazonS3/latest/userguide/checking-object-integrity.html | Primary |
| S23 | InfoQ, "AWS kills QLDB" (quotes AWS end-of-support date) | https://www.infoq.com/news/2024/07/aws-kill-qldb | Secondary |
| S24 | MySQL 8.4 GRANT | https://dev.mysql.com/doc/refman/8.4/en/grant.html | Primary |
| S25 | MySQL 8.4 Privileges Provided | https://dev.mysql.com/doc/refman/8.4/en/privileges-provided.html | Primary |
| S26 | MySQL 8.4 Trigger syntax | https://dev.mysql.com/doc/refman/8.4/en/trigger-syntax.html | Primary |
| S27 | MySQL 8.4 SIGNAL | https://dev.mysql.com/doc/refman/8.4/en/signal.html | Primary |
| S28 | MySQL 8.4 TRUNCATE TABLE | https://dev.mysql.com/doc/refman/8.4/en/truncate-table.html | Primary |
| S29 | MySQL 8.4 Binary log | https://dev.mysql.com/doc/refman/8.4/en/binary-log.html | Primary |
| S30 | MySQL 8.4 Binary log options | https://dev.mysql.com/doc/refman/8.4/en/replication-options-binary-log.html | Primary |
| S31 | MySQL Enterprise Audit | https://dev.mysql.com/doc/refman/8.4/en/audit-log.html | Primary |
| S32 | Percona Server 8.4 Audit Log Filter | https://docs.percona.com/percona-server/8.4/audit-log-filter-overview.html | Primary (vendor) |
| S33 | MariaDB Audit Plugin | https://mariadb.com/kb/en/mariadb-audit-plugin/ | Primary (vendor) |
| S34 | RDS MySQL MariaDB Audit Plugin option | https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Appendix.MySQL.Options.AuditPlugin.html | Primary |
| S35 | RDS MySQL binary log format | https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_LogAccess.MySQL.BinaryFormat.html | Primary |
| S36 | RDS master user privileges | https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/UsingWithRDS.MasterAccounts.html | Primary |
| S37 | S3 Batch Operations compute checksums; integrity at rest; What's New 18 Aug 2025; What's New Dec 2024 | https://docs.aws.amazon.com/AmazonS3/latest/userguide/batch-ops-compute-checksums.html ; https://docs.aws.amazon.com/AmazonS3/latest/userguide/checking-object-integrity-at-rest.html ; https://aws.amazon.com/about-aws/whats-new/2025/08/amazon-s3-verify-content-stored-datasets/ ; https://aws.amazon.com/about-aws/whats-new/2024/12/amazon-s3-default-data-integrity-protections | Primary |
| S38 | AWS Price List API, S3 ap-southeast-1 (published 2026-10-15) | https://pricing.us-east-1.amazonaws.com/offers/v1.0/aws/AmazonS3/current/ap-southeast-1/index.json | Primary |
| S39 | AWS Price List API, Data Transfer ap-southeast-1 (2026-09-16) | https://pricing.us-east-1.amazonaws.com/offers/v1.0/aws/AWSDataTransfer/current/ap-southeast-1/index.json | Primary |
| S40 | S3 pricing page and FAQ (same-Region transfer free) | https://aws.amazon.com/s3/pricing/ ; https://aws.amazon.com/s3/faqs/ | Primary |
| S41 | RFC 9162, Certificate Transparency v2.0 | https://www.rfc-editor.org/rfc/rfc9162 | Primary |
| S42 | Crosby & Wallach, "Efficient Data Structures for Tamper-Evident Logging", USENIX Security 2009 (PDF link not confirmed) | https://www.usenix.org/conference/usenixsecurity09/technical-sessions/presentation/efficient-data-structures-tamper-evident | Primary (abstract page) |
| S43 | Schneier & Kelsey, "Secure Audit Logs to Support Computer Forensics", ACM TISSEC vol. 1 no. 3, 1999 | https://www.schneier.com/academic/archives/1999/05/secure_audit_logs_to.html | Primary |
| S44 | RFC 3161, Time-Stamp Protocol; RFC 5816 update | https://www.rfc-editor.org/rfc/rfc3161 ; https://www.rfc-editor.org/rfc/rfc5816 | Primary |
| S45 | MySQL 8.4 DATE, DATETIME and TIMESTAMP types | https://dev.mysql.com/doc/refman/8.4/en/datetime.html | Primary |
| S46 | FreeTSA.org | https://www.freetsa.org/index_en.php ; https://www.freetsa.org/freetsa.html | Primary (operator) |
| S47 | DigiCert RFC 3161 TSA server | https://knowledge.digicert.com/general-information/rfc3161-compliant-time-stamp-authority-server | Vendor |
| S48 | Sectigo time-stamping server | https://sectigo.com/resource-library/time-stamping-server | Vendor |
| S49 | OpenSSL `openssl-ts` manual | https://docs.openssl.org/3.0/man1/openssl-ts/ | Primary |
| S50 | PNPKI Time-Stamping Authority TSP/PS v1.0 (2015) | https://cms-cdn.e.gov.ph/DICT/pdf/Policy_PNPKI-TSP-PS-_20150904_n.pdf | Primary |
| S51 | ICTO Memorandum Circular adopting the TSA-TSP/PS (unnumbered); DICT draft MC page (403) | https://cms-cdn.e.gov.ph/DICT/pdf/ICTO_MC-for-the-PNPKI-TSA-TSP-PS_20150904_j.pdf ; https://dict.gov.ph/draft-memorandum-circular-time-stamping-authority/ | Primary |
| S52 | DICT Foxit timestamp guide (endpoint), plus direct tests on 2026-10-20: PNPKI TSA POST (HTTP 404); SNTP query to `ntp.pagasa.dost.gov.ph` (stratum 1); FreeTSA, DigiCert and `time.aws.com` responses | https://cms-cdn.e.gov.ph/DICT/pdf/How-to-Add-a-Timestamp-with-Foxit.pdf | Primary plus direct test |
| S53 | A.M. No. 24-10-14-SC, Rules on Electronic Notarization (SC E-Library) | https://elibrary.judiciary.gov.ph/thebookshelf/showdocs/11/98644 | Primary |
| S54 | OpenTimestamps site and client README | https://opentimestamps.org/ ; https://github.com/opentimestamps/opentimestamps-client | Vendor / Primary (project) |
| S55 | spatie/laravel-activitylog: Packagist, composer.json, UPGRADING.md, v5 docs | https://packagist.org/packages/spatie/laravel-activitylog ; https://github.com/spatie/laravel-activitylog/blob/main/composer.json ; https://raw.githubusercontent.com/spatie/laravel-activitylog/main/UPGRADING.md ; https://spatie.be/docs/laravel-activitylog/v5/introduction | Primary (project) |
| S56 | owen-it/laravel-auditing: Packagist, audits.stub, docs | https://packagist.org/packages/owen-it/laravel-auditing ; https://github.com/owen-it/laravel-auditing/blob/master/database/migrations/audits.stub ; https://laravel-auditing.com/guide/introduction.html | Primary (project) |
| S57 | laravel-chronicle/core on Packagist; other chain packages found via Packagist search | https://packagist.org/packages/laravel-chronicle/core | Primary (registry) |
| S58 | Laravel 13.x Eloquent (mass updates and deletes) | https://laravel.com/docs/13.x/eloquent | Primary |
| S59 | Laravel 13.x release notes | https://laravel.com/docs/13.x/releases | Primary |
| S60 | Wasabi locations, pricing, immutability docs | https://www.wasabi.com/locations ; https://wasabi.com/pricing ; https://docs.wasabi.com/docs/immutability-compliance-and-object-locking | Vendor |
| S61 | Backblaze B2 Object Lock, regions, pricing | https://www.backblaze.com/docs/cloud-storage-object-lock ; https://www.backblaze.com/docs/cloud-storage-data-regions ; https://www.backblaze.com/cloud-storage/pricing | Vendor |
| S62 | Akamai Object Lock | https://techdocs.akamai.com/cloud-computing/docs/protect-data-with-object-lock | Vendor |
| S63 | Alibaba Cloud OSS regions; retention policies | https://www.alibabacloud.com/help/en/oss/user-guide/regions-and-endpoints ; https://help.aliyun.com/en/oss/user-guide/oss-retention-policies | Vendor |
| S64 | Google Cloud Storage Bucket Lock; locations | https://docs.cloud.google.com/storage/docs/bucket-lock ; https://docs.cloud.google.com/storage/docs/locations | Vendor |
| S65 | Huawei Cloud Philippines Region press release (3 Dec 2024) | https://huaweicloud.com/intl/en-us/news/20241203162826823.html | Vendor |
| S66 | Cloudflare R2 bucket locks; data location | https://developers.cloudflare.com/r2/buckets/bucket-locks/ ; https://developers.cloudflare.com/r2/reference/data-location/ | Vendor |
| S67 | DigitalOcean Spaces S3 compatibility; Spaces API | https://docs.digitalocean.com/products/spaces/reference/s3-compatibility/ ; https://docs.digitalocean.com/reference/api/spaces/ | Vendor |
| S68 | Philstar, Converge sovereign cloud (June 2026) | https://www.philstar.com/business/motoring/2026/06/18/2535897/beyond-connectivity-converge-unveils-country-sovereign-cloud-stack-built-whats-next | Secondary |
| S69 | Amazon EC2 time: set-time; configure NTP | https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/set-time.html ; https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/configure-ec2-ntp.html | Primary |
| S70 | NDSA Levels of Digital Preservation v2.0 (matrix on OSF) | https://ndsa.org/publications/levels-of-digital-preservation/ ; https://osf.io/download/qd54c/ | Primary |
| S71 | NDSA, "Checking Your Digital Content: What is Fixity, and When Should I Be Checking It?" (2014) | https://ndsa.org/documents/NDSA-Fixity-Guidance-Report-final100214.pdf | Primary |
| S72 | NDSA 2017 Fixity Survey Report | https://ndsa.org/documents/Report_2017NDSAFixitySurvery.pdf | Primary |
| S73 | Archivematica Storage Service fixity; Artefactual Fixity tool | https://www.archivematica.org/en/docs/storage-service-0.17/fixity/ ; https://github.com/artefactual/fixity | Primary (project) |
| S74 | S3 Data protection (durability) | https://docs.aws.amazon.com/AmazonS3/latest/userguide/DataDurability.html | Primary (vendor) |
| S75 | laravel/laravel 13.x `config/app.php` | https://raw.githubusercontent.com/laravel/laravel/13.x/config/app.php | Primary (source) |
| S76 | LKML SHA-NI interleaving patch (Feb 2025), plus a local `openssl speed` run on Apple M1 | https://lkml.iu.edu/hypermail/linux/kernel/2502.1/06361.html | Secondary; local test |
| S77 | M-Files user guide: event log, event types, User Action Log, 21 CFR Part 11 | https://userguide.m-files.com/user-guide/latest/eng/Event_log.html ; https://userguide.m-files.com/user-guide/latest/eng/event_types.html ; https://userguide.m-files.com/user-guide/latest/eng/user_action_log.html ; https://userguide.m-files.com/user-guide/latest/eng/FDA_21_CFR_Part_11.html | Vendor |
| S78 | Google Workspace data retention and lag times; Drive log events | https://knowledge.workspace.google.com/admin/reports/data-retention-and-lag-times ; https://knowledge.workspace.google.com/admin/reports/drive-log-events | Vendor |
| S79 | Microsoft Purview audit overview, retention policies, activities | https://learn.microsoft.com/en-us/purview/audit-solutions-overview ; https://learn.microsoft.com/en-us/purview/audit-log-retention-policies ; https://learn.microsoft.com/en-us/purview/audit-log-activities | Vendor |
| S80 | Odoo Documents 18.0 and 19.0 docs | https://www.odoo.com/documentation/18.0/applications/productivity/documents.html ; https://www.odoo.com/documentation/19.0/applications/productivity/documents.html | Vendor |
| S81 | Box Access Stats Report | https://support.box.com/hc/en-us/articles/47760082930707-Access-Stats-Report | Vendor |
| S82 | Internal: R09, R10, R13; Sections 05, 10, 11 | research/ ; sections/ | Internal |
