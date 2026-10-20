# R19 — Backup, retention and disaster recovery

- **Status:** First pass, design-level. Research only, no code. Feeds Section 15 (backups and exit deletion), Section 18 (cost) and the data processing addendum.
- **Researched:** 2026-10-20. **All sources accessed 2026-10-20.**
- **Question:** How should Officetul Documents back up, restore and recover a company's documents and data, including deletion at contract exit, in a way a 1–2 person team can actually run and test?
- **Labels:** **Verified fact** (read in the primary source), **Vendor claim**, **Secondary** (news, blog or search excerpt only), **Estimate**, **Open question**, **Recommendation** (proposed, not approved). Source IDs `[S#]` point to §14.

**Access caveats.**
- Most web pages were read through a summarising fetch tool. Quotes are close to the page but may differ by a word. Re-check the live page before copying any quote into a contract.
- AWS prices come from the AWS Price List API files for Asia Pacific (Singapore) and nearby Regions, published between 2026-09-11 and 2026-10-20 [S19]. The RDS free backup allowance comes from a 2022 AWS blog, because the pricing page returned no backup section [S7].
- NPC texts were read from Internet Archive copies, as in R10. The extracted text was searched for every use of "backup".
- Hosting is not chosen (R18 covers hosting and cost). This note gives an **AWS path** in full and a **non-AWS path** for a Philippine data centre. Officetul's current backups are unknown: ERP questionnaire Q13 is unanswered.
- Terms follow [GLOSSARY.md](../GLOSSARY.md). "Company" means a client tenant.

---

## 1. Recovery targets

### 1.1 The two numbers, in plain language

- **RPO (recovery point objective):** how much of the most recent work could be lost. "RPO 5 minutes" means that after a failure, at most the last 5 minutes of uploads and edits might have to be redone.
- **RTO (recovery time objective):** how long until people can work again. "RTO 4 hours" means the app is usable within 4 hours of us starting the recovery.

Both are **targets** we design and test for, not guarantees. Odoo publishes its targets the same way.

### 1.2 What vendors publish

| Vendor | Backups | RPO / RTO | Label |
|---|---|---|---|
| **Odoo Online** | "14 full backups for at least 3 months: 1/day for 7 days, 1/week for 4 weeks, 1/month for 3+ months", on at least 3 machines in different data centres plus "a fourth frozen copy in an immutable cold storage". Customers can download their own backup at any time. | Server failure: RPO 24 h, RTO 6 h. Data-centre failure: RPO 24 h, RTO 24 h. Called "not legally binding guarantees". | **Vendor claim** [S37] |
| **Odoo.sh** | Production backed up daily: "Seven daily, four weekly, and three monthly backups". Each backup holds the database dump and the filestore. Staging and development builds are not backed up. | None published | **Vendor claim** [S39] |
| **Zoho** | Daily incremental and weekly full backups; "All backed up data are retained for a period of three months." The primary data centre is replicated to a secondary one "in near real time". DR drills are "performed periodically". | No numbers | **Vendor claim** [S40] |
| **M-Files Cloud** | Backups included; "six copies of your data are always maintained", three in a secondary region. A "Backup Delivery" add-on lets customers download offsite copies. | No numbers found | **Vendor claim** [S41] |
| **Atlassian Cloud** | Daily RDS snapshots "retained for 30 days with support for point-in time recovery"; "quarterly testing of our backups". Its backups "are not used to revert customer-initiated destructive changes". | RPO 1 h, RTO 6 h. The paid Backup & Restore add-on separately states RPO 24 h, RTO 12 h, within size limits. | **Vendor claim** [S47][S48][S49] |
| **Microsoft 365 / Google Workspace** | No RPO/RTO found on the pages read. Microsoft's consumer services agreement: "We recommend that you regularly backup Your Content", and Microsoft is "not liable" for loss. Google lets admins recover Drive items for 25 days after a user empties the trash. | — | **Vendor claim** [S43][S46] |

**Reading:** SME-oriented vendors publish daily backups and a 24-hour RPO; Atlassian publishes 1 hour. Nobody we checked promises restore of data a customer deleted themselves.

### 1.3 Proposed targets for Officetul Documents (**Recommendation**)

| Situation | RPO (data at risk) | RTO (back to work) | Why it is achievable |
|---|---|---|---|
| **Everyday incident** in the main Region: a bad release, a corrupted table, files deleted by our bug | Database: **≤ 5 minutes**. Files: **minutes**. | **4 business hours** to a verified state | RDS uploads transaction logs to S3 "every five minutes" [S3]. S3 versioning plus AWS Backup continuous backup cover files [S8]. |
| **Disaster**: the main Region is down for a long time, or our main AWS account is taken over | **≤ 24 hours** | **2 business days** | Daily locked copies in a separate account and a second Region (§2, §3). Rebuilding the app stack elsewhere is the slow part. |
| **One company's deleted Documents** restored on request | To the chosen point in time inside the backup window | **2 business days** from the request | Side restore, then copy back (§5) |

- These sit between Odoo (24 h / 6–24 h) and Atlassian (1 h / 6 h). They need only a 1–2 person team working business hours.
- Put them in the data processing addendum as **targets, not guarantees**. NPC Circular 2023-06 s.21 asks for "Personal data backup, restoration, and remedial time" in the business continuity plan [S35].
- **The first drill (§7) confirms or corrects the RTOs.** Until then they are **Estimates**.

---

## 2. Database backups (MySQL 8.4)

The Documents tables live in Officetul's **shared** MySQL database (D-074), so any database backup is an **Officetul-wide** backup. Its design has to be agreed with the ERP maintainer (Q13).

### 2.1 Managed: Amazon RDS for MySQL

| Point | Detail | Label |
|---|---|---|
| Version | RDS supports MySQL **8.4** (8.4.5 to 8.4.11 listed; 8.4 standard support to 31 July 2029). MySQL 8.0 community end of life was **30 April 2026**; RDS 8.0 is now Extended Support only. This confirms the secondary source behind D-074. | **Verified fact** [S6] |
| Automated backups | A storage-volume snapshot of the **whole instance**, not of single databases. First snapshot full, later ones incremental. | **Verified fact** [S1] |
| Retention | **0–35 days** (0 turns backups off). The default is 7 days in the console and 1 day via API/CLI. | **Verified fact** [S2] |
| Point-in-time restore | Any time within the retention period. Transaction logs uploaded "every five minutes". A restore **creates a new DB instance** "without modifying the source". It is usable before all blocks have loaded, but slower until then. | **Verified fact** [S3] |
| Deleting the instance | Without "Retain automated backups", **all automated backups are deleted with the DB instance**. Manual and final snapshots survive. Up to 100 manual snapshots per Region. | **Verified fact** [S1] |
| Cross-Region | "Replicate snapshots and transaction logs" to another Region. Singapore can send to Hong Kong, Taipei, Malaysia, New Zealand, Thailand, Mumbai, Seoul, Sydney, Tokyo and three US Regions. **Jakarta is not on Singapore's list.** Up to 20 cross-Region automated backups per account. | **Verified fact** [S4] |
| AWS Backup | RDS DB instances support cross-Region and cross-account copy, continuous backup and point-in-time restore, and restore testing. They are **not** supported in a logically air-gapped vault (Aurora is). | **Verified fact** [S9] |
| Encryption (D-059) | An encrypted instance encrypts "its logs, automated backups, read replicas, and snapshots" with AES-256. Encryption can only be chosen at creation. **"You can't share a snapshot that has been encrypted using the AWS managed key"**, so a cross-account copy needs a **customer-managed KMS key**. A cross-Region copy needs a key in the destination Region. | **Verified fact** [S5] |
| Cost | Backup storage is free up to 100% of database storage in the Region (2022 blog). Above that, Singapore charges **$0.095 per GB-month**. | Allowance **Secondary** [S7]; price **Verified fact** [S19] |

