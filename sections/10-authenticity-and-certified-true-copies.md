# 10 — Authenticity and Certified True Copies

## 1. Purpose

Decide what the Documents app may **truthfully claim** about scanned and electronic documents, which features **strengthen their evidentiary weight**, and whether it should offer anything like a "certified true copy".

## 2. Status

**Owner agreed (Round 7, D-029 to D-031), 2026-10-20.** Professional review pending (D-027). Research: [R09](../research/R09-electronic-records-evidence-and-certified-copies.md) (2026-10-19). This is research, not legal advice; the questions in §14 go to a Philippine lawyer (D-027).

## 3. Executive Summary

- **A scan of paper is a copy, not an electronic original.** The Supreme Court refused electronic-document status to photocopies with hand signatures (*NPC v. Codilla*, 2007) and to fax printouts (*MCC v. Ssangyong*, 2007). *(Verified fact for the holdings; applying them to scans is an Estimate)*
- **The 2019 evidence rules help "scan and keep".** Since 1 May 2020, an accurate copy made by photography or "electronic re-recording" is admissible like the original, **unless** the original's authenticity is genuinely questioned (Rule 130 s.4). *(Verified fact for the rule)*
- **"Scan and destroy" is risky for clients.** Without the paper, they must prove it was lost or destroyed without bad faith (Rule 130 s.5). **This confirms D-008** (never advise destroying originals). *(Verified fact)*
- **Only two kinds of real certified true copy exist:** a copy attested by the **public officer who has custody** of a public record, and a **notary's copy certification**, which is not allowed for vital, public or registrable records. A company's own "CTC" is a private attestation. Printing "Certified True Copy" automatically risks falsification charges (Revised Penal Code art. 171(7) and 172). *(Verified fact; company attestation is an Estimate)*
- **eNotarization went live 19 Oct 2026, but it doesn't cover copy certification.** *(Verified fact, from the Supreme Court FAQ)*
- **The Rules on Electronic Evidence weigh how reliably a record was captured, stored and protected** (Rule 7). That maps directly onto features the app is already planning: a fingerprint (hash) at intake, unchanged originals, an activity log, access control and a written system description. *(Recommendation)*
- **A hash shows a file hasn't changed since it was taken.** It does not prove the paper was genuine or the scan complete. *(Estimate)*

## 4. Business Context

Clients will ask "is this scan as good as the original?" and "can you certify copies?". Wrong answers expose them to lost cases and expose us to misrepresentation claims. Right answers, plus evidence features, are a genuine selling point.

## 5. Verified Research Findings

| Topic | Finding | Source (R09) |
|---|---|---|
| E-Commerce Act (RA 8792) | Electronic documents are legally recognised (s.6–7), but laws requiring a particular form still apply. E-signatures are recognised if the signing method is proved (s.8). Electronic retention satisfies a legal duty to keep records only under conditions (s.13). | §1 |
| Rules on Electronic Evidence | An electronic document is the equivalent of paper (R3). Original rule for e-documents (R4). Authentication (R5–6). Weight factors (R7). Business-records exception (R8). Proof by affidavit (R9). | §2 |
| 2019 Evidence Rules | Accurate duplicates admissible like originals unless authenticity is genuinely questioned (Rule 130 s.4) | §2 |
| Scans | Treated as copies. "Scan and destroy" requires proof of loss without bad faith. | §3 |
| Wet ink / public instrument | Still needed for some acts: Civil Code art. 1358 public instruments, donations (art. 749), sale of land through agents (art. 1874), registrable deeds | §4.3 |
| Certified copies | Public custodian (Rule 132 s.24–27) or notary copy certification (2004 Notarial Rules Rule II s.4), which excludes vital, public and registrable records | §5 |
| Falsification | RPC art. 171(7): issuing an authenticated-looking "copy" of a non-existent original; RA 10175 s.4(b)(1): computer-related forgery | §5 |

## 6. Technical Analysis

Rule 7 weight factors mapped to features (R09 §6):

| What a court weighs | App feature | When |
|---|---|---|
| How it was **captured** | Capture record: source (upload, email, phone), user, time, device, email headers for emailed files | **MVP** (proposed) |
| How it was **stored**; integrity since it was first in final form | SHA-256 at intake, stored apart from the file; original never modified; OCR, thumbnails and previews kept as linked derived files | **MVP** (D-026) |
| Integrity checked over time | Scheduled re-hash ("fixity check") with logged results | **MVP** (proposed) |
| Integrity of the **system** | Append-only activity log; access control; no hard delete of originals. Hash-chaining the log is a Phase 8 choice. | MVP (log) / Phase 8 (protection level) |
| **Who** created or approved it | Identity via the Officetul login and MFA; "official version" marking logs who, when and why | MVP |
| Accurate **time** | Synced server clocks; trusted timestamps optional, later | MVP (clock) / later |
| A witness or custodian can explain the system | Per-tenant system description; draft custodian affidavit for the tenant's lawyer | **Next release** ("Evidence pack") |
| E-notarized or digitally signed PDFs | Never rewrite them (no OCR overwrite, no PDF/A conversion); show signature status and the SC verification link | MVP: don't rewrite / Later: status display |

