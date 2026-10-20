# 11 — Philippine Legal and Regulatory Research

## 1. Purpose

Record the legal obligations that shape the Documents app and the company that runs it: **data privacy**, **electronic records**, **tax records** and **industry rules**. Separate legal requirements from recommendations, and list what a lawyer or tax adviser must confirm.

## 2. Status

**Owner agreed (Round 7, D-032 to D-034), 2026-10-20.** Professional review pending (D-027). Research: [R10](../research/R10-data-privacy-for-saas-dms.md) (data privacy, 2026-10-19), [R09](../research/R09-electronic-records-evidence-and-certified-copies.md) (electronic records), and [R01](../research/R01-ph-sme-market-and-demand-drivers.md), [R07](../research/R07-doc-types-general-sme-and-accounting.md), [R08](../research/R08-doc-types-law-and-construction.md) (tax and industry). privacy.gov.ph blocked automated access, so NPC texts were read from Internet Archive copies. **This is research, not legal advice.**

## 3. Executive Summary

- **Officetul is a "personal information processor" (PIP); its clients are "controllers" (PICs).** A storage provider acts as a PIP (NPC Circular 2023-06 s.10). **Most security duties apply to both** *(Verified fact, R10 §1–2)*:
  - MFA or similar for access to sensitive data;
  - secure email;
  - backups and continuity planning;
  - log retention;
  - vulnerability management.
- **A contract is required.** Click-through terms plus a **data processing addendum** can serve as the outsourcing agreement (NPC Advisory Opinion 2020-018), but they must contain the IRR s.44 items: hosting location, sub-processors, instructions, breach help, and delete-or-return at the end. *(Verified fact)*
- **Our discovering a breach can start the client's 72-hour notification clock** (IRR s.38(a); Circular 16-03). We must tell clients fast; **24 hours** is the recommended contract term. *(Verified fact for the rule; Recommendation for 24 hours)*
- **The company likely must register with the NPC as a processor and name its own Data Protection Officer** (Circular 2022-04 s.5(B); 2023-06 s.4). *(Verified fact for the rule; applying it to us is for the lawyer to confirm)*
- **"Never auto-delete" is fine during a subscription but not after it ends.** At the end, the client chooses delete or return, including backup copies. *(Verified fact)*
- **Hosting in Singapore is allowed** (Advisory Opinion 2017-012); the company stays accountable. *(Verified fact)*
- **Electronic records:** scans are copies; keep paper originals (Section 10).
- **Tax records:** 5 years (EOPT), with hard copies required for manual and loose-leaf books (Section 03).

## 4. Business Context

Officetul already holds client data in its ERP apps, so the company is probably **already a processor today**. The Documents app raises the stakes: it holds IDs, tax returns, payroll, health and court records, which are **sensitive personal information**.

## 5. Verified Research Findings

### 5.1 Data Privacy Act duties (R10 §10)

| Requirement | Who | Source | What it means for us |
|---|---|---|---|
| Outsourcing contract with IRR s.44 contents | Client ensures; we sign | IRR ss.43–44; AO 2020-018 | Standard data processing addendum at sign-up |
| Process only on the client's instructions | Us | IRR s.44(b)(1) | No secondary use of client documents (e.g. no AI training on them) |
| Sub-processor control | Us | IRR s.44(b)(4) | Published list (hosting, email intake, OCR); notice of changes |
| Hosting location stated | Us | IRR s.44(a) | Region named in the contract |
| Encryption in transit and at rest | Both | IRR s.28(g); 2023-06 s.25 | TLS; encrypted storage, backups and exports |
| MFA for access to sensitive data | Both | 2023-06 s.16 | Client-enforceable MFA; **mandatory MFA for our own admins** |
| Need-to-know access | Both | 2023-06 s.13 | Folder permissions; logged, time-limited "support access" for our staff |
| Download, print and export controls | Both | 2023-06 s.24 | Per-role permissions |
| Logging and log retention | Both | 2023-06 s.29 | Activity log; security logs kept longer |
| Tested backups and continuity plan | Both | 2023-06 s.21 | Restore tests on record; stated recovery targets (Section 15) |
| Vulnerability management | Both | 2023-06 s.32 | Patching, scanning, periodic penetration test |
| Secure email intake | Both | 2023-06 s.23 | TLS, sender checks, malware scan, quarantine (D-015) |
| Breach: we tell the client | Us (by contract) | 16-03 ss.16–17 | Within 24 hours of awareness |
| Breach: client tells the NPC and data subjects within 72 hours | Client | RA s.20(f) | Export of affected documents and people |
| Registration and DPO | Both | 2022-04; 2023-06 s.4 | Company registers as a processor with a DPO |
| Data subject access and portability | Client (we assist) | RA ss.16, 18 | Person search, access report, export (later release) |
| Erasure vs legal retention | Client | RA s.16(e) | A "blocked" state; legal hold overrides delete (next release) |
| Delete or return at contract end | Us | IRR s.44(b)(7) | Export window, then certified deletion, including backups |

### 5.2 Electronic records and evidence

See [Section 10](10-authenticity-and-certified-true-copies.md).