**Note on D-059:** a customer-managed KMS key is still the provider's key service, not a per-company key, so it fits D-059. It does add a small monthly key fee (see R18).

### 2.2 Self-managed (for a Philippine data centre or plain VMs)

| Tool | Facts | Fit | Label |
|---|---|---|---|
| **Percona XtraBackup 8.4** (GPLv2) | Physical hot backup. Backs up "only on MySQL 8.4 and Percona Server for MySQL 8.4"; "does not support backups on MySQL 8.0 or 9.x". Latest release **8.4.0-7 (2026-09-17)**. Use "a Percona XtraBackup version that is equal to your source server major version". Incremental by page LSN. ZSTD compression by default. Encryption `--encrypt=AES256` with a key file. **The keyring file is not copied into the backup**, so it must be saved separately if InnoDB encryption is used. Percona releases nothing for MySQL Innovation releases; 9.7 is at RC only. | Best self-managed choice. Matches D-074 (8.4 LTS, no Innovation releases). | **Verified fact** [S20][S21][S22][S23][S24] |
| **Binary-log point-in-time recovery** | Restore the full backup, then replay binlogs with `mysqlbinlog … \| mysql`. MySQL advises `--start-position`/`--stop-position` over datetimes ("higher risk of missing binary log events"). `binlog_expire_logs_seconds` defaults to **30 days**. `binlog_encryption` defaults to OFF and needs a keyring. | Gives an RPO of minutes if binlogs are copied off the server often | **Verified fact** [S25][S26] |
| **MySQL Shell dump utilities** | `util.dumpInstance/dumpSchemas/dumpTables` with a **per-table `where` filter**. Consistent by default (`FLUSH TABLES WITH READ LOCK`, then `LOCK INSTANCE FOR BACKUP`). 4 threads by default, zstd compression. **No native dump encryption** found on the page. `util.loadDump` reads from S3-compatible storage. | Not a full backup method at our size. **Useful for extracting one company's rows** (§5). | **Verified fact** [S27]; no encryption by omission only |
| **mysqldump** | `--where`; `--single-transaction` gives a consistent InnoDB dump "without blocking any applications". | Small pilot or one-off exports only | **Verified fact** [S28] |
| MySQL Enterprise Backup | "Only available in select Commercial Editions" | Licence cost; not needed | **Verified fact** [S29] |

**Recommendation (non-AWS):**
- Nightly XtraBackup full backup (weekly full plus daily incrementals once the database passes about 100 GB), encrypted with `--encrypt=AES256`.
- Binlogs copied off the server **every 5 minutes**.
- Both shipped to a second provider with Object Lock (§3.4).
- Keep the encryption key file and any keyring in a password manager held by two people, **never only on the backup host**.

### 2.3 Recommended database setup on AWS (**Recommendation**)

1. RDS for MySQL 8.4, encrypted with a customer-managed KMS key. Automated backups at **35 days**. "Retain automated backups" on. Deletion protection on.
2. An **AWS Backup plan**: one **daily** snapshot copied in a single action to a **separate backup AWS account** in a **second Region** (Tokyo proposed: on the RDS list from Singapore and fully featured in AWS Backup [S4][S9]). Keep the daily copies **35 days**. Keep the copy taken on the 1st of each month **120 days** (§6.3 explains why).
3. The backup account's vault uses **Vault Lock in compliance mode** (§3.2) and its **own KMS key**, owned by the backup account.
4. Later, if a client needs it: RDS cross-Region automated backup replication, which brings the disaster RPO down from 24 hours to minutes [S4].

---

## 3. File backups

### 3.1 Why versioning (D-073) is not a backup

S3 versioning keeps old and deleted versions **in the same bucket, account and Region**. It does not protect against:
- anyone holding the account's admin keys: a delete **with a version ID** removes that version permanently [R17 §5.1];
- our own permanent-deletion job, which deliberately deletes every version (R17 §5.3);
- a Region-wide or account-wide event.

Versioning stays as the fast "undo" layer. **Recommendation:** set the noncurrent-version window to **35 days**, inside D-073's 30–90 range and matching the database window.

### 3.2 AWS options

| Option | Facts | Label |
|---|---|---|
| **S3 replication (CRR)** | Copies new objects asynchronously, to another Region and/or **another account**, keeping version IDs and metadata. Existing objects need Batch Replication. A **delete with a version ID is not replicated** ("This protects data from malicious deletions"). Delete markers replicate only if delete-marker replication is switched on. Lifecycle actions are not replicated. **Object Lock retention is replicated**. The "owner override" option makes the destination account own the replicas. AWS's comparison table puts plain cross-Region replication under "within 24-48 hours (not SLA backed)"; Replication Time Control copies 99.99% of new objects within 15 minutes under an SLA. | **Verified fact** [S14][S15] |
| **AWS Backup for S3** | Needs versioning. **Continuous** backup gives point-in-time restore for **35 days**. **Periodic** backups can be kept up to 99 years. Incremental at object level. Restores whole buckets, prefixes or single objects. Restores "all backed-up data and metadata **except original creation date, version ID**, storage class, and e-tags". No SSE-C, Glacier Flexible or Deep Archive. Cross-Region and cross-account copies are possible, but **copies lose point-in-time restore**. **If EventBridge notifications are turned off on the bucket, continuous backup stops silently**: "no notification, alert, or backup job failure". | **Verified fact** [S8] |
| **Vault Lock** | Compliance mode: after a grace time of **at least 3 days**, the vault and lock "cannot be changed or deleted by any user or by AWS". "If any user (including the root user) attempts to delete a backup … AWS Backup will deny." Assessed by Cohasset for SEC 17a-4. No extra charge. **But:** if the AWS account is closed, AWS keeps it suspended for 90 days, then "deletes the contents of your backup vault, even if AWS Backup Vault Lock was in place." | **Verified fact** [S11] |
| **Logically air-gapped vault** | A vault that is **always** locked in compliance mode. Its backups live in an AWS-owned service account. It can be shared to another account through AWS RAM, or recovered through multi-party approval even if our account is closed (until the post-closure period ends). Minimum retention 7 days. **Supports S3 but not RDS DB instances.** The vault page says it is not available in Malaysia, Thailand, Taipei or New Zealand, but the Region table on another AWS page ticks Malaysia: **the two pages disagree**. Singapore, Tokyo and Sydney support it on both. | **Verified fact** [S9][S10]; Malaysia **Open question** |
| **In-Philippines AWS** | There is **no AWS Region in the Philippines**. The Manila Local Zone (`ap-southeast-1-mnl-1a`) lists EC2 (T3, C5, R5, M5), EBS gp2, ALB and a few networking services. **S3 and RDS are not listed.** | **Verified fact** [S18] (table read in flattened form) |

### 3.3 Recommended file setup on AWS (**Recommendation**)

**What is backed up:**

| Storage area (D-075) | Backed up? | Why |
|---|---|---|
| **Originals** | **Yes** | The record itself |
| **Log digests** | **Yes, replicated** to a compliance-locked bucket in the backup account with the same 10-year retention (D-062). This is cheap: hashes only. | The tamper-evidence anchor must survive losing the main account |
| **Derived** | **No** | Regenerable from Originals. OCR text for search lives in MySQL (R16), so search survives a database restore. Previews and OCR PDFs regenerate on demand, or by a background queue. This saves about $400–480 a month at Growth scale (§9). |
| **Quarantine** (incl. infected files) | **No** | Infected files must be gone after 30 days (D-047). A backup would keep malware for months. |
| **Exports** | **No** | They expire after 7 days by design |

