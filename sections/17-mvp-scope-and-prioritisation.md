# 17 — MVP Scope and Prioritisation

## 1. Purpose

Bring decisions D-001 to D-098 together into **one MVP scope**: what gets built, in what order, what waits, how long it takes, and what must be true before each release. This is the scope the senior partner approves before any build starts (Master plan Phase 11).

## 2. Status

**Owner agreed (Round 14, D-089 to D-098), 2026-10-21.** Research: [R20](../research/R20-mvp-scope-synthesis.md), which holds the full item-by-item inventory (§1) and the consistency check (§3). **Senior partner approval pending:** scope, timeline and two developers (D-089), via [meeting guide 18](../meeting-guides/18-costs-and-pricing.md).

## 3. Executive Summary

- **Two releases inside the MVP:**
  - the **own-company pilot** (about 74 must-have items, less those moved by D-090, D-093 to D-095);
  - the **first external client** (about 25 more).

  Everything else waits for the next release or later. *(R20 §1–§2)*
- **About 8 months to our own pilot with two full-time developers** (range 7–10 months), and about 3 months more to the first client. *(Estimate → D-089, amending D-005)*
- **Cut to fit:**
  - conveniences moved to Should (D-090);
  - folders alone decide access (D-093);
  - scan-to-email after the pilots (D-094);
  - for the first client, no staff access to content (D-095);
  - usage shown, not enforced (D-097).
- **Gaps closed:**
  - Inbox visibility (D-092);
  - sensitivity labels never decide access (D-098a);
  - the backup recovery wording (D-098d);
  - Officetul-wide changes given an owner (D-096).
- **Release gates** list the technical, testing, legal and approval items needed before each release (§6.4).

## 4. Business Context

The senior partner sells Officetul as a one-stop ERP. Documents has to work well enough on our own paper to become a live demo (D-017), and be safe enough for a client's confidential files. Since the early "4–6 months" estimate (D-005), about 70 decisions have added security, integrity, backup and operations work. This section is the honest re-plan.

## 5. Verified Research Findings

| Finding | Source |
|---|---|
| 99 must-have items across 10 epics (74 own-pilot + 25 first-client), plus 12 Should and 2 Could, before D-090 to D-097 | R20 §1 |
| Must work 57–86 developer-weeks raw, 65–98 with 15% contingency; two developers have 28–42 productive weeks in 6 months | R20 §4 |
| 31 conflicts and gaps found between decisions, now resolved or tracked | R20 §3 |
| Microsoft 365 disables Basic-auth SMTP AUTH by default at the end of December 2026 | R04 |

## 6. Technical Analysis

### 6.1 Epics *(R20 §1, §4.2; Estimate)*

| Epic | What it covers | Must effort (developer-weeks, raw) | Changes from Round 14 |
|---|---|---|---|
| E1 Capture and Inbox | Browser upload (resumable, ≤ 1 GB), phone upload, Inbox review and filing | 4.5–7 | Scan-to-email → Should (D-094); Inbox visibility (D-092) |
| E2 Documents, Versions, lifecycle | Document types and presets, Versions and states, Official version, details, Paper original, Trash | 7.5–10.5 | Stale-draft warning and Related documents → Should (D-090) |
| E3 Folders and permissions | Company guard, Access levels, Restricted and Confidential folders, role abilities, "Why can I see this?" | 5.5–7.5 | Link visibility → Should (D-093) |
| E4 Links and ERP contract | Linkable-record provider, Documents panel, events, nightly check, expenses report | 6–9 | — |
| E5 Search and OCR | MySQL full-text search, identifiers, OCR jobs, snippets | 5–7.5 | History and Trash search, synonyms, low-OCR flag, PDF/A → Should (D-090) |
| E6 File pipeline and security | Quarantine, validate, scan, promote; isolated tools; previews; download domain | 6.5–9.5 | — |
| E7 Activity log and integrity | Log writer, chain, digests, verification, fixity checks | 6.5–9 | — |
| E8 Admin, privacy and exit | MFA and sessions (with D-096), export (ops-run), exit job and certificate, purge ledger, restore marker | 4.5–7.75 | Support access → before second client (D-095) |
| E9 Ops, backup, monitoring | Storage buckets, backups, alerts, runbooks, first drill | 4.5–7.5 | — |
| E10 Pilot tooling and quality | Isolation suite, code review, OCR benchmark, search test, two-company checks | 6–10 | — |

The full item list, with decision IDs and Must/Should/Could per item, is in R20 §1. Round 14 moves out about 8–13 developer-weeks of raw effort (**Estimate**, R20 §4.6).

### 6.2 Out of scope *(R20 §2)*

- **Next release:**
  - approval chains;
  - reminders;
  - retention, legal hold and certified deletion;
  - share links and the client portal;
  - document requests;
  - the Evidence pack;
  - time-stamps on log digests;
  - person-level access reports.
- **Later:**
  - AI extraction;
  - e-signatures;
  - law and construction packs;
  - Meilisearch (on triggers);
  - per-company keys;
  - Philippine hosting (on request);
  - the in-browser PDF viewer;
  - a second scan engine.
- **Never (by decision):**
  - supplying scanners;
  - advising destruction of paper originals;
  - a "Certified True Copy" feature;
  - SMS codes;
  - AGPL tools;
  - VirusTotal;
  - per-Document permissions in the MVP;
  - check-out locking;
  - automatic purge.

### 6.3 Timeline *(D-089; Estimate)*

