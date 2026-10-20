# 09 — Activity Log and Integrity

## 1. Purpose

Decide how Officetul Documents records **what happened, who did it and when**, in a form that a client, an auditor or a court can trust, and how it shows that stored files **have not changed** since they arrived. The section was called "Audit trails and integrity"; it now uses the agreed term **Activity log** ([GLOSSARY.md](../GLOSSARY.md)).

## 2. Status

**Owner agreed (Round 11, D-060 to D-063), 2026-10-20.** Research: [R14](../research/R14-activity-log-and-integrity.md), with the permissions side in [R15](../research/R15-tenant-isolation-and-permissions.md) (both 2026-10-20). The lawyer has not yet reviewed the log retention periods (§14).

## 3. Executive Summary

- **Two logs.** The **Activity log** belongs to the client company and records what happened to its Documents. The **Security log** is Officetul's own record of logins, two-factor checks, refused access and Support access. *(Recommendation → D-060)*
- **No document content and no search terms** go into either log. Downloads and exports are logged one by one. Views are grouped per person, per Version, per 10 minutes. *(Recommendation → D-060)*
- **No log, no access.** If a download's log entry can't be saved, the download is refused. *(Recommendation → D-060)*
- **Tamper-evidence at SME cost:**
  - the app can only add log entries, never change or delete them;
  - entries are chained by fingerprint;
  - a daily **Log digest** goes to locked storage that nobody, including Officetul, can delete.

  Running cost is pennies a month per company (**Estimate**). *(Recommendation → D-061)*
- **Fixity checks** re-read every Original file every 90 days, plus a nightly random sample. A full pass over 1 TB costs about **$0.20** in storage requests on our own worker (**Estimate**). *(Recommendation → D-061)*
- **The Philippine rules set no log period.** NPC Circular 2023-06 s.29 says only "as long as deemed necessary", with security logs kept longer than general logs. We therefore set our own periods, and the lawyer confirms them. *(Verified fact; D-062)*
- **Wording:** the product may say "tamper-evident for changes after upload" only once the chain and the daily digest are running. It never says "tamper-proof", "immutable" or "court-admissible". *(D-061, extends D-029)*

## 4. Business Context

Clients digitise paper because they want to find, trust and prove their records. A tax audit, a labour case or a contract dispute asks two questions: *is this the file you received?* and *who touched it?* The Rules on Electronic Evidence don't prescribe a log format. A judge weighs "the manner or method in which it was generated, stored or communicated, including… controls, tests and checks" (REE Rule 7 s.1). A trustworthy Activity log and regular Fixity checks are exactly those "controls, tests and checks".

Competitors keep much less:
- **Odoo Documents:** no built-in who-viewed or who-downloaded log was found.
- **Microsoft 365 and Google Workspace:** standard plans keep access logs for 180 days and 6 months respectively.

Keeping the log for the account's life is a real difference (**Estimate**, R14 §8.1).

## 5. Verified Research Findings

| Finding | Source |
|---|---|
| NPC Circular 2023-06 s.29: logs kept "as long as deemed necessary"; security logs longer than general system logs; logs must be backed up or archived; **no number of days or years** | R14 §4.1 |
| RA 10173 s.16(c)(7): data subjects may learn when their personal information was last accessed and changed | R14 §4.1 |
| IRR s.44: a processor must allow and contribute to the client's audits | R14 §4.1 |
| REE Rule 5 s.2(c): a private electronic document can be authenticated by "other evidence showing its integrity and reliability"; the Rules never use the words "audit trail" or "log" | R14 §4.3 |
| RR 9-2009 requires an audit trail for computerised accounting systems; Documents is **not** such a system, but source documents linked to ledger entries help a client show the trail | R14 §4.2 |
| MySQL allows insert-and-read-only table grants; TRUNCATE needs the DROP privilege; triggers can block updates and deletes, but anyone with the TRIGGER privilege can drop them | R14 §2.2 |
| AWS S3 Object Lock in compliance mode: an object "can't be overwritten or deleted by any user, including the root user" | R14 §2.2 |
| No Philippine storage provider with published Object Lock was found; Alibaba (Manila) and Huawei (Philippines) WORM availability is unconfirmed | R14 §2.3 |
| AWS QLDB (a managed ledger database) ended on 31 July 2025 (secondary source) | R14 §2.2 |
| DICT published a PNPKI time-stamping policy in 2015, but its endpoint returned HTTP 404 when tested on 2026-10-20 | R14 §2.4 |
| spatie/laravel-activitylog and owen-it/laravel-auditing offer no tamper-evidence; Eloquent mass updates and deletes skip model events | R14 §3 |
| Digital-preservation practice (NDSA) checks fixity on receipt, on every move, and at fixed intervals (monthly, quarterly or yearly), and advises against sampling alone | R14 §6.1 |