**How (Plan A, MVP to about 1 TB of Originals):**
- AWS Backup **continuous** backup of the Originals bucket in Singapore (35-day point-in-time restore).
- A **daily copy** to the locked vault in the backup account in the second Region: dailies kept 35 days, monthlies 120 days. This is the same vault and rules as the database, so there is one console, one lock and one restore-test tool for everything.
- **An alarm on the bucket's EventBridge setting**, because of the silent-failure trap above [S8].

**How (Plan B, when Originals pass about 1 TB):** switch files to cheaper S3 replication.
- Cross-account replication to a backup-account bucket in the second Region, in S3 Standard-IA, with **Object Lock compliance mode, 120-day default retention**.
- **Delete-marker replication on**, and the replica's lifecycle **expires noncurrent versions after 120 days**. A permanent deletion first places a delete marker, which replicates, and then removes versions by ID, which does not. The replica therefore keeps a deleted Original for exactly the retention window, then lets it go.
- **Weakness to handle:** an Object Lock retention period counts from when each object was written. A three-year-old Original in the replica is no longer locked. Two fixes: a monthly job that **extends retention** on all live replicas (S3 supports extending [S16]; Kopia does the same with "extend-object-locks" [S33]), or keep AWS Backup Vault Lock for a monthly copy.

### 3.4 Non-AWS path (Philippine data centre)

| Point | Detail | Label |
|---|---|---|
| Second site with Object Lock | No Philippine provider with published S3 Object Lock was found (R14 §2.3). The simplest second site for a Philippine-hosted primary is **an S3 bucket in AWS Singapore** with Object Lock, which also holds the Log digests (R14). | **Verified fact** (R14); choice **Recommendation** |
| rclone | Since v1.74.0 (2026-05-01) it can set Object Lock mode, retain-until date and legal hold on uploads. You "must set BOTH object_lock_mode AND object_lock_retain_until_date". Latest release v1.75.2. | **Verified fact** [S32] |
| restic | Encrypts everything with AES-256 (counter mode) and Poly1305-AES. **No native Object Lock**: a maintainer says setting it up manually "is extremely tricky to get right". `rest-server --append-only` is its alternative. | **Verified fact** [S30][S31] |
| Kopia | Compliance-mode Object Lock with `--retention-mode COMPLIANCE`, plus optional lock extension during maintenance. The docs contradict themselves on backends other than S3. | **Verified fact** [S33] |
| Backblaze B2 | Compliance and governance modes and legal hold; Object Lock can be turned on for an existing bucket. **No Asia region** (R14). | **Verified fact** [S34] |

**Recommendation (non-AWS):**
- Database: as in §2.2, uploaded with rclone and Object Lock retention, or with Kopia.
- Originals: **`rclone copy`** (adds files and never deletes; never `sync`) to a locked bucket, nightly, plus a weekly check that the counts match.
- Each upload gets a **120-day** retain-until date, and a monthly job re-uploads or extends the lock on live Originals.

### 3.5 Ransomware and stolen admin keys

Assume someone holds Officetul's **main AWS account admin keys**.

| They can | They cannot |
|---|---|
| Delete the RDS instance, and its automated backups too unless "retain" is set [S1] | Touch the backup account's compliance-locked vault, even as root [S11] |
| Delete every Original version in the main bucket | Delete Object-Locked Log digests before they expire [S16] |
| Schedule deletion of main-account KMS keys. AWS enforces a **7–30-day** waiting period, and a deleted key makes its data "unrecoverable" [S17]. | Undo a delete-by-version-ID replica in Plan B, because those deletes don't replicate [S15] |

**Controls (Recommendation):**
1. A separate backup account. Its own root user with a hardware MFA key held offline by two people. **No standing access** from the main account or from CI.
2. Backup copies encrypted with **backup-account** KMS keys. If copies used main-account keys, an attacker could schedule key deletion and make every backup unreadable within 7–30 days.
3. Alarms (CloudTrail to EventBridge) on `DeleteDBInstance`, `ScheduleKeyDeletion`, bucket lifecycle or notification changes, mass `DeleteObject`, and Backup job failures. These alarms go to two people's phones.
4. **Ship the Security log off the database in near real time** (for example to a locked log bucket). A point-in-time restore would otherwise erase the attacker's own trail.

---

## 4. Keeping database and files consistent on restore

### 4.1 What the existing rules guarantee

- **Object before row** (D-075): every Original file a row points to was stored before that row existed.
- **Row before delete:** bytes are deleted only after the commit that removed the last row (R17 §5.3).

So after restoring the **database to time T**:
- every row from T has an object, **unless** that object was purged after T (a permanent deletion or an exit deletion). A **purge ledger** (§6.2) and the backups cover that case;
- any object **newer than T** has no row. It is an "orphan": usually an upload whose row was lost in the rollback.

### 4.2 Restore runbook (**Recommendation**)

1. **Freeze.** Put Documents in maintenance mode and stop the queue workers and the scheduler. On the restored copy, **disable outgoing mail and inbound email capture**, so the copy never emails clients.
2. **Restore the database to T** as a new instance (RDS does this by default [S3]). Never overwrite the only copy.
3. **Files before users.** For every `original_files` row not marked purged, check the object exists. Restore missing ones from versioning, or else from AWS Backup (single objects or prefixes [S8]).
   - AWS Backup restores **new version IDs** [S8]. Our rows store the S3 `VersionId` (R13 §1.4), so update the stored version ID and log it as a "move".
4. **Fixity on every restored object** (D-061 "on every move"). Re-hash, compare with the stored SHA-256, and write a `fixity_checks` row. Any mismatch is an incident (Section 09 §6.3).
5. **Orphan sweep in report-only mode.** After a restore, orphans are probably lost uploads. They are **never auto-deleted**. Next release: offer them to the company admin as Inbox items marked "recovered after restore". Their Capture record is gone, so the label must say so.
6. **Re-apply deletions** that happened after T, from the purge ledger (§6.2).
7. **Verify the Activity log chain** against the Log digests (§4.3).
8. **Write a restore marker** in each affected company's Activity log, then reopen. Derived files regenerate in the background.
9. **Tell affected companies** the restore point and what to re-enter. A loss of data may also be a personal data breach under the 72-hour rules (R10). The incident procedure (D-032) decides that.

### 4.3 Activity log chain after a point-in-time restore

**What breaks:** the chain in the restored database ends at entry N, written just before T. The Log digests in locked storage include days **after** T. Those digests refer to entries that no longer exist, so plain verification reports a rollback.

**Proposed handling (Recommendation):**
- **Never rewrite or invent lost entries.** The daily digests for the lost days stay in locked storage as proof that entries existed.
- The first entry after the restore is a **restore marker**. It chains from entry N and records:
  - the restore point T;
  - the incident ticket;
  - the last digest that still verifies;
  - the digests that cover the lost range (count and final hash).
- **The verifier accepts a marker** and reports, for example: "externally verified up to the digest of 14 Oct; 14 Oct 00:00 to 15 Oct 09:42 internally consistent only; 15 Oct 09:42 to 16 Oct 11:05 lost in restore (incident #12, digest of 15 Oct proves 3,214 entries existed)."
- **Partial-day gap:** a daily digest covers the whole day. Entries from midnight to T can only be checked for internal consistency, not against an outside digest. An **hourly digest** would shrink this gap to under an hour, for a few hundred extra tiny objects per company per month. That is an option for later, not a change to D-061 now.
- **Lawyer question (L8):** does a documented gap weaken the log's weight under REE Rule 7?

