# 04 — ISO Certification Strategy

## 1. Purpose

Turn "the senior partner is aiming for ISO certification" into a **clear, costed goal**, and define what the Documents app does and doesn't contribute to it.

## 2. Status

**Future goal, owned by the senior partner; not blocking (D-028, 2026-10-19).** The company intends to pursue ISO certification in the future. The owner has handed the choice of standard and timing to the senior partner. The Documents app stays ISO-ready (D-011), and the claims rule in §10 applies by default. Research: [R03](../research/R03-iso-standards-primer.md) (2026-10-18). Owner decisions so far: D-011 (design the Documents app for company-level ISO/IEC 27001 evidence; decide the goal here).

## 3. Executive Summary

- **Software cannot be ISO-certified. A company can.** An accredited certification body audits the company's *management system*, i.e. its policies, risk decisions, people and processes, against a requirements standard. "Officetul is ISO certified" is not a valid claim. "[Company] holds ISO/IEC 27001:2022 certification for the development, hosting and support of Officetul" is. *(Verified fact, R03 §2)*
- **Two standards are realistic:**
  - **ISO/IEC 27001:2022** (information security) is what clients' security questions and data-privacy expectations point to, for a company that hosts clients' documents.
  - **ISO 9001:2026** (quality) mostly helps with government and corporate tenders. The 2026 edition was published 16 September 2026; new certifications must use it from 31 March 2028. *(Verified fact)*
- **ISO 30301 (records management) is not worth pursuing.** A new edition is about to replace it, and no accredited certification offering was found. ISO 15489 and 16175 are good product checklists, not certificates. *(Verified fact / Estimate)*
- **In the Philippines:** PAB-accredited certification bodies exist for ISO 9001 (TÜV, SGS, Bureau Veritas, DQS and others). **None is PAB-accredited for 27001.** 27001 certificates come from bodies accredited abroad, e.g. BSI Group Philippines. *(Verified fact)*
- **It's a programme, not a purchase.** The cycle is stage 1 and stage 2 audits, then yearly surveillance audits, then recertification every 3 years. Practitioners suggest **6–12 months** from zero for an SME *(Estimate, unverified)*. No credible Philippine SME price was found. Government budget ceilings of ₱400,000–469,000 over 3 years cover auditor fees only *(Estimate)*.
- **The Documents app helps with evidence, not certification.** It can hold policies, control document versions, record approvals, log access, and support backups and retention. It cannot do the risk assessment, management review, internal audit or the certification audit (see the [ISO control mapping](../appendices/iso-control-mapping.md)).

**Recommendation:** pursue **ISO/IEC 27001:2022 for the company**, with scope "development, hosting and support of Officetul", **when a paying client or tender asks for it, or after the Documents pilot**, whichever comes first. Until then, keep the app's controls 27001-ready (D-011) and get 2–3 certification-body quotes. Consider ISO 9001:2026 only if tenders require it.

## 4. Business Context

| Goal (from R03 §6) | Who is certified | What it would mean for us |
|---|---|---|
| **A.** Our company's quality (ISO 9001:2026) | Our company | Helps win tenders; doesn't address client data security |
| **B.** Helping clients with their own ISO | Nobody (clients get certified) | A selling point for the Documents app. The next release's approval chains and review dates matter most. |
| **C.** Our company's information security (ISO/IEC 27001:2022) | Our company, for Officetul | Answers "can we trust you with our documents?"; strongest fit for a cloud ERP vendor |

The owner reported that the senior partner is "aiming to be ISO certified, the company or the software, not yet decided" (Round 2 Q5).

## 5. Verified Research Findings

| Fact | Label | Source |
|---|---|---|
| ISO 9001:2026 published 2026-09-16; the 2015 edition is withdrawn; 2015 certificates are valid until 2029-09-30; new certifications to 2026 only from 2028-03-31 | Verified fact | R03 §1, [S-012] |
| ISO/IEC 27001:2022 plus Amd 1:2024 is current; no new edition in progress | Verified fact | R03 §1 |
| IAF closed on 2026-01-01; its successor is Global ACI, and the "IAF MLA" is now the "Global ACI MRA" | Verified fact | R03 §2 |
| The PAB directory lists QMS, EMS, OH&S and food-safety scopes only, with no ISMS (27001) | Verified fact | R03 §3.2, [S-013] |
| BSI Group Philippines certified BPI to 27001:2022 and 27701:2019 (announced 2026-09-03) | Verified fact | R03 §3.2 |
| Cycle: stage 1 → stage 2 → certificate → yearly surveillance → recertification in year 3 | Verified fact | R03 §4 |
| One certification body requires the system to have run for at least 3 months, including an internal audit and management review, before assessment | Verified fact (one body) | R03 §4 |
| ISO/IEC 27701:2025 (privacy) is now a standalone certifiable standard, but outside the Global ACI MRA | Verified fact | R03 §1 |
| Philippine certificate counts: unknown (needs a free IAF CertSearch login) | Open question | R03 §3.1 |

## 6. Technical Analysis

What D-011 means for the Documents app's design (detail in Sections 08, 09 and 12):

