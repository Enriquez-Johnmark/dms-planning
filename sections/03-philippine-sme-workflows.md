# 03 — Document Types and Workflows

## 1. Purpose

Identify **which documents Officetul's clients actually handle**, which Officetul records they belong to, and what the first version must get right for them. This sets the MVP preset document types (D-021) and what goes into later industry packs.

Re-scoped 2026-10-18: desk research plus the owner's [client document profile](../questionnaires/client-document-profile.md), instead of interviews (D-004, D-009). Approval and revision workflows are designed in Phase 6.

## 2. Status

**Owner agreed (Round 6, D-023 to D-027), 2026-10-19.** Senior partner review via the meeting guide. Research: [R07](../research/R07-doc-types-general-sme-and-accounting.md) (general SME and accounting firms) and [R08](../research/R08-doc-types-law-and-construction.md) (law and construction), both 2026-10-18. Both agents ran out of web-search quota near the end, so some government rules remain open questions (§14). **The client document profile is not yet filled in.**

## 3. Executive Summary

- **"Official Receipt" is no longer the main proof of sale.** Since 27 April 2024 (RA 11976 and RR 7-2024), the **Invoice** proves a sale for both goods and services. Official receipts only prove payment and can't support an input-VAT claim. Leftover paper ORs may be stamped "Invoice" and used up. *(Verified fact, R07 §1, from the full RR 11-2024 text)* **D-021's "Official Receipt" preset should therefore change**, and the MVP report becomes "Expenses without an invoice or receipt".
- **About 22 general presets cover every SME,** from invoices and BIR 2307 to permits, contracts, employee documents and bonds. They are listed in the [catalogue](../appendices/document-type-catalogue.md).
- **The accounting-firm pack is setup data only** (8 types), so it can ship with the MVP. **Law and construction packs need new features**: matters and court deadlines; drawing registers, transmittals and change orders. They belong in later releases.
- **Six things the MVP must not get wrong for these firms** *(Recommendation, R08 §3)*:
  1. Large files: at least 100 MB per file, because drawing sets and court filings are big.
  2. Keep the original file unchanged, with a fingerprint (hash), and store OCR text separately. In court the scanned image counts, not the OCR text.
  3. Record whether the paper original is kept, where, and by whom.
  4. Expiry dates and key event dates (completion, matter closed), captured now even though reminders and retention come later.
  5. A revision label and status (e.g. "Rev C, For Construction") separate from the "official version" marker.
  6. Search results and previews never reveal privileged or sensitive documents to users without access.
- **Paper originals matter more than expected.** Manual and loose-leaf books, the displayed BIR COR, signed contracts, notarized instruments, signed and sealed plans, bonds and the construction logbook all have paper originals that rules require. This supports D-008.

## 4. Business Context

Officetul's clients span industries (D-003). The common foundation is the BIR, registration, permit and HR paperwork every SME has; industry differences sit on top. This matches "one horizontal module, industry packs later" (D-003, D-014).

## 5. Verified Research Findings

### 5.1 Common to every SME (R07)

| Area | Key facts | Label |
|---|---|---|
| Invoicing | Invoice replaces OR as proof of sale from 27 Apr 2024. Machine-printed ORs stopped supporting input VAT by 31 Dec 2024 at the latest. | Verified fact |
| Tax record retention | 5 years (EOPT); 10 years where a return is fraudulent or was never filed (NIRC s.222(a)) | Verified fact |
| Books of accounts | Manual and loose-leaf books kept in hard copy (RR 7-2024) | Verified fact (secondary detail) |
| Renewals | DTI business name every 5 years; Mayor's permit fees due in the first 20 days of January; SEC GIS within 30 days of the annual meeting | Verified fact / Secondary |
| Employment records | At least 3 years (DOLE checklist) | Secondary; exact rule is an open question |
| Sensitive personal information | Employee IDs, government numbers, health and 2316 records are SPI under the Data Privacy Act s.3(l) | Verified fact |

### 5.2 Accounting and bookkeeping firms (R07 §D)

They collect client submissions monthly and yearly (sales, purchases, 2307s, payroll, bank statements) and produce working papers, filings and reports. **Chasing missing client documents** is the recurring pain, which document requests (next release, D-022) address.

### 5.3 Law firms (R08 §1)

- Matter files hold pleadings, court orders, evidence, affidavits, correspondence and notarized instruments.
- **No CPRA minimum period for keeping client files was found.** The CPRA requires turning client files over when an engagement ends (s.56). *(Secondary; lawyer to confirm)*
- Notarial registers go to the Executive Judge when a notary's commission ends. *(Verified fact)*
- Court-proceeding data is SPI. Search must never leak privileged content.
- Supreme Court e-filing uses PDF/A, up to 100 MB per file. *(Secondary; official text not read)*

### 5.4 Construction (R08 §2)

- The core need is **revision control of drawings**: revision label, issue status, superseded sheets. Other key documents are transmittals, RFIs, submittals, change orders (Civil Code art. 1724 requires written approval), progress billings, the logbook (PD 1096) and the closeout set for the Certificate of Occupancy.
- **Liability runs 15 years from completion, plus 10 years to sue** (art. 1723, 1144). *(Verified fact)*
- The PCAB licence renews annually (30 June per older sources). *(Open question: current cycle)*

## 6. Technical Analysis

These are principles to carry into Phases 6–9, not designs:

- **Presets are data**: types, fields and default sensitivity are configurable per tenant. Packs are importable sets of presets.
- **Common fields on every type** (catalogue §1): sensitivity, paper-original location and custodian, expiry, event date, revision label and status, optional notarial details.
- **Original preserved plus a hash, with OCR as a separate layer**: this aligns with the Rules on Electronic Evidence and Section 09 (integrity).
- **At least 100 MB uploads, with streamed preview**: this affects the Phase 7 upload pipeline and the Phase 10 server sizing.

## 7. Alternatives and Trade-offs

| Question | Options | Trade-off |
|---|---|---|
| OR preset | Keep "Official Receipt" / replace with Invoice + Payment Receipt | Keeping it misleads users about input VAT. Replacing it with an "OR" search alias loses nothing. |
| Report name | "Expenses without a receipt" / "…without an invoice" / "…without an invoice or receipt" | The combined name is clearest and shows which is missing |
| Accounting pack | MVP / later | It's only setup data, so it's cheap to ship now |
| Law and construction packs | MVP / later | They need new entities and workflows, which would break the 4–6 month MVP |
| Common fields | Minimal / full set | The full set costs little now and avoids data migration later |

## 8. Recommendation

1. **Replace the "Official Receipt" preset** with **Invoice (purchase)**, **Sales Invoice (issued)** and **Payment Receipt (OR/CR/AR)**, searchable as "OR". Rename the report to **"Expenses without an invoice or receipt"**, showing which is missing. *(Amends D-021)*
2. **Adopt the 22 general presets** in the [catalogue](../appendices/document-type-catalogue.md), plus the **optional accounting-firm pack** in the MVP.
3. **Law and construction packs come later**, as separate releases after the MVP and pilot.
4. **Adopt the common fields and the six must-get-right rules** (§3) as MVP requirements. The technical details are designed in Phases 6–9.
5. **Book a short consultation with a Philippine lawyer and a tax adviser in Phase 5.** Use the questions collected in §14. *(Partner decision required: who and budget)*

## 9. Cost and Operational Implications

- Presets and the accounting pack are configuration, so development cost is low.
- 100 MB+ uploads and preserved originals raise storage and processing needs, which feed Phase 10 sizing.
- A professional consultation is a one-off cost. *(Estimate: one or two sessions; no figure researched)*

## 10. Risks and Limitations

| ID | Risk | Mitigation |
|---|---|---|
| RK-15 | **Presets built on outdated rules** (e.g. the OR change, PCAB cycle) | Labelled sources; professional check in Phase 5; presets are editable data |
| RK-16 | **Clients scan and discard originals** that must be kept | Paper-original field on every type; D-008 messaging |
| RK-17 | **Industry clients find the MVP too generic** (law, construction) | Common fields now; packs on the roadmap; pilot feedback |

**Limitations:** no client data yet (profile not filled in); some rules are from secondary sources; several retention periods are unverified (§14).

## 11. Questions for the Senior Partner

See the [meeting guide](../meeting-guides/03-document-types.md).

## 12. Decisions Required

Grilling Round 6: OR preset change, the preset list and packs, common fields and must-get-right rules, professional consultation.

## 13. Confirmed Decisions

| ID | Decision |
|---|---|
| D-023 | Invoice / Sales Invoice / Payment Receipt replace "Official Receipt"; report renamed "Expenses without an invoice or receipt" |
| D-024 | 22 general presets plus the accounting-firm pack in the MVP |
| D-025 | Law and construction packs after the MVP and pilot |
| D-026 | Six MVP requirements (100 MB, original plus hash, paper-original field, expiry and event dates, revision label and status, permission-safe search) |
| D-027 | Lawyer and tax adviser sessions in Phase 5 |

## 14. Open Questions

**For a tax adviser** (R07 §4):
- Can scans replace paper purchase invoices and 2307s for the 5-year period?
- Does RR 5-2014's electronic-storage standard still apply?
- Do e-invoices have their own retention rule?
- Default "review after" period: 5 or 10 years?

**For a lawyer** (R07 §4, R08 §5):
- Minimum client-file retention, and whether the firm may keep copies after turnover.
- Notarial records custody.
- Cloud hosting of privileged files: is client consent needed?
- AMLA coverage of accounting and law firms.
- DOLE, PhilHealth and Pag-IBIG retention periods.
- Whether LGUs accept digital logbooks.

**Research gaps** (search quota ran out):
- Current PCAB cycle; RA 12009 IRR warranty and retention terms; COA retention for contractors.
- FSIC validity; official SC e-filing rule text.

These are added to [OPEN-QUESTIONS.md](../OPEN-QUESTIONS.md) as OQ-021 to OQ-023.

## 15. Dependencies

- **Depends on:** D-003, D-008, D-014, D-021; the client document profile, when filled in.
- **Feeds:** Section 05/08 (lifecycle, revision label vs official version), Section 06/07 (file size, OCR layer), Section 09 (hash, integrity), Section 11 (legal questions), Section 17 (MVP).

## 16. Sources

R07 and R08 source lists (all accessed 2026-10-18).

## 17. Next Steps

**Acceptance criteria for closing Section 03:**
- [x] MVP preset list agreed (D-023, D-024)
- [x] Industry pack timing agreed (D-025)
- [x] Common fields and must-get-right rules agreed (D-026)
- [x] Professional consultation agreed in principle (D-027); who is still to be named