---

## 5. Restoring one company; client deletions

### 5.1 Per-company restore from a shared database

- **A single company cannot be restored in place.** RDS restores a whole instance [S1].
- **Feasible pattern (Recommendation):** restore the whole database to a **side instance** at T, then copy back **only that company's missing Documents**.
  - Copy back with an Artisan command that has a second database connection. It runs through the normal company guard (D-052) and writes **insert-if-absent by ULID** (D-075).
  - Files: the missing Originals are restored and hash-checked **before** their rows are inserted (object before row).
  - Each restored Document gets an Activity log entry: "Restored from backup at the request of [admin], ticket #…".
  - The Activity log itself is **never** rolled back for one company. It is insert-only (D-061) and keeps running.
  - MySQL Shell `dumpTables` with a `where` filter on `tenant_id` is the fallback extraction tool [S27].
- **Full company rollback** (making the company look exactly as it did at T) is **not recommended**. It would destroy work done after T and fight the insert-only log.

| Step | Year 1 (company ≈ 1 GB of rows, 9,500 Originals) | Growth (company ≈ 3 GB, 63,000 Originals) | Label |
|---|---|---|---|
| Side instance at T | under 1 h | 1–3 h | **Estimate** (AWS publishes no restore duration) |
| Copy back rows and files, hash-check | minutes | about 1 h | **Estimate** |
| Checks, client confirmation, tear-down | 2–4 h of staff time | 2–4 h | **Estimate** |
| **Building the tool** | 1–2 developer-weeks, including tests on the two-company test setup | — | **Estimate** |

### 5.2 A client deletes something by accident

- **Trash covers most cases.** There is no automatic purge (D-041).
- **Permanent deletion** by a company admin (D-041). **Recommendation: yes, backups may be used to recover it within the backup window, explicitly and with limits:**
  1. The permanent-delete dialog asks for the **reason category**: "clean-up/mistake", "disposal under our retention schedule" or "data subject erasure request".
  2. The dialog says plainly: "This Document is gone from Officetul now. Our backups keep a copy for up to 120 days. Only a company admin can ask Officetul Support to recover it during that time."
  3. **Recovery is refused** for "disposal" and "erasure" deletions. Those stay deleted, and the purge ledger re-applies them after any restore.
  4. A request comes from a company admin through Support access rules (D-057). It is logged in the company's Activity log. Whether it is charged is a pricing question (D-018).
- **Malicious admin.** One company admin purges many Documents. Detection: the Activity log, plus an alert to the other company admins on any permanent deletion of more than 20 Documents a day (**Recommendation**). Recovery uses §5.1 if another authorised person at the company asks. A **7-day "purge pending" delay** that a second admin can cancel is worth considering in the retention release.

---

## 6. Deletion at exit, erasure and backup retention

### 6.1 What the rules say

| Source | Rule | Label |
|---|---|---|
| IRR s.44(b)(7) | At the controller's choice, **delete or return all personal data at the end of services, "including existing copies"**, unless storage is authorised by law | **Verified fact** (R10 §2) |
| NPC Circular 2023-06 s.30 | Disposal must "render further processing impossible", covering personal data "regardless of how such files are stored", including data "stored offsite" and by "encryption, or secure wiping programs". **Backups are not named.** | **Verified fact** [S35] |
| 2023-06 s.28 | Disposal policy must consider retention period, laws, contracts, the technique, and "Required documentation before the deletion" | **Verified fact** [S35] |
| 2023-06 s.29(B) | Log "Backup copies may be retained for shorter periods, while archived copies can be stored for a longer duration" | **Verified fact** [S35] |
| 2023-06 s.21; 16-03 s.7 | Backup, restoration and remedial time in the continuity plan; "Implementation of back-up solutions" | **Verified fact** [S35][S36] |
| AO 2020-017 | The form of disposal proof (e.g. a certificate) is "best left with the parties" | **Verified fact** (R10 §6) |

**No NPC text sets how long a processor's backups may hold an exited client's data.** That is legal question A11, extended in §11.

### 6.2 Industry practice: "beyond use" until backups age out

| Vendor | Deleted data in backups | Label |
|---|---|---|
| Odoo | Database deactivated 3 weeks after cancellation, then destroyed. Personal data "could remain stored for up to 12 months in those backups", which are "secured and inalterable"; Odoo commits "not to use those backup copies … for any purpose" | **Vendor claim** [S38] |
| Zoho | Deleted from the active database at the next 6-monthly clean-up; "deleted from backups after 3 months" | **Vendor claim** [S40] |
| Google Cloud | Backups "designed to expire deleted data … within six months"; overall "about six months (180 days)" | **Vendor claim** [S42] |
| Microsoft 365 | 90-day limited-function account after the subscription ends; all customer data deleted "no more than 180 days after expiration or termination". SharePoint keeps backups 14 days after deletion. | **Vendor claim** [S44][S45] |
| Atlassian | Sites deactivated 15 days after the subscription ends; data kept 60 days, then "permanently deleted". Infrastructure snapshots kept 30 days. | **Vendor claim** [S47][S48] |

**Pattern:** every vendor deletes from live systems first and lets backups **expire on schedule**. Nobody edits backups to remove one customer. Until they expire, the copies are kept "beyond use". A compliance Vault Lock makes early deletion **impossible** for us too [S11], so this is the only model available on locked backups.

**Purge ledger (Recommendation).** Deletions after a restore point must not come back.
- Each permanent deletion, disposal, erasure and exit deletion writes a small entry to a **ledger outside the database**: an append-only, versioned bucket.
- An entry holds only IDs: company, Document/Version/Original file IDs, time, reason code. **No names and no content.**
- Restore step 6 (§4.2) replays every entry after T **before** anyone can see the restored data.

### 6.3 How long to keep backups (**Recommendation**)

| Copy | Kept | Reason |
|---|---|---|
| Database point-in-time restore (RDS) | 35 days | RDS maximum [S2]. Bugs are usually noticed within weeks. |
| S3 versioning of Originals | 35 days | Fast undo; matches the database |
| AWS Backup continuous (files) | 35 days | AWS maximum for continuous [S8] |
| Daily locked copies (database + files) | 35 days | Disaster copy |
| Monthly locked copies (database + files) | **120 days** | Must outlast the **90-day Fixity check cycle** (D-061). A silently damaged Original found on day 89 still has a good copy, with a month's margin. |
| Log digests (replicated) | 10 years | D-062; hashes only |

**Longest life of any backup copy: 120 days.** That is shorter than Google, Microsoft and Odoo, and close to Zoho's 3 months.

**How this fits other rules:**
- **D-062 log retention:** the Activity log lives in the database. Backups add at most 120 days to any deleted log row. The Security log (2 years) and app logs (30 days, kept outside the database) are unaffected. Backups are **not an archive**: long-term keeping is the live system's job.
- **D-047 infected files:** quarantine is never backed up, so the 30-day rule holds.
- **D-041 no automatic purge:** unaffected. Backups only ever hold what the live system held in the last 120 days.

### 6.4 Exit timeline (**Recommendation**, lawyer to confirm)