## 6. Technical Analysis

### 6.1 What is logged *(D-060; R14 §1)*

| Activity log (client) | Security log (Officetul) |
|---|---|
| Inbox item captured, filed, rejected | Login, logout, failed login |
| Document created, details changed (old and new values, D-043) | Two-factor set up, used, failed, reset |
| Version uploaded, marked Official, Superseded, Withdrawn, restored (D-037, D-038) | Refused access, including attempts to open another company's IDs |
| Link and Related document added or removed | Support access requested, approved, used, ended; break-glass (D-057) |
| Folder created, moved; access changed | Session ended by timeout or by an admin |
| Viewed or previewed (grouped per 10 minutes), downloaded, exported | Fixity or chain-verification failure (also shown to the client as an alert) |
| Moved to Trash, restored, permanently deleted with reason (D-041) | |
| Duplicate warning outcome (D-042); Encrypted file accepted (D-048) | |
| Fixity results: failures, plus one summary per run | |

Each entry records the actor (or "Officetul Support – name"), the company, the time, IP and device for access events, and the object.

### 6.2 Protection *(D-061; R14 §2)*

| Layer | What it stops or reveals | MVP? |
|---|---|---|
| Insert-only database grants, with migrations run by a separate database user | An app bug or attack through the app can't change or delete entries | Yes |
| Blocking triggers | Accidental updates or deletes | Yes |
| Fingerprint chain per company, plus a per-Document link | Any later edit or gap is detected; a one-Document extract can be checked on its own | Yes |
| Daily **Log digest** in compliance-mode locked storage | A rewrite by anyone with database access, including Officetul staff, is detected | Yes |
| Separate, redactable payload (old and new values, personal details) | Personal data can be erased on instruction without breaking the chain | Yes |
| Nightly chain verification; failure treated as a security incident | Silent damage | Yes |
| RFC 3161 time-stamps on the digest (FreeTSA plus one commercial authority; PNPKI if it becomes available) | Proves when the digest existed, from outside our control | Next release |
| Weekly digest email to company admins | The client holds its own copy of the fingerprint | Next release |
| Database audit plugin recording Officetul staff queries | Watches people with production access | Next release |

**Hosting dependency:**
- **AWS Singapore:** use S3 Object Lock.
- **Local data centre without Object Lock:** keep the small digest bucket in AWS Singapore anyway. It holds only hashes (**Estimate**; lawyer to confirm, §14).

### 6.3 Fixity checks *(D-061; R14 §6)*

1. Verify the fingerprint on arrival (D-046) and on every move (quarantine to storage, restores, migrations).
2. Run a rolling full pass: each night checks 1/90 of each company's files, oldest-checked first.
3. Also check a small random sample each night, so a fault doesn't wait 90 days.
4. Write each result to an insert-only results table. The Document page shows "Integrity last verified: [date]".
5. **On a mismatch:**
   - mark the Version "Integrity check failed";
   - alert Officetul ops and the company admins;
   - restore from a previous storage version or a backup, re-verify, and log every step;
   - never replace the file silently;
   - treat it as a security incident until it is explained.

### 6.4 Who sees what *(D-060; R14 §5)*

| Viewer | Sees |
|---|---|
| Anyone with history rights (Contribute and above) | Changes and Version events on Documents they can see |
| Folder **Manage** holders and company admins | Also views, downloads, IP and device |
| Company admins | A company-wide Activity page; Support access highlighted; CSV and fingerprinted export |
| Officetul staff | Client logs only through approved Support access (D-057); the Security log only for named ops staff |

The log follows the no-hints rule (D-056): nobody sees entries about Documents they can't open.

### 6.5 Time *(D-063)*

- Times are stored in UTC (`DATETIME(6)`) and shown in Asia/Manila.
- Exports carry ISO 8601 times with the offset.
- Servers sync by chrony; an alert fires if drift exceeds 1 second.
- Event order comes from the chain, not from clock times.

## 7. Alternatives and Trade-offs