| Milestone | When (two full-time developers) |
|---|---|
| Gate 0: scope approved, questionnaire answered, code review | Month 0 |
| Officetul platform work done (D-096) | Before Gate A |
| Gate A: own-company pilot starts (2–4 weeks, D-017) | About month 8 (7–10) |
| Gate B: first external client | About month 11 (10–14) |
| Second external client: support-access workflow ready (D-095) | After Gate B |

With one developer, everything takes about twice as long, and the two-person duties (D-057, D-085, D-087) can't be met.

### 6.4 Release gates *(R20 §5, amended)*

**Gate 0 — before the build:**
- the senior partner approves the scope, timeline and team (D-089), and signs off Section 01;
- the ERP technical questionnaire is answered;
- repository access is granted;
- the code review (D-013) is held in the first two weeks.

**Gate A — own-company pilot:**
- all own-pilot Must items are built;
- the isolation suite is green and one manual two-company check is done (D-052, D-091);
- the database is on MySQL 8.4;
- hosting is confirmed;
- the processing server is isolated;
- point-in-time restore and versioning are on;
- alerts go to the named on-call people;
- Officetul platform work is done (D-096);
- Inbox visibility is enforced (D-092).

**Gate B — first external client:**
- *Technical:*
  - all first-client Must items are built;
  - the chain, digest and verification are running (only then the "tamper-evident" claim);
  - fixity checks are running;
  - locked backups are in place;
  - the purge ledger and restore marker work;
  - the first restore drill is done;
  - MFA is required for admins;
  - the interim support policy is in place (D-095);
  - the exit runbook has been tested on our own company.
- *Testing:* the OCR benchmark (D-050) and the search and leak test (D-068) pass.
- *Legal and privacy:*
  - NPC registration and a named DPO (OQ-024);
  - the data processing addendum is signed;
  - the sub-processor list is published;
  - the incident procedure is written;
  - the lawyer has reviewed the claims and the addendum, or the open items are recorded as accepted risks.
- *Approvals:*
  - the pilot client is chosen;
  - pricing is decided, or written free-pilot terms are agreed;
  - code-review findings are closed.

## 7. Alternatives and Trade-offs

| Question | Options | Choice |
|---|---|---|
| Fit the timeline | Keep everything, take 10+ months / **cut conveniences and re-baseline to ~8 months** / cut security work | Cut conveniences, keep security and integrity: they protect clients and the claims we make |
| Team | One developer / **two** | Two: half the time, and the two-person duties can be met |
| Pilot environment | Separate staging copy / **production, switched on for us only** | Production: real conditions, with the isolation checks first |
| Support for client 1 | Full workflow / **screen-share only** | Screen-share: saves about 2 weeks; workflow before client 2 |

## 8. Recommendation

The senior partner approves this scope, the ~8-month timeline and two full-time developers (D-089). Then the remaining discovery phases follow: architecture, risk register and roadmap (Sections 19–21), and the handoff.

## 9. Cost and Operational Implications

- **Developer time:** about 65–98 developer-weeks of Must work, including contingency. About 49–72 of those come before our own pilot (**Estimate**).
- **Officetul platform work:** about 0.5–8 developer-weeks, owned by the Officetul maintainer (D-096).
- **Company tasks** (not developer time): NPC registration and DPO, the data processing addendum, the incident procedure, lawyer and tax sessions, the client document profile, and pricing (R20 §4.4).
- Running costs are in Section 18.

## 10. Risks and Limitations

| ID | Risk | Mitigation |
|---|---|---|
| RK-68 | **Timeline slips silently** | Re-baselined (D-089); track the own-pilot burn-down weekly |
| RK-69 | **Officetul platform work lands on the Documents developers** unplanned | Owned by the Officetul maintainer (D-096) |
| RK-70 | **A confidential scan is seen in a shared Inbox** | Inbox visibility (D-092) |
| RK-71 | **The code review invalidates estimates** late | Review in the first two weeks of the build; re-estimate right after |
| RK-72 | **Senior partner can't fund two developers** | Fall back to one developer: about double the time; the two-person duties need a non-developer second person |

## 11–12. Questions and Decisions

Grilling Round 14, Q1–Q10.

## 13. Confirmed Decisions

| ID | Decision |
|---|---|
| D-089 | Two full-time developers; own pilot about 8 months after scope approval |
| D-090 | Conveniences moved to Should |
| D-091 | Own pilot on production, switched on for us only |
| D-092 | Inbox visibility: capturer and Inbox reviewers; rejected items deleted after 30 days |
| D-093 | Folders alone grant access in the MVP |
| D-094 | No own mail service; scan-to-email after the pilots |
| D-095 | First client: no staff access to content |
| D-096 | Officetul platform work owned by the Officetul maintainer |
| D-097 | Usage shown, not enforced |
| D-098 | Housekeeping fixes |

## 14. Open Questions

- Senior partner approval of scope, timeline and team (D-089), and pricing (OQ-020).
- The ERP technical questionnaire (Q1–Q33), including OQ-025 for link visibility.
- NPC registration and DPO (OQ-024); lawyer and tax sessions (D-027).
- The pilot client's copier check (OQ-019), for scan-to-email.

## 15. Dependencies

Depends on all decisions D-001 to D-098. Feeds Section 19 (risk register), Section 20 (architecture) and Section 21 (roadmap).

## 16. Sources

R20 (internal synthesis of the decision log, sections and research notes, 2026-10-21) and R04.

## 17. Next Steps

- [x] MVP scope agreed by the owner (D-089 to D-098)
- [ ] Senior partner approves scope, timeline and team
- [ ] Phase 12: architecture, risk register, roadmap (Sections 19–21)