1. **Day 0** contract ends. The company becomes read-only plus export (full tenant export, D-033) for **30 days**. R10 suggested 30–90 days.
2. **Day 30–37** live deletion of rows, every Original version, Derived files, search text, exports and the Activity log (handed over in the export, D-062). The purge ledger records it.
3. **Day 37:** **deletion certificate** listing what was deleted, any shared stored copies (D-070), and **the date the last backup containing the data expires (Day 37 + 120 = about Day 157)**.
4. **About Day 157:** backups have aged out automatically. An optional second note confirms it.
5. Officetul keeps a small **exit record** (company name, dates, export fingerprint, certificate), per D-062 and A17.

### 6.5 Crypto-shredding

- **What it is:** if each company's data were encrypted with its own key, destroying that key would make every copy, backups included, unreadable at once. NIST calls this **cryptographic erase**, a purge technique that makes "recovery of decrypted data infeasible" [S50]. AWS KMS key deletion is irreversible after a 7–30-day wait [S17].
- **Why not in the MVP:**
  - D-059 rejected per-company keys for the MVP.
  - AWS Backup and RDS encrypt per vault or instance, not per company.
  - Real crypto-shredding would need **app-level** encryption of Originals and of database payloads with per-company data keys. That is new work and a new risk: a lost key means lost data.
- **Recommendation:** revisit only if a client contract or the lawyer requires deletion from backups faster than 120 days.

---

## 7. Restore testing

### 7.1 What the rule says

NPC Circular 2023-06 s.21 requires a business continuity plan that considers "Personal data backup, restoration, and remedial time" and "**Periodic review and testing** of the business continuity plan". **No frequency is stated** [S35]. Quarterly is our choice (R10 already proposed it), in line with Atlassian's "quarterly testing of our backups" [S48].

### 7.2 Schedule (**Recommendation**)

| How often | What | Effort |
|---|---|---|
| Daily (automatic) | Backup job success, vault lock status, EventBridge setting on the bucket, KMS keys enabled and not pending deletion. Alert on any failure. | None once set up |
| Monthly (automatic) | AWS Backup **restore testing**: a random recent recovery point of the database and a sample of Originals restored, then deleted after validation [S13]. **$1.80 per recovery point** in Singapore [S19]. | 15 min to review |
| **Quarterly drill** | Half-day manual drill (§7.3), rotating the scenario: database point in time → one company's deleted Documents → whole-bucket file restore → second-Region recovery | About 4 hours for one person |
| Yearly | Full disaster drill: rebuild in the second Region, run by **the person who did not write the runbook**. Plus a ransomware tabletop exercise. | 1–2 days |

### 7.3 Quarterly drill checklist

1. Pick the scenario and a **random** restore point (not "last night").
2. Restore the database as a new instance in an **isolated network with no internet**. Mail, email capture and queues are off.
3. Restore 200 random Originals plus all Originals of the test company. **Re-hash every one** against the stored SHA-256.
4. Run Activity log chain verification against the Log digests, plus a dry run of the restore-marker tool.
5. Start the app read-only against the copy. Log in as the test company, open 10 Documents and run 5 searches.
6. Record the times: start, database available, files verified, app usable. Compare with the RTO. Record the RPO achieved (restore point vs latest available).
7. Replay the purge ledger and confirm that one known purged test Document stays gone.
8. **Tear down** and confirm the copies are deleted, because they contain client data.
9. Write the drill record and open tickets for any gaps.

**Evidence to keep:**
- the drill record: date, who, scenario, recovery point ID, times, RPO/RTO achieved, hash results, chain result, issues and tickets, sign-off by the DPO;
- AWS restore job IDs and CLI output.

Keep it at least as long as the Security log (2 years, D-062). This is also ISO 27001-style evidence (D-011).

---

## 8. Disaster scenarios

| Scenario | How we notice | Recovery path | Target |
|---|---|---|---|
| **Singapore Region outage** (hours) | AWS Health, uptime monitor | Wait. Data in the Region is intact. Tell clients. | Back when AWS is back; tell clients within 1 h |
| **Region lost for days** | Same, plus AWS statements | Owner decides to fail over. Rebuild the stack in the second Region from code; restore the latest daily copies; regenerate derived files lazily; switch DNS. | RPO 24 h, RTO 2 business days. Needs infrastructure as code (**Open question**, §12). |
| **Database corruption** (bad migration, broken table) | Errors, failed nightly chain verification, monitoring | Point-in-time restore to just before the event (§4.2) | RPO ≤ 5 min, RTO 4 business hours |
| **Our bug mass-deletes** (e.g. a release purges Trash for every company) | Activity log spike, purge-ledger volume alert, client reports | Side instance at T; copy back missing rows for all affected companies (insert-if-absent); files from versioning, else from AWS Backup continuous restore. Bug-caused purges are **not** replayed from the ledger. | RPO minutes, RTO 1 business day |
| **Ransomware or stolen main-account admin keys** | CloudTrail alarms on deletes, key-deletion scheduling, lifecycle changes; GuardDuty | Lock out the attacker (rotate everything, root MFA). Restore from the **backup account's locked vault** into a clean account or Region. Check the Activity log chain against the Log digests to find tampering. Breach procedure (D-032, 24-h client notice). | RPO 24 h, RTO 2–3 business days |
| **A client's admin deletes data maliciously** | Bulk-purge alert to other company admins (§5.2) | Per-company restore (§5.1) at the request of another authorised person at the client | 2 business days, within the 120-day window |
| **Provider account suspended or closed** (billing failure, abuse report) | AWS email, sudden access loss | Fix billing or appeal. Vault contents survive 90 days after closure, then are deleted even if locked [S11]. The backup account sits in the same organisation, so it may be affected too. Later: a third, off-AWS copy (§9.3). | Depends on AWS; **Open question** (AWS terms on suspension, see R18) |
| **Staff unavailable** (bus factor) | The other person, the senior partner | Written runbook; two named people with access; hardware MFA keys and a sealed break-glass envelope (root credentials, recovery codes) held by the senior partner; AWS root email on a company address, not a personal one; yearly drill by the non-author | Any one person can be away without blocking a restore |
| **KMS key disabled or scheduled for deletion** | CloudWatch alarm on key state [S17] | Cancel the deletion (possible during the 7–30-day wait). An RDS instance whose key is lost goes "inaccessible" and can only be restored from a backup [S5]. | Same day |
| **Backups silently not running** | Daily check (§7.2), EventBridge alarm | Fix, then take a fresh full backup | Found within 1 day |

---

## 9. Backup storage cost (Estimate)

### 9.1 Sizes

Originals = Documents × 1.26 Originals per Document (1.3 Versions × 0.97 after duplicates, R17 §6.1) × 1.5 MB.

| Tier | Companies × Documents | Originals | Derived (not backed up) | Database (whole Documents share) |
|---|---|---|---|---|
| **Pilot** (own company + one client, D-017; assumed 5,000 each) | 2 × 5,000 = 10,000 | 10,000 × 1.26 × 1.5 MB ≈ **19 GB** | ≈ 8 GB | ≈ **5 GB** |
| **Year 1** | 20 × 5,000 = 100,000 | 100,000 × 1.26 × 1.5 MB ≈ **189 GB** | ≈ 80 GB | ≈ **25 GB**: core < 1 GB, full-text 3–4 GB, Activity log 4–30 GB a year (Section 13 §6.3) |
| **Growth** | 100 × 50,000 = 5M | **≈ 9.5 TB** (R17) | ≈ 4 TB (R17) | ≈ **300 GB**: core 35 GB, full-text 130–215 GB, Activity log 18–150 GB a year |

Officetul's other apps share the database. Their size is unknown (Q13) and not included.