| Question | Options | Choice |
|---|---|---|
| Log tool | spatie/owen-it packages / **own small writer** | Own writer (D-063): the packages offer no tamper-evidence and miss bulk changes |
| Protection | Grants only / grants + chain / **grants + chain + locked daily digest** / + time-stamps | Locked digest in the MVP; time-stamps next release (D-061) |
| View logging | Every view / **grouped per 10 minutes** / none | Grouped (Microsoft groups file-access events per 5 minutes); lawyer to confirm for law firms |
| Fixity | Sampling only / **rolling 90-day full pass + nightly sample** / monthly full pass | Rolling 90 days (NDSA's range is monthly to yearly) |
| Where a failed log write leaves the user | Allow access anyway / **refuse** | Refuse: an unlogged download is an unaccountable one |

## 8. Recommendation

Adopt §6 as the MVP Activity log and integrity design (D-060 to D-063). The claim "tamper-evident for changes after upload" is unlocked only when the chain, the locked daily digest and nightly verification are running in production.

## 9. Cost and Operational Implications

- **Storage (Estimate):** about 1.5 GB of log per company per year at 4,000 events a day; the locked copies cost about $0.04 a month per company after a year.
- **Fixity:** about $1–20 a year per 1 TB company (own worker versus AWS's checksum job).
- **Build:** medium. The Activity log is already MVP scope (D-014); the chain, digest and verifier add low-to-medium work.
- **Operations:** someone must act on chain-verification and fixity alerts (the incident procedure from D-032).

## 10. Risks and Limitations

| ID | Risk | Mitigation |
|---|---|---|
| RK-38 | **Hosting lacks Object Lock** and the digest has nowhere locked to go | Small AWS Singapore digest bucket; time-stamps next release |
| RK-39 | **Officetul's app connects to MySQL as an admin user**, so insert-only grants can't be enforced without deploy changes | Check in the code review (§15); separate migration user |
| RK-40 | **Log volume** grows faster than estimated (heavy viewing) | Grouped views; measure in the own-company pilot (D-017) |
| RK-41 | **Over-claiming** the log's legal weight in sales material | D-029 wording rules; lawyer review |

## 11–12. Questions and Decisions

Grilling Round 11, Q9–Q12.

## 13. Confirmed Decisions

| ID | Decision |
|---|---|
| D-060 | Two logs; what goes in; grouping views; who sees what; no log, no access |
| D-061 | Insert-only, chained, locked daily digest, redactable payload; fixity schedule; claim wording |
| D-062 | Retention: Activity log for the account's life; Security log 2 years; app logs 30 days; digests 10 years |
| D-063 | UTC storage, Manila display, drift alert; own log writer |

## 14. Open Questions

- **Lawyer (added to the [legal question list](../questionnaires/legal-and-tax-questions.md), A17–A21):**
  - Are 2 years for the Security log and keeping an exit record acceptable?
  - Are hashes in a 10-year locked store personal data?
  - Does an external anchor carry weight under REE Rule 7?
  - Is grouping views acceptable for law firms?
  - What period does the NPC require for incident logs?
- **Hosting (Phase 10):** AWS Singapore or a local data centre, and Object Lock availability.
- **PNPKI time-stamping:** is it live and open to private users? (Ask DICT before the next release.)
- **Officetul code (D-013):** the R14 §10 items, now in the [ERP technical questionnaire](../questionnaires/erp-technical-questionnaire.md).
- **Real event volume:** measure in the own-company pilot.

## 15. Dependencies

Depends on D-014, D-029, D-031, D-033, D-037–D-043 and D-053–D-057. Feeds Section 13 (database design), Section 14 (storage sizing), Section 15 (backups must keep the log and its fingerprints), Section 18 (cost) and the Evidence pack (D-031, next release).

## 16. Sources

R14 (82 sources, accessed 2026-10-20), especially NPC Circular 2023-06, NPC Circular 16-03, the IRR of RA 10173, the Rules on Electronic Evidence, RR 9-2009, MySQL 8.4 documentation, AWS S3 Object Lock documentation, and the NDSA fixity guidance.

## 17. Next Steps

- [x] Two logs and their contents agreed (D-060)
- [x] Protection and fixity agreed (D-061)
- [x] Retention agreed, pending lawyer (D-062)
- [x] Time and log-writer defaults agreed (D-063)
- [ ] Lawyer questions A17–A21 answered
- [ ] Hosting decision confirms where the Log digest lives (Phase 10)
- [ ] Code review confirms database users and existing logging (D-013)