| 27001 area (clause or control title) | Documents-app support |
|---|---|
| 7.5 Documented information | Version history, the official-version marker, approvals (next release), access control on the company's own policies |
| A.5.12 / A.5.13 Classification and labelling | Sensitivity field per document (D-026) |
| A.5.15 / A.5.18 / A.8.3 Access control, access rights, information access restriction | Folder permissions, opt-in link visibility (D-020), access reviews |
| A.5.33 Protection of records | Original file plus hash (D-026), retention and legal hold (next release) |
| A.5.34 Privacy and protection of PII | SPI flag and defaults (D-024) |
| A.8.10 Information deletion | Controlled disposal with evidence (next release) |
| A.8.13 Information backup | Backup and restore design (Section 15) |
| A.8.15 Logging | Activity log (D-014), protected as in Section 09 |
| A.8.5 Secure authentication | MFA via Officetul login (questionnaire Q7) |

The full mapping, with non-software controls, evidence, gaps and owners, is in [appendices/iso-control-mapping.md](../appendices/iso-control-mapping.md).

## 7. Alternatives and Trade-offs

| Option | Benefit | Cost and burden | Risk |
|---|---|---|---|
| **27001 now** | Strongest trust signal for a document host | A full information-security management system (ISMS) while also building the MVP; yearly audits | Distracts a 1–2 developer team during the build |
| **27001 when triggered** (recommended) | Same benefit, timed to demand; the app is already designed for it | Quotes and a gap check now; programme later | A deal could need it before we're ready (lead time 6–12 months, estimated) |
| **9001:2026** | Tenders; process discipline | A quality management system across the company | Doesn't answer security questions |
| **Both** | Broadest | Highest cost | Over-investment for an SME |
| **None, product-only (B)** | Lowest cost | — | Weaker trust story |

## 8. Recommendation

1. **Goal: C, ISO/IEC 27001:2022 for the company**, scoped to Officetul's development, hosting and support, **started when a client or tender asks, or after the Documents pilot**, whichever comes first. *(Partner decision required)*
2. **Meanwhile:**
   - keep the app's controls 27001-ready (D-011);
   - adopt the safe claims wording (§10);
   - **get 2–3 quotes** from accredited certification bodies (check accreditation on IAF CertSearch);
   - create a free CertSearch login to see how many Philippine companies hold each certificate (OQ-011). *(Recommendation)*
3. **Goal B as a product angle:** market the next release's approval chains, review dates and retention as "supports your ISO 9001 or 27001 document control", never as "makes you compliant". *(Recommendation)*
4. **Skip ISO 30301.** Use ISO 15489-1 and ISO 16175 as product checklists. *(Recommendation)*
5. **ISO 9001:2026 only if a tender requires it.** If so, pick a PAB-accredited certification body. *(Recommendation)*

## 9. Cost and Operational Implications

- **No published Philippine SME price.** Get quotes (OQ-012). The only available anchor is government budget ceilings of ₱400,000–469,000 over 3 years, for auditor fees only *(Estimate)*. Staff time, any consultant, and tooling are extra.
- **Ongoing:** yearly surveillance audits; internal audit and management review each year; policies and training kept current.
- **The Documents app reduces the evidence burden.** It can host the ISMS documents and records for our own company (dogfooding, D-017).

## 10. Risks and Limitations

| ID | Risk | Mitigation |
|---|---|---|
| RK-03 | **Over-claiming** ("ISO-certified software") | Claims wording below; marketing review |
| RK-18 | **A deal needs 27001 before we're ready** | Quotes and a gap check now; app already designed for it; start after the pilot at the latest |
| RK-19 | **ISO effort competes with the MVP build** | Time the programme after the pilot; consider an external consultant |

**Claims wording** *(Recommendation, R03 §7)*:

| Avoid | Use instead |
|---|---|
| "Officetul is ISO certified" / "ISO-certified DMS" | "[Company] is certified to ISO/IEC 27001:2022 for the development, hosting and support of Officetul" (only once true) |
| "Makes you ISO compliant" | "Helps you control documented information and keep evidence for ISO 9001 and ISO/IEC 27001" |
| "ISO 9001:2015" (in new material) | "ISO 9001:2026" |

## 11. Questions for the Senior Partner

See the [meeting guide](../meeting-guides/04-iso-certification.md).

## 12. Decisions Required

| Decision | Owner |
|---|---|
| Which goal: A (9001), C (27001), both, or none for now | Senior partner |
| Trigger and timing | Senior partner |
| Budget for quotes and the certification programme | Senior partner |
| Which legal entity is certified | Senior partner |

## 13. Confirmed Decisions

- D-011: the app is designed for company-level 27001 evidence.
- D-028: ISO certification is a future goal; the senior partner decides the standard and timing; not blocking.

## 14. Open Questions

OQ-003 (goal), OQ-011 (Philippine certificate counts), OQ-012 (quotes). Also: which legal entity owns Officetul, and whether any client or tender has asked for a certificate.

## 15. Dependencies

- **Feeds:** Sections 08 and 09 (approvals, audit integrity), 12 (security controls), 15 (backup and DR), 17 (MVP vs next release).
- **Depends on:** the senior partner's decision.

## 16. Sources

R03 and its sources (S1–S41), all accessed 2026-10-18. Register entries S-012 and S-013.

## 17. Next Steps

**Acceptance criteria for closing Section 04:**
- [ ] The senior partner chooses the goal and trigger
- [ ] Quotes requested (if 27001 or 9001 is chosen)
- [x] Claims wording adopted as the default rule (D-028)