### 9.2 Prices used (AWS, USD, Price List API [S19])

| Item | Price |
|---|---|
| AWS Backup S3 warm storage, Singapore | $0.06 per GB-month |
| AWS Backup S3 lower-cost tier (data older than 60 days [S12]) | $0.042 per GB-month (logically air-gapped vault: $0.069 / $0.0483) |
| Cross-Region backup transfer, Singapore → Tokyo | $0.09 per GB (→ Malaysia $0.08) |
| RDS backup storage beyond the free allowance, Singapore | $0.095 per GB-month |
| S3 Standard-IA, Malaysia | $0.01242 per GB-month (Glacier Instant Retrieval $0.0045) |
| S3 Standard / Standard-IA / Deep Archive, Singapore | $0.025 / $0.0138 / $0.002 per GB-month |
| AWS Backup S3 restore | $0.024 per GB |
| Restore-testing fee | $1.80 per recovery point |
| Internet egress, Singapore | $0.12 per GB (first 10 TB) |

**Assumptions:**
- Tokyo storage priced at the Singapore rate (not checked).
- File backups hold 5% extra for deleted files still inside the window.
- The database changes 2% a day.
- Database backup storage is 1.5 × the database size, of which 0.5 × is above the free allowance.
- New uploads: Pilot 1.6 GB, Year 1 16 GB, Growth 250 GB a month.

### 9.3 Monthly cost

**Plan A (AWS Backup for database and files, recommended to about 1 TB):**

| Line | Arithmetic | Pilot | Year 1 | Growth |
|---|---|---|---|---|
| Files, Singapore continuous | Originals × 1.05 × $0.06 | 19.85 × 0.06 = **$1.19** | 198.45 × 0.06 = **$11.91** | 9,975 × 0.06 = **$598.50** |
| Files, locked copy in second Region | Same; at Growth most data is older than 60 days, so × $0.042 | **$1.19** | **$11.91** | 9,975 × 0.042 = **$418.95** (one-off tiering fee 6.3M objects × $0.012/1,000 = $75.60) |
| Files, copy transfer | New GB × $0.09 | 1.6 × 0.09 = **$0.14** | 16 × 0.09 = **$1.44** | 250 × 0.09 = **$22.50** |
| DB, RDS 35-day restore window beyond free | 0.5 × DB × $0.095 | 2.5 × 0.095 = **$0.24** | 12.5 × 0.095 = **$1.19** | 150 × 0.095 = **$14.25** |
| DB, locked copies (35 d daily, 120 d monthly) | 1.5 × DB × $0.095 | 7.5 × 0.095 = **$0.71** | 37.5 × 0.095 = **$3.56** | 450 × 0.095 = **$42.75** |
| DB, copy transfer | DB × 2% × 30 × $0.09 | 3 × 0.09 = **$0.27** | 15 × 0.09 = **$1.35** | 180 × 0.09 = **$16.20** |
| Restore testing | 2 × $1.80 + 10 GB × $0.024 | **$3.84** | **$3.84** | **$3.84** |
| Log digest replication | KB-scale hash files | < $0.10 | < $0.10 | < $0.10 |
| **Total per month** | | **≈ $8** | **≈ $35** | **≈ $1,117** |

**Plan B at Growth (files moved to S3 replication):**
- Replica: 9,975 GB × $0.01242 = $123.89.
- Transfer: 250 × $0.08 = $20.00.
- Database and testing as in Plan A: $73.20 + $3.84.
- **Total ≈ $221 a month**, plus a one-off seeding transfer of 9,500 × $0.08 = **$760**.
- Plan B in Year 1 would be about **$3.74** for files, against $25.26 in Plan A.

**Reading:**
- At Pilot and Year 1, Plan A costs **under $40 a month** and buys one tool, a lock that covers every copy, and built-in restore tests.
- Plan A grows at about **$0.10 more per GB-month** than Plan B. At about **1 TB of Originals** the difference reaches about $100 a month, which is the point to move files to Plan B.

**Not backing up Derived files:**
- At Growth that skips about 4,000 GB × $0.06 × 2 copies = **$480 a month** under Plan A (about $400 with tiering), or about $50 under Plan B.
- **Cost of regenerating them** after a total loss (OCR CPU hours, days of queue time) is an **Open question**. Measure it in the pilot. Search keeps working meanwhile, because the text is in MySQL.

**Third copy outside AWS (later, against account suspension):**
- Wasabi at $7.99 per TB-month (**Vendor claim**, R14; 90-day minimum).
- Growth: 9.975 TB × 7.99 = **$79.70 a month**, plus egress 250 GB × $0.12 = $30 a month, plus a one-off 9,500 × $0.12 = **$1,140**.
- Year 1: about $1.60 + $1.92 = **$3.52 a month**.
- Wasabi Object Lock in Singapore is unconfirmed (R14).

**Not costed here (R18):** KMS key fees, the temporary database instance hours during drills and restores, compute for a Region rebuild.

---

## 10. Recommendations summary

### MVP (before the first external client)

1. Adopt the recovery targets in §1.3 and put them in the data processing addendum as targets.
2. **Database:** RDS for MySQL 8.4, encrypted with a customer-managed key; 35-day point-in-time restore; retain-on-delete and deletion protection on.
3. **One backup account** (separate root, hardware MFA, own KMS keys) with a **compliance-locked vault** in a second Asian Region (Tokyo proposed). Daily copies kept 35 days, monthly copies 120 days, for the database and Originals.
4. **Files:** versioning 35 days; AWS Backup continuous for Originals; replicate Log digests to a compliance-locked bucket in the backup account. **Do not back up** Derived, Quarantine or Exports.
5. **Purge ledger** outside the database; replayed on every restore.
6. Restore runbook (§4.2), including the **restore marker** for the Activity log and the version-ID update after file restores.
7. Permanent-delete dialog with a reason category and the 120-day backup notice. Recovery from backup only for "clean-up/mistake" deletions.
8. Exit: 30-day export window, live deletion within 7 days, a certificate naming the backup expiry date, backups expire by about Day 157.
9. Daily backup health checks with alarms; monthly automatic restore test; **quarterly drill** with a written record; **first drill before the first external client**.
10. Security log shipped off the database in near real time.
11. Bus-factor kit: runbook, two people with access, break-glass envelope held by the senior partner.
12. **Non-AWS hosting:** XtraBackup 8.4 encrypted nightly, binlogs every 5 minutes, `rclone copy` of Originals with Object Lock to AWS Singapore. The same retention, ledger and drills apply.

### Later

- Per-company **copy-back tool** as a tested command (1–2 developer-weeks). Until then, the same steps are done by hand from the runbook.
- Plan B for files at about 1 TB of Originals, with a monthly lock-extension job.
- RDS cross-Region automated backup replication (disaster RPO in minutes) if a client needs it.
- Hourly Log digests, to shrink the post-restore verification gap.
- "Recovered after restore" Inbox items for orphaned uploads.
- 7-day purge-pending delay with second-admin cancel; bulk-purge alerts.
- A third, off-AWS copy once revenue justifies it or a client asks.
- Crypto-shredding with per-company keys, only if a contract requires faster backup deletion.

---

## 11. Open questions