### 5.3 Tax, corporate and industry records

| Area | Rule | Status | Source |
|---|---|---|---|
| Tax records | 5 years (RA 11976); 10 years if a return is fraudulent or not filed | Verified fact | R07 |
| Books of accounts | Manual and loose-leaf books kept in hard copy (RR 7-2024) | Verified fact (secondary detail) | R07 |
| Invoices | Invoice is proof of sale from 27 Apr 2024 | Verified fact | R07 |
| Contract claims | 10 years (Civil Code art. 1144) | Verified fact | R01 |
| Construction liability | 15 years from completion, plus 10 to sue (art. 1723) | Verified fact | R08 |
| Law-firm client files | No CPRA minimum found; turnover on termination (s.56) | Secondary; lawyer to confirm | R08 |
| Employment records | At least 3 years (DOLE checklist) | Secondary | R07 |

## 6. Technical Analysis

Privacy and security features this section adds to the plan *(Recommendation)*:

| Feature | When |
|---|---|
| Client-enforceable MFA; mandatory MFA for our admins | **MVP**; depends on Officetul's login (questionnaire Q7) |
| Per-role download and export permission | **MVP** |
| Logged, time-limited support access for our staff | **MVP** |
| Full tenant export (originals plus metadata), for exit and portability | **MVP** |
| Person-level search and access report (data subject requests) | Next release |
| "Blocked" state, legal hold, disposal log, certified deletion | Next release (with retention) |
| No use of client documents for AI training or analytics | Policy from day one |

## 7. Alternatives and Trade-offs

| Question | Options | Trade-off |
|---|---|---|
| Breach notice to clients | "Immediately" / within 24 h / within 72 h | 72 h leaves the client no time; 24 h is defensible and achievable |
| Contract form | Negotiated per client / standard click-through plus addendum | Standard is scalable and accepted by NPC opinion |
| Hosting | Singapore cloud / Philippine data centre | Both lawful; decided in Phases 9–10 on cost and client expectations |

## 8. Recommendation

1. **Pre-launch company tasks**, before the first external client pilot (our own-company pilot can start earlier) *(Recommendation; the company's obligation)*:
   - confirm NPC registration as a processor and a named DPO;
   - adopt a standard data processing addendum with the §5.1 contents and **24-hour breach notice**;
   - publish a sub-processor list;
   - write an incident response procedure.
2. **Add the MVP privacy features** in §6. *(Recommendation)*
3. **Never use client documents for anything except serving that client.** *(Recommendation)*
4. **Take the consolidated question list** to the lawyer and tax adviser (D-027). *(Recommendation)*

## 9. Cost and Operational Implications

- **Company effort:** DPO role (can be an existing staff member), registration, contract template, incident procedure.
- **Lawyer time** to review the addendum and claims list (D-027).
- **Engineering:** MFA, export, support-access logging. These are moderate and partly depend on Officetul.

## 10. Risks and Limitations

| ID | Risk | Mitigation |
|---|---|---|
| RK-23 | **The company isn't registered or has no DPO** when the Documents app launches | Pre-launch checklist (§8.1) |
| RK-24 | **Slow breach notice** makes clients miss the NPC 72-hour deadline | 24-hour contractual commitment; incident procedure; logs |
| RK-25 | **Exit data lingers in backups** | Backup retention designed with a deletion window (Section 15) |
| RK-26 | **Law firms' professional duties** (CPRA s.27–28) conflict with cloud hosting | Lawyer question; law-firm rider; law pack later |

**Limitations:** NPC texts were read from archives; NPC Advisory 2026-02 is known only from a law-firm article; no NPC fine against a processor was found; BSP and DICT localisation rules are not covered.

## 11. Questions for the Senior Partner

See the [meeting guide](../meeting-guides/10-11-legal-authenticity-and-privacy.md).

## 12. Decisions Required

Grilling Round 7, Q4–Q6.

## 13. Confirmed Decisions

| ID | Decision |
|---|---|
| D-032 | Pre-launch company privacy tasks before the first external client |
| D-033 | MVP privacy features |
| D-034 | Question list is the professional brief |

## 14. Open Questions

Consolidated in [questionnaires/legal-and-tax-questions.md](../questionnaires/legal-and-tax-questions.md).

## 15. Dependencies

- **Feeds:** Section 12 (security controls), Section 15 (backup, exit deletion), Section 16 (Officetul login and MFA), Section 17 (MVP), Section 18 (company costs).
- **Depends on:** D-027 (professional sessions); questionnaire answers on Officetul's login.

## 16. Sources

R10 sources (RA 10173, IRR 2016, NPC Circulars 16-03, 2022-04, 2023-06, Advisory Opinions 2017-012, 2020-017, 2020-018, Advisory 2024-01), accessed 2026-10-19 via lawphil.net and Internet Archive copies of privacy.gov.ph.

## 17. Next Steps

**Acceptance criteria:**
- [x] Pre-launch company tasks agreed (D-032); **owner not yet named**
- [x] MVP privacy features agreed (D-033)
- [x] Question list approved (D-034)
