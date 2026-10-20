# 15 — Backup, Retention and Disaster Recovery

## 1. Purpose

Decide how Officetul Documents **backs up, restores and recovers** a company's documents and data, and how deletion (including at contract exit) works when copies still exist in backups. The design has to be something a 1–2 person team can actually run and test.

## 2. Status

**Owner agreed (Round 13, D-079 to D-085), 2026-10-21.** Research: [R19](../research/R19-backup-retention-and-disaster-recovery.md) (2026-10-20). Lawyer questions A23–A30 are pending. What Officetul backs up today is unknown ([ERP questionnaire](../questionnaires/erp-technical-questionnaire.md) Q13).

## 3. Executive Summary

- **Recovery targets** (published as targets, not guarantees):
  - Everyday incident: at most **5 minutes** of work lost, back within **4 business hours**.
  - Disaster: at most **24 hours** lost, back within **2 business days**. *(D-079)*
- **Backups live where an attacker with our main keys can't reach them.** That means a separate backup account, a locked vault and a second region (Tokyo proposed). *(D-080)*
- **Not backed up:** Derived files (they regenerate), quarantine and exports. *(D-080)*
- **Nothing in backups is older than 120 days.** Daily copies are kept 35 days and monthly copies 120 days. *(D-081)*
- **Mistakes can be undone; erasures can't.** A permanently deleted Document can be recovered on request, unless it was deleted for disposal or a data-subject erasure. *(D-082)*
- **Exit:**
  - 30 days to export;
  - live deletion within 7 days after that;
  - a certificate stating when the last backup expires (about day 157). *(D-083)*
- **Restores never bring deleted data back and never rewrite history.** A purge ledger re-applies deletions, and a restore marker in the Activity log records any lost period. *(D-084)*
- **Restore tests:** monthly automatic, quarterly drill, yearly full rebuild. The first drill happens before the first external client. *(D-085)*

## 4. Business Context

A document system that loses documents is worse than paper. Clients also have the right to have data deleted (RA 10173) and expect it gone when they leave. Backups pull in both directions: they must keep data safe, yet never quietly bring back what was deleted. SME vendors publish daily backups and a 24-hour recovery point (Odoo Online). None that we checked promises to restore data a customer deleted themselves.

## 5. Verified Research Findings

| Finding | Source |
|---|---|
| Odoo Online: 14 backups over 3+ months on separate machines plus an immutable copy; RPO 24 h, RTO 6–24 h, "not legally binding" | R19 §1.2 |
| Atlassian: 30-day point-in-time restore, RPO 1 h, RTO 6 h; backups not used to undo customer deletions | R19 §1.2 |
| Amazon RDS uploads transaction logs every 5 minutes; point-in-time restore up to 35 days | R19 §2 |
| AWS Backup vaults can be compliance-locked; vault contents are deleted 90 days after the account closes, even if locked | R19 §3, §8 |
| NPC Circular 2023-06 s.21 requires backup, restoration and remedial time in the business continuity plan, and "periodic" testing, with no set frequency | R19 §7 |
| No NPC text says how long backups may hold an exited client's data | R19 §6 |
| AWS pages disagree on whether the Malaysia region supports the air-gapped vault | R19 §11 |

## 6. Technical Analysis

### 6.1 What is protected, and how *(D-080, D-081)*

| Data | Main protection | Backup copy | Kept |
|---|---|---|---|
| Database (all Documents tables) | Managed MySQL, 35-day point-in-time restore | Daily, separate account, locked vault, second region | 35 days daily; 120 days monthly |
| Original files | Storage versioning (D-073) + continuous backup | Daily, as above | 35 days; 120 days monthly |
| Log digests | Compliance-locked bucket (D-061) | Replicated to a locked bucket in the backup account | 10 years (D-062) |
| Derived files, quarantine, exports | — | **Not backed up** | Regenerated or short-lived |

**Non-AWS hosting:**
- an encrypted nightly database backup (XtraBackup 8.4);
- binary logs every 5 minutes;
- a locked copy of originals in AWS Singapore.

The same retention, ledger and drills apply.

### 6.2 Restore order *(R19 §4)*

1. Restore the database to the chosen point.
2. **Replay the purge ledger**, so deleted or exited data is deleted again before anyone logs in.
3. Point restored rows at the right file versions. Run the orphan sweep (D-075).
4. Run fixity checks on a sample of the restored files (D-061).
5. Verify the Activity log chain against the Log digests. Add a **restore marker** for any lost period (D-084).
6. Restart outbound email and integrations last.

### 6.3 Deletions and backups *(D-082, D-083)*

| Case | What happens |
|---|---|
| Moved to Trash | Restored by the user from Trash (D-041) |
| Permanently deleted, category **clean-up/mistake** | Recoverable from backup on a company admin's request within 120 days; logged |
| Permanently deleted, category **disposal** or **data-subject erasure** | Not recoverable; the purge ledger keeps it deleted after any restore |
| Client leaves | 30-day export → live deletion within 7 days → certificate naming the backup expiry date (about day 157) |

### 6.4 Disaster scenarios *(R19 §8)*