## 7. Alternatives and Trade-offs

| Question | Options | Recommendation |
|---|---|---|
| Certified copies | (a) Stamp "Certified True Copy"; (b) "Copy issued from records", signed by a named officer, with a disclaimer; (c) nothing | **(b), later, after a lawyer reviews it.** (a) is legally dangerous. |
| Evidence pack | MVP / next release | Next release. The MVP already captures the data it needs. |
| Signed-PDF handling | Treat like any PDF / detect and protect | Never rewrite them in the MVP (cheap); show status later |

## 8. Recommendation

1. **Adopt the claims policy** (§10) for the app, marketing and the senior partner's sales pitch. A lawyer reviews the claim list before launch. *(Recommendation)*
2. **No "Certified True Copy" feature.** Later, consider "Copy issued from [Company] records": signed by a named officer with recorded authority, with a fixed disclaimer, blocked for public and vital records. *(Recommendation)*
3. **MVP evidence features:**
   - capture record;
   - hash at intake (already D-026);
   - derived files linked to the original;
   - scheduled fixity checks;
   - never rewrite signed or e-notarized PDFs. *(Recommendation)*
4. **Next release:** an "Evidence pack" export containing the original, hash, capture record, activity-log extract, system description and a draft custodian affidavit. *(Recommendation)*

## 9. Cost and Operational Implications

- Hashing and fixity checks are cheap (CPU time). The capture record is metadata only.
- The Evidence pack is a moderate feature, aimed at law firms and disputes.
- Lawyer review of the claims list is part of the D-027 sessions.

## 10. Risks and Limitations

**Claims policy** *(Recommendation, R09 §7)*:

| Avoid | Use instead |
|---|---|
| "Court-admissible" | "Designed to help you authenticate records under the Rules on Electronic Evidence" |
| "Tamper-proof" | "Tamper-evident for changes after upload" (only if the hash and log are protected) |
| "Legally valid copy" | "Digital copy for reference" |
| "BIR-compliant" | "Helps organise records you must keep for the BIR" |
| "Certified true copy" | "Copy issued from [Company] records by [officer]: not a notarial or government certification" |
| "Go paperless" | "Find paper fast" / "less paper handling" |
| "Satisfies the duty to keep originals" | (no claim) |
| "Notarized" badge on user uploads | Only for files actually verified as e-notarized |

| ID | Risk | Mitigation |
|---|---|---|
| RK-20 | **Over-claiming legal effect** in sales or UI | Claims policy; lawyer review |
| RK-21 | **A tenant presents an attested copy as a government CTC** | Disclaimer; block for public and vital records; officer authority recorded |
| RK-22 | **Processing alters signed PDFs** and breaks their signatures | Never rewrite originals; derived files only |

## 11. Questions for the Senior Partner

See the [meeting guide](../meeting-guides/10-11-legal-authenticity-and-privacy.md).

## 12. Decisions Required

Grilling Round 7, Q1–Q3.

## 13. Confirmed Decisions

| ID | Decision |
|---|---|
| D-029 | Claims policy |
| D-030 | No CTC feature; possible "Copy issued from records" later, after lawyer approval |
| D-031 | MVP evidence features; Evidence pack next release |

## 14. Open Questions

The lawyer questions are consolidated in [questionnaires/legal-and-tax-questions.md](../questionnaires/legal-and-tax-questions.md), part A.

## 15. Dependencies

- **Depends on:** D-008, D-026.
- **Feeds:** Section 08 (official-version marking), Section 09 (audit protection), Section 12 (security), Section 17 (MVP).

## 16. Sources

R09 sources S1–S22 (accessed 2026-10-19), including lawphil.net texts of RA 8792, the Rules on Electronic Evidence, the 2019 Evidence Rules, the 2004 Notarial Rules, the Revised Penal Code and the Revised Corporation Code.

## 17. Next Steps

**Acceptance criteria:**
- [x] Claims policy adopted (D-029)
- [x] CTC approach agreed (D-030)
- [x] MVP evidence features agreed (D-031)
- [ ] Lawyer review scheduled (D-027): reviewers not yet named