**Technical:**
1. What does Officetul back up today, how often, where, and has anyone restored from it (Q13)?
2. Tokyo or another second Region? Malaysia is cheaper ($0.08 transfer, lower S3 prices), but the AWS pages disagree on its air-gapped vault support, and it lacks Backup Audit Manager [S9][S10].
3. Real RDS point-in-time restore and AWS Backup restore durations at our sizes. Measure them in the first drill.
4. How long it takes, and how much CPU, to regenerate Derived files after losing them all.
5. The Tokyo price for AWS Backup and RDS snapshot storage (Singapore prices were used).
6. How AWS treats a suspended account and its backups before closure (R18).
7. Wasabi Object Lock in Singapore, and any Philippine provider with S3 Object Lock (R14 carry-over).

**Lawyer** (add to the [legal question list](../questionnaires/legal-and-tax-questions.md), extending A11):
- **L1.** Is "deleted from live systems within 7 days of the export window, from backups within 120 days, kept beyond use meanwhile" acceptable under IRR s.44(b)(7) "including existing copies"? We cannot delete earlier from locked backups.
- **L2.** May we restore a client's permanently deleted data on the request of one company admin? Who at the client may authorise it, and must it be in writing?
- **L3.** If a restore brings back data the client had erased for a data subject, is re-applying the deletion before anyone can see it enough? Must anything be reported?
- **L4.** Does naming a second Region (e.g. Tokyo) for backups and disaster recovery in the addendum need more than listing it (AO 2017-012)? This joins A12.
- **L5.** Is a deletion certificate issued before backups expire, stating the expiry date, acceptable proof (AO 2020-017)?
- **L6.** Is a quarterly test "periodic testing" enough under 2023-06 s.21?
- **L7.** Does data lost in a restore count as a personal data breach that the client must report within 72 hours, even with no leak?
- **L8.** Does a documented gap in the Activity log after a restore (§4.3) weaken its weight under REE Rule 7?

---

## 12. Check in the Officetul code and operations (D-013 review)

- Current backup method, schedule, location, retention, encryption, and any restore test (Q13).
- Where the database runs (RDS or a VM), its exact MySQL version, total size, binlog on or off, binlog format and GTID mode.
- Whether the app connects as an admin user (RK-39). This affects insert-only logging and who can drop backups.
- AWS account layout: one account or an Organization, root MFA, who holds credentials, existing CloudTrail and GuardDuty.
- **Infrastructure as code** (Terraform, CDK, Forge or none). Without it, rebuilding in a second Region in 2 days is unrealistic.
- Where other Officetul apps store files, and whether they share buckets with Documents.
- `APP_KEY` and other secrets: are they backed up outside the server? If Officetul uses Laravel encrypted casts, a lost `APP_KEY` makes those columns unreadable even after a perfect database restore.
- Queue backend (Redis or database). Jobs in flight are lost or replayed on restore, so check that every job is idempotent.
- Whether every table has `tenant_id`; any soft deletes; existing tenant export or import scripts.
- Existing monitoring and alerting, and who receives alerts out of hours.
- Mail and other integrations: can a restored copy start with all outbound traffic switched off?

---

## 13. Decisions for the owner

Each has a recommended answer, worded so "yes" accepts it.

1. **Adopt these recovery targets, published as targets not guarantees: everyday incidents lose at most 5 minutes of work and are fixed within 4 business hours; a Region or account disaster loses at most 24 hours and is fixed within 2 business days?**
   - *Example:* at 10:05 on a Tuesday a release corrupts the Folders table. We restore the database to 10:00, check it, and the app is back by about 14:00. Uploads between 10:00 and 10:05 must be redone. → **Yes**

2. **Back up the database and Original files with AWS Backup into a separate, compliance-locked backup account in a second Asian Region (Tokyo proposed, named in the addendum), and never back up Derived files, Quarantine or Exports?**
   - *Example:* someone steals our main AWS admin keys on a Sunday and deletes the database and every file. Monday morning we restore Saturday night's locked copy from the backup account, which the attacker could not touch. → **Yes** (switch files to cheaper S3 replication at about 1 TB of Originals)

3. **Keep point-in-time restore, versioning and daily copies for 35 days and monthly copies for 120 days, so no backup copy is ever older than 120 days?**
   - *Example:* a Fixity check on 1 March finds a scan damaged since early December. The monthly copy from 1 December still exists, because monthlies live 120 days, and we restore the good file. → **Yes**

4. **Allow a company admin to ask Officetul Support to recover a permanently deleted Document from backups within those 120 days, but only for "clean-up/mistake" deletions, never for disposal or erasure deletions, and say so in the delete dialog?**
   - *Example:* on Friday an admin permanently deletes the "2024 BIR" folder thinking it was a duplicate. On Monday they ask for it back. We restore it, logged as "Restored from backup at the request of [admin]". Had they chosen "data subject erasure", the answer would be no. → **Yes**

5. **At exit: 30 days of read-only export, live deletion within 7 days after, a deletion certificate that states the date the last backup expires (about 157 days after the contract ends), and no crypto-shredding in the MVP?**
   - *Example:* a client's contract ends on 1 June. They export until 1 July. We delete everything live by 8 July and issue a certificate saying the backup copies expire by 5 November, after which nothing remains. → **Yes** (lawyer to confirm L1, L5)

6. **Keep a purge ledger outside the database and replay it after every restore, so deleted or exited data never comes back?**
   - *Example:* we restore the database to Wednesday 09:00 after a bug. A client deleted a former employee's records on Wednesday at 11:00 for an erasure request. The ledger re-deletes them before anyone logs in. → **Yes**

7. **Test restores monthly (automatic), quarterly (a half-day drill with a written record), and yearly (full rebuild in the second Region by the person who did not write the runbook), with the first drill before the first external client?**
   - *Example:* in the April drill, one developer restores the database to a random March day, re-hashes 200 files and finds the restore takes 3 hours. The record goes into the ISO evidence folder. → **Yes**

8. **After a point-in-time restore, continue the Activity log with a "restore marker" that records the lost period and the digests that prove it existed, and never rewrite or re-create lost entries?**
   - *Example:* after restoring to 09:42, a company's log shows "Restore: entries from 09:42 to 11:05 on 15 Oct were lost in incident #12; the locked digest proves 3,214 entries existed." An auditor sees the gap, its cause and its size, instead of an unexplained break. → **Yes** (lawyer to confirm L8)

---

## 14. Sources (all accessed 2026-10-20)