| Scenario | Recovery | Target |
|---|---|---|
| Database corruption or bad release | Point-in-time restore | 5 min / 4 business hours |
| Our bug mass-deletes data | Restore alongside, copy back the missing rows; files from versioning | Minutes / 1 business day |
| Stolen main-account admin keys or ransomware | Lock the attacker out; restore from the locked backup vault; check the log chain; breach procedure (D-032) | 24 h / 2–3 business days |
| Singapore region lost for days | Rebuild in the second region from code; restore daily copies | 24 h / 2 business days (needs infrastructure as code) |
| A client's admin deletes data maliciously | Per-company restore at another authorised person's request | 2 business days |
| Provider account suspended | Fix or appeal; later, a third copy outside AWS | Depends on the provider |
| Key person unavailable | Runbook; two people with access; break-glass kit held by the senior partner | Anyone can be away |

### 6.5 Restore testing *(D-085)*

| Test | How often | Record |
|---|---|---|
| Automatic restore of a recent backup, with checks | Monthly | Automatic report |
| Half-day drill: restore to a random day, re-check files, time it | Quarterly | Written record (who, when, how long, problems) |
| Full rebuild in the second region by the person who didn't write the runbook | Yearly | Written record |
| First drill | Before the first external client | Confirms the D-079 targets |

## 7. Alternatives and Trade-offs

| Question | Options | Choice |
|---|---|---|
| Backup location | Same account / same region separate account / **separate account, second region, locked** | Second region, locked: survives stolen keys and region loss for about ₱2,200 a month in year 1 |
| Backup length | 30 days / **35 days + 120-day monthlies** / 1 year | 120 days: covers late-found damage without keeping exited data long |
| Recovering deletions | Never / always / **mistakes only** | Mistakes only: helpful without undoing erasure |
| Faster exit deletion | **Wait for backups to expire** / per-company keys (crypto-shredding) | Wait, with an honest certificate; per-company keys only if a contract demands it |

## 8. Recommendation

Adopt §6. Build the purge ledger, the restore runbook and the backup health checks before the first external client, and run the first drill then.

## 9. Cost and Operational Implications

- **Backup storage (Estimate, R19 §9):**
  - about ₱500 a month at pilot and ₱2,200 at year 1;
  - about ₱70,000 at growth, falling to about ₱14,000 once files move to cheaper copying at about 1 TB.
- **Ops time:** the drills account for 1.5–2 hours a month within the D-087 budget.
- **Infrastructure as code** is needed for a 2-day rebuild in another region. Without it, the disaster target is unrealistic (**Open question**).

## 10. Risks and Limitations

| ID | Risk | Mitigation |
|---|---|---|
| RK-60 | **Restored data includes something the client erased** | Purge ledger replayed before anyone logs in (D-084) |
| RK-61 | **Backups silently stop** | Daily health check and alarm (R19 §7) |
| RK-62 | **No infrastructure as code**, so a region rebuild takes far longer than 2 days | Check in the code review; build it before growth |
| RK-63 | **Lost `APP_KEY`** makes encrypted columns unreadable even after a perfect restore | Keep secrets backed up outside the server (questionnaire Q32) |
| RK-25 | **Exit data lingers in backups** (from Section 11) | 120-day cap; certificate with the expiry date; beyond use meanwhile |

## 11–12. Questions and Decisions

Grilling Round 13, Q4–Q10.

## 13. Confirmed Decisions

| ID | Decision |
|---|---|
| D-079 | Recovery targets |
| D-080 | Locked backups in a separate account, second region; what isn't backed up |
| D-081 | 35-day and 120-day retention |
| D-082 | Recovery of permanently deleted Documents for mistakes only |
| D-083 | Exit timeline and deletion certificate |
| D-084 | Purge ledger and restore marker |
| D-085 | Restore testing schedule |

## 14. Open Questions

- **Lawyer (A23–A30 in the [legal question list](../questionnaires/legal-and-tax-questions.md)):**
  - deleting from backups within 120 days;
  - who may authorise a restore;
  - re-applying erasure after a restore;
  - naming Tokyo;
  - the deletion certificate;
  - testing frequency;
  - whether lost data is a breach;
  - log gaps after a restore.
- Tokyo or Malaysia as the second region (Malaysia is cheaper, but its locked-vault support is unclear).
- Real restore times at our sizes (first drill).
- The time and CPU needed to regenerate all Derived files.
- Officetul's current backups, infrastructure as code and secrets handling (questionnaire Q13, Q31–Q33).

## 15. Dependencies

Depends on D-032, D-033, D-041, D-047, D-059 to D-063, D-073 to D-078. Feeds Section 18 (cost), Section 19 (risks) and the data processing addendum (D-032).

## 16. Sources

R19 (51 sources, accessed 2026-10-20), especially the Amazon RDS and AWS Backup documentation, the Odoo, Zoho, M-Files and Atlassian backup statements, NPC Circular 2023-06 and the IRR of RA 10173.

## 17. Next Steps

- [x] Targets, backup design and retention agreed (D-079 to D-081)
- [x] Deletion and exit rules agreed (D-082, D-083)
- [x] Restore safety and testing agreed (D-084, D-085)
- [ ] Lawyer questions A23–A30 answered
- [ ] First restore drill before the first external client