| ID | Source | URL | Type |
|---|---|---|---|
| S1 | Amazon RDS, Introduction to backups | https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_WorkingWithAutomatedBackups.html | Primary |
| S2 | Amazon RDS, Backup retention period | https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_WorkingWithAutomatedBackups.BackupRetention.html | Primary |
| S3 | Amazon RDS, Restoring to a specified time | https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_PIT.html | Primary |
| S4 | Amazon RDS, Replicating automated backups to another Region | https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_ReplicateBackups.html | Primary |
| S5 | Amazon RDS, Encrypting resources | https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Overview.Encryption.html | Primary |
| S6 | Amazon RDS, MySQL versions | https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/MySQL.Concepts.VersionMgmt.html | Primary |
| S7 | AWS Database Blog, "Demystifying Amazon RDS backup storage costs" (25 Oct 2022) | https://aws.amazon.com/blogs/database/demystifying-amazon-rds-backup-storage-costs | Secondary (vendor blog) |
| S8 | AWS Backup, Amazon S3 backups | https://docs.aws.amazon.com/aws-backup/latest/devguide/s3-backups.html | Primary |
| S9 | AWS Backup, Feature availability | https://docs.aws.amazon.com/aws-backup/latest/devguide/backup-feature-availability.html | Primary |
| S10 | AWS Backup, Logically air-gapped vault | https://docs.aws.amazon.com/aws-backup/latest/devguide/logicallyairgappedvault.html | Primary |
| S11 | AWS Backup Vault Lock | https://docs.aws.amazon.com/aws-backup/latest/devguide/vault-lock.html | Primary |
| S12 | AWS Backup, Backup tiering | https://docs.aws.amazon.com/aws-backup/latest/devguide/backup-tiering.html | Primary |
| S13 | AWS Backup, Restore testing | https://docs.aws.amazon.com/aws-backup/latest/devguide/restore-testing.html | Primary |
| S14 | Amazon S3, Replicating objects | https://docs.aws.amazon.com/AmazonS3/latest/userguide/replication.html | Primary |
| S15 | Amazon S3, What does S3 replicate? | https://docs.aws.amazon.com/AmazonS3/latest/userguide/replication-what-is-isnot-replicated.html | Primary |
| S16 | Amazon S3, Object Lock | https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lock.html | Primary |
| S17 | AWS KMS, Deleting a key | https://docs.aws.amazon.com/kms/latest/developerguide/deleting-keys.html | Primary |
| S18 | AWS Local Zones features (Manila row) | https://aws.amazon.com/about-aws/global-infrastructure/localzones/features/ | Primary (table read flattened) |
| S19 | AWS Price List API, offers AmazonS3 (ap-southeast-1, -3, -5), AmazonS3GlacierDeepArchive, AWSBackup, AWSDataTransfer, AmazonRDS (ap-southeast-1); published 2026-09-11 to 2026-10-20 | https://pricing.us-east-1.amazonaws.com/offers/v1.0/aws/{offer}/current/{region}/index.json | Primary |
| S20 | Percona XtraBackup 8.4 documentation | https://docs.percona.com/percona-xtrabackup/8.4/ | Primary |
| S21 | Percona XtraBackup 8.4 release notes | https://docs.percona.com/percona-xtrabackup/8.4/release-notes/release-notes.html | Primary |
| S22 | Percona XtraBackup, server version comparison | https://docs.percona.com/percona-xtrabackup/8.4/server-backup-version-comparison.html | Primary |
| S23 | Percona XtraBackup, encrypted backups; encrypted InnoDB tablespaces; incremental; compressed | https://docs.percona.com/percona-xtrabackup/8.4/encrypt-backups.html ; https://docs.percona.com/percona-xtrabackup/8.4/encrypted-innodb-tablespace-backups.html | Primary |
| S24 | Percona blog, no 9.x Innovation releases (16 Jul 2024); XtraBackup 9.7.1-rc1 notice (15 Jul 2026) | https://www.percona.com/blog/no-mysql-9-x-innovation-releases-from-percona/ ; https://docs.percona.com/new/2026/07/15/percona-xtrabackup-971-rc1-has-been-released/ | Primary (vendor) |
| S25 | MySQL 8.4, Point-in-time recovery (binlog, positions) | https://dev.mysql.com/doc/refman/8.4/en/point-in-time-recovery.html ; https://dev.mysql.com/doc/refman/8.4/en/point-in-time-recovery-positions.html | Primary |
| S26 | MySQL 8.4, Binary log options; binary log encryption | https://dev.mysql.com/doc/refman/8.4/en/replication-options-binary-log.html ; https://dev.mysql.com/doc/refman/8.4/en/replication-binlog-encryption.html | Primary |
| S27 | MySQL Shell 8.4, dump and load utilities | https://dev.mysql.com/doc/mysql-shell/8.4/en/mysql-shell-utilities-dump-instance-schema.html ; https://dev.mysql.com/doc/mysql-shell/8.4/en/mysql-shell-utilities-load-dump.html | Primary |
| S28 | MySQL 8.4, mysqldump | https://dev.mysql.com/doc/refman/8.4/en/mysqldump.html | Primary |
| S29 | MySQL Enterprise Backup | https://www.mysql.com/products/enterprise/backup.html | Primary (vendor) |
| S30 | restic references (encryption); restic forget docs; restic forum (Oct 2023) | https://restic.readthedocs.io/en/stable/100_references.html ; https://restic.readthedocs.io/en/stable/060_forget.html ; https://forum.restic.net/t/any-way-to-get-restic-to-work-with-s3-object-locking-or-a-read-only-key/6858 | Primary / forum |
| S31 | restic rest-server README | https://github.com/restic/rest-server | Primary (project) |
| S32 | rclone S3 backend and changelog | https://rclone.org/s3/ ; https://rclone.org/changelog/ | Primary (project) |
| S33 | Kopia, Ransomware protection | https://kopia.io/docs/advanced/ransomware-protection/ | Primary (project) |
| S34 | Backblaze B2, Object Lock with the S3-compatible API | https://www.backblaze.com/docs/en/cloud-storage-enable-object-lock-with-the-s3-compatible-api | Primary (vendor) |
| S35 | NPC Circular 2023-06, Security of Personal Data (ss.21, 28–31), Internet Archive copy | https://web.archive.org/web/20261012063023/https://privacy.gov.ph/wp-content/uploads/2024/03/NPC-Circular-Repeal-16-01-Signed.pdf | Primary (archive) |
| S36 | NPC Circular 16-03, Personal Data Breach Management (s.7), Internet Archive copy | https://web.archive.org/web/20261012063023/https://privacy.gov.ph/wp-content/uploads/2022/01/sgd-npc-circular-16-03-personal-data-breach-management.pdf | Primary (archive) |
| S37 | Odoo Cloud Hosting SLA | https://www.odoo.com/cloud-sla | Vendor |
| S38 | Odoo Privacy Policy | https://www.odoo.com/privacy | Vendor |
| S39 | Odoo.sh documentation (branches, backups) and FAQ | https://www.odoo.com/documentation/18.0/administration/odoo_sh/getting_started/branches.html ; https://www.odoo.sh/faq | Vendor |
| S40 | Zoho Security | https://www.zoho.com/security.html | Vendor |
| S41 | M-Files Cloud | https://www.m-files.com/editions/m-files-cloud/ | Vendor |
| S42 | Google Cloud, Data deletion | https://docs.cloud.google.com/docs/security/deletion | Vendor |
| S43 | Google Workspace, Recover deleted Drive files | https://knowledge.workspace.google.com/admin/drive/recover-deleted-files-and-folders-for-drive-users | Vendor |
| S44 | Microsoft, Data retention, deletion and destruction in Microsoft 365 | https://learn.microsoft.com/en-us/compliance/assurance/assurance-data-retention-deletion-and-destruction-overview | Vendor |
| S45 | Microsoft, SharePoint data deletion | https://learn.microsoft.com/en-us/sharepoint/sharepoint-data-deletion | Vendor |
| S46 | Microsoft Services Agreement (consumer) | https://www.microsoft.com/en-us/servicesagreement | Vendor |
| S47 | Atlassian Trust, Data management | https://www.atlassian.com/trust/security/data-management | Vendor |
| S48 | Atlassian Trust, Security practices | https://www.atlassian.com/trust/security/security-practices | Vendor |
| S49 | Atlassian, Overview of Atlassian backups | https://support.atlassian.com/organization-administration/docs/overview-of-atlassian-backups/ | Vendor |
| S50 | NIST CSRC Glossary, "Cryptographic Erase" (citing SP 800-88r2) | https://csrc.nist.gov/glossary/term/ce | Primary |
| S51 | Internal: R10 (privacy), R13 (uploads), R14 (log, Object Lock stores, Wasabi/B2 prices), R16 (search), R17 (data model, sizing, storage layout), Sections 09, 11, 13, 16; DECISION-LOG D-008 to D-075 | research/, sections/, DECISION-LOG.md | Internal |
