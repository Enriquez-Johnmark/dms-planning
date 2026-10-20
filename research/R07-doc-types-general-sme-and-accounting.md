# R07: Document-type catalogue for PH SMEs and accounting/bookkeeping firms

All sources were accessed on 2026-10-18. Labels used: **VF** = Verified fact (primary source read), **Sec** = Secondary (Big-4, law-firm or tax-software summary), **Est** = Estimate or typical practice, **OQ** = Open question, **Rec** = Recommendation. [Sx] refers to the source list in §5.

> Scope note: no web searches were available near the end of the research, so a few items (tax-agent accreditation rules, FSIC validity text, PhilHealth and Pag-IBIG retention) rest on secondary sources or stay open. They are flagged as such.

---

## 1. EOPT invoicing change: decision on "Official Receipt"

**Decision (Rec):** drop **"Official Receipt"** as a primary preset. Replace it with **"Invoice (purchase)"**, **"Sales Invoice (issued)"** and **"Payment receipt (OR / CR / AR)"**, a supplementary-document type. Keep "Official Receipt" as a search alias and a legacy subtype, because ORs from before the change stay within the 5-year retention window until about 2030.

| Point | Finding | Label |
|---|---|---|
| Law | RA 11976 (EOPT) amended NIRC s.237. Invoices are issued "for each sale and transfer of merchandise **or for services rendered**" valued at ₱500 or more. VAT-registered sellers issue an invoice "regardless of the amount". The ₱500 threshold is CPI-adjusted every 3 years. The Act itself never mentions official receipts. | **VF** [S1] |
| Act in force | Approved 5 Jan 2024; effective 15 days after publication. KPMG gives the effective date as **22 Jan 2024**. | **VF** [S1]; date **Sec** [S6] |
| Implementing rules | **RR 7-2024**, issued 11 Apr 2024, implements NIRC ss.113 and 235–243 on invoicing, registration and preservation of books. It took effect **27 Apr 2024**. | **VF** (issuance) [S2]; effective date **VF** via RR 11-2024 text [S3] |
| OR replaced? | **Yes.** The invoice is now the evidence of sale for both goods and services. "Official Receipt, along with … Collection Receipt, Acknowledgement Receipt and Payment Receipt … serve as **proof of payment**" and are supplementary documents. | **VF** [S3] |
| Option 1: keep using unused ORs | Allowed **until fully consumed** as supplementary documents, stamped "THIS DOCUMENT IS NOT VALID FOR CLAIM OF INPUT TAX." | **VF** [S3] |
| Option 2: convert unused manual or loose-leaf ORs | Strike through "Official Receipt" and stamp "Invoice", "Cash Invoice", "Service Invoice" or similar. Billing statements and SOAs may be converted to "Billing Invoice" the same way. No RDO approval is needed. An inventory of converted booklets was due **31 Jul 2024**. Converted documents are valid for the buyer's input tax "from April 27, 2024 until they are fully consumed". The taxpayer should get a new ATP before running out. | **VF** [S3]; **Sec** [S4][S5] |
| Unconverted manual ORs | From 27 Apr 2024 they are supplementary documents only, with **no input tax**. | **VF** [S3] |
| CRM/POS/e-receipting/CAS ORs | System-generated "Official Receipts" were valid for input tax only until **31 Dec 2024** or until the system was reconfigured, whichever came first. Extensions could not run past six months after 31 Dec 2024. After that, issuing an "Official Receipt" for a sale is treated as **failure to issue an invoice**. | **VF** [S3] |
| COR registration fee | Old CORs that show the ₱500 annual registration fee stay valid, and the fee is no longer paid. | **VF** [S3] |

**Product implications (Rec):**
- Supplier documents headed "OFFICIAL RECEIPT" with the title struck through and "INVOICE" stamped are **invoices**. OCR and classification must handle this.
- An unstamped OR dated on or after 27 Apr 2024 supports payment but not input VAT. Flag it.
- Rename the MVP report from "Expenses without a receipt" to **"Expenses without an invoice / supporting document"**. Show a sub-flag for "receipt only (no input VAT)".

---

## 2. Catalogue

### Retention baseline for tax records (applies to every BIR item below)
| Rule | Content | Label |
|---|---|---|
| NIRC s.235 as amended by EOPT | Books and "other accounting records" are kept **5 years** from the day after the filing deadline (or from the actual filing date, if late) for the year of the last entry. | **VF** [S1] |
| RR 7-2024 detail | Manual and loose-leaf books are kept as **hard copies**; computerized books are kept electronically. Records material to a pending protest or refund are kept until the case is finally resolved. The independent CPA keeps electronic copies of the AFS and working papers for 5 years. | **Sec** [S6]; the section exists **VF** [S2] |
| Exception: fraud or no return | Tax may be assessed within **10 years after discovery** of a false or fraudulent return or a failure to file (NIRC s.222(a)). The normal assessment period is 3 years (s.203). | **VF** [S10] |
| Contracts (Civil Code art. 1144) | Actions on a written contract prescribe in **10 years**. | **VF** [S31] |
| Electronic copies | RA 8792 s.13 lets an electronic copy meet a legal duty to retain, under conditions. BIR's hard-copy rule for manual books remains a conflict. | **VF** (see R01) |

"Retention" below means the **minimum legal period**. Officetul never advises destroying paper, so retention rules should drive "review after" dates, not deletion. **Rec**

### A. Sales, purchases and tax
| Document type | Source | Link to | Key metadata | Sensitivity | Retention | Keep paper original? | Expires? | MVP preset? |
|---|---|---|---|---|---|---|---|---|
| **Sales Invoice (issued)**: Cash, Charge, Service or Billing Invoice | Created | Sales invoice, customer, project | Invoice no. (serial), date, seller TIN + branch, buyer name/TIN, gross, VAT, VAT class (VATable/zero/exempt), ATP or PTI no. | Internal | 5 yrs, s.235 [S1] | Seller keeps the duplicate. Whether a scan suffices: **OQ** (books hard-copy rule [S6] vs RA 8792) | No | **Yes** |
| **Invoice (purchase)**: supplier invoice, incl. converted ORs stamped "Invoice" | Received | Expense/bill, supplier, project, asset | Supplier TIN, invoice no., date, gross, VAT, VAT class, "converted OR" flag | Internal | 5 yrs [S1] | **Yes**, keep until tax adviser confirms. Supports input VAT and deductions | No | **Yes** |
| **Payment receipt (OR / Collection / Acknowledgement Receipt)**: supplementary | Received / created | Payment, expense, customer, supplier | Receipt no., date, payer, amount, "not valid for input tax" flag, linked invoice no. | Internal | 5 yrs [S1] | Yes (**Rec**) | No | **Yes** (absorbs "Official Receipt") |
| Legacy Official Receipt (before 27 Apr 2024) | Received | Expense, payment | As above + "legacy OR" | Internal | 5 yrs; e.g. FY2024 records run to about 2030 [S1] | Yes | No | Subtype of above |
| Electronic invoice (EIS) + transmission acknowledgement | Created / received | Sales invoice, expense | Invoice no., PTI no., transmission ID/status, JSON + rendered copy | Internal | 5 yrs [S1]; any separate e-storage rule **OQ** | Not applicable (born digital) | No | Later |
| Credit / debit memo | Created / received | Sales invoice, expense | Memo no., reference invoice, amount, reason | Internal | 5 yrs | Yes (**Rec**) | No | Optional |
| Purchase order / delivery receipt / SOA | Created / received | PO, bill, supplier | Doc no., date, ref. invoice | Internal | 5 yrs (supporting record) **Est** | Optional | No | No (generic "Supporting document") |
| **BIR 2307**: Certificate of Creditable Tax Withheld at Source | Received from customers; issued to suppliers | Sales invoice/payment (received); expense (issued); customer/supplier | Payor TIN, payee TIN, period (quarter from/to), ATC, income payment, tax withheld, date received | Confidential | 5 yrs [S1]. RR 2-2015 keeps the RR 2-2006 rule on retaining **hard copies** of withholding certificates: **Sec** [S11] | **Yes**. Scanned copies go to BIR via DVD/eAFS [S12] | No | **Yes** |
| BIR 2306: Certificate of Final Tax Withheld at Source | Received / issued | Payment, supplier | As 2307 | Confidential | 5 yrs | Yes (**Rec**) | No | Optional |
| Withholding returns: 1601-C (monthly compensation), 0619-E (monthly expanded), 1601-EQ (quarterly expanded), 1604-C/1604-E (annual) + alphalists/QAP | Created (filed) | Tax period, payroll run | Form no., period, amount, filing reference, payment reference | Confidential | 5 yrs | No (e-filed); keep filing and payment confirmations | No | **Yes**, as one type "Tax return" with a form field |
| VAT 2550Q / percentage tax 2551Q + SLSP/SAWT | Created | Tax period | Form, quarter, tax due, filing ref. Monthly 2550M became optional from 2023; 2550Q is mandatory. | Confidential | 5 yrs | No | No | In "Tax return" |
| Income tax returns: 1701/1701Q, 1702/1702Q | Created | Fiscal year | Form, year/quarter, filing ref, payment ref | Confidential (SPI for individual filers: tax returns are SPI [S25]) | 5 yrs; 10 yrs if fraud [S10] | No | No | In "Tax return" |
| Tax payment proof: bank or e-payment confirmation | Received | Tax return, payment | Form, period, amount, bank ref, date | Confidential | 5 yrs | No | No | **Yes** |
| **Financial statements (audited, or with Statement of Management's Responsibility)** + BIR/eAFS receipt | Created / received | Fiscal year | FY, auditor, report date, eAFS ref. no., SEC filing date | Confidential | 5 yrs | Yes, signed originals (**Rec**) | No | **Yes** |
| Books of accounts (manual, loose-leaf, CAS) + registration proof | Created | Fiscal year | Book type, FY, registration date/ref, CAS AC/PTU no. | Confidential | 5 yrs [S1] | **Yes for manual/loose-leaf (hard copy)** [S6] | No | **Yes** |
| BIR audit/assessment correspondence (LOA, notices, PAN/FAN, protest) | Received / created | Fiscal year, tax type | Notice type, date, period, amount, response deadline | Confidential | Until finally resolved, even past 5 yrs [S6] | Yes | Has deadlines | **Yes** |
| Bank statement | Received | Bank account, period | Bank, account (masked), period | Confidential | 5 yrs (accounting record) **Est** | No | No | **Yes** |
| Contract / agreement | Created / received | Customer, supplier, employee, project, asset | Parties, TINs, effective date, end date, value, notarized? | Confidential | 10 yrs after cause of action accrues (art. 1144) [S31] | **Yes** (wet-ink or notarized originals) | Often (end/renewal date) | **Yes** |

Filing deadlines (Sec [S13]): 1601-C by the 10th of the following month (15 Jan for December); 0619-E by the 10th; 1601-EQ by the last day of the month after the quarter; 2550Q/2551Q by the 25th day after quarter-end; 2307 issued by the 20th day after quarter-end; 2316 by 31 Jan. Do not hard-code these; confirm them before building the reminders feature.

### B. Registration and permits
| Document type | Source | Link to | Key metadata | Sensitivity | Retention | Keep paper original? | Expires / renews? | MVP preset? |
|---|---|---|---|---|---|---|---|---|
| **BIR Certificate of Registration** (COR; form commonly cited as 2303, **Sec**) | Received | Company/branch | TIN + branch code, RDO, tax types, registration date, line of business | Internal | Life of registration **Est** | **Yes**. The original COR/eCOR must be **posted** at each place of business; online sellers display it electronically [S6] | No (no annual fee [S3]); update on any change | **Yes** |
| ATP / CAS Acknowledgement Certificate / e-invoicing Permit to Issue | Received | Company, branch | Permit no., serial range, issue date | Internal | Life of permit **Est** | Yes (**Rec**) | Per permit | Optional |
| **DTI Business Name Certificate** (sole proprietors) | Received | Company | BN, BN no., scope, registration date, **expiry** | Public | Life of BN **Est** | Yes (**Rec**) | **Every 5 years**. Renew from 180 days before expiry; 90-day regular period after expiry; then a 90-day late period at 50% surcharge; then cancelled | **Yes** (VF [S18]) |
| **SEC Certificate of Incorporation** + Articles + By-laws (and amendments) | Received | Company | SEC reg. no., date, amendment no. | Public (filed with SEC) | Permanent **Est** | **Yes** | No | **Yes** |
| **SEC General Information Sheet (GIS)** | Created / filed | Company, fiscal year | Year, meeting date, eFAST ref., directors/officers | Confidential (contains personal data such as names, TINs, nationality) **Est** | Permanent **Est** | Keep eFAST acknowledgement | **Annual**: within **30 calendar days after the actual annual stockholders'/members' meeting** (foreign corp: license anniversary) | **Yes**. GIS is required annually by RCC s.177 [S16] **VF**; 30-day rule **Sec** [S17] |
| AFS filed with SEC (eFAST) | Created | Fiscal year | FY, eFAST ref., BIR receipt ref. | Confidential | Permanent **Est** | As AFS | **Annual**. Dec FYE filers: **29 May 2026** for FY2025; others within 120 days of FYE; audit needed if assets or liabilities exceed ₱3M | In "Financial statements" (**Sec** [S17]) |
| Board resolutions, Secretary's Certificates, minutes, stock and transfer book | Created | Company | Date, type, subject | Confidential | Corporate records must be preserved (RCC s.73); no period stated **OQ** | **Yes** | No | Optional ("Corporate record") |
| **Mayor's / Business Permit** | Received | Company/branch | Permit no., LGU, year, **expiry (31 Dec)**, line of business | Public | Life of permit **Est** | Yes (display is usually required by LGU ordinance: **OQ**) | **Annual**. LGU taxes and fees are due "within the first twenty (20) days of January"; the sanggunian may extend up to 6 months [S19] **VF**. Permit renewed annually: **Sec** (DTI regional office) | **Yes** |
| Barangay Business Clearance | Received | Company/branch | Barangay, year, issue date | Public | **Est** 5 yrs | Yes | Annual in practice **Est**. A prerequisite for the municipal permit (LGC s.152(c)) **VF** [S19] | Recommended as "Permit/Clearance" subtype |
| Fire Safety Inspection Certificate (FSIC) | Received (BFP) | Company/branch, asset (premises) | FSIC no., premises, issue date, **valid until** | Public | **Est** 5 yrs | Yes | Generally **1 year**; required for permit renewal **Sec** [S21]. Exact IRR text **OQ** | Subtype of "Permit/Clearance" |
| Sanitary Permit | Received (city/municipal health office) | Company/branch | Permit no., year, expiry | Public | **Est** 5 yrs | Yes | Food establishments: valid 1 year, **ending 31 Dec**, renewed yearly (DOH IRR, PD 856 ch. III) **VF** [S20]. Other establishments **OQ** | Subtype of "Permit/Clearance" |
| Community Tax Certificate (cedula), corporate | Received | Company | Year, amount | Internal | **Est** | No | **Annual**, payable by the last day of February (LGC s.161) **VF** [S19] | No |
| Lease of business premises | Received | Company, asset | Lessor, address, term, rent | Confidential | 10 yrs (art. 1144) | Yes | Yes (lease end) | Under "Contract" |

### C. Employer, HR and payroll
| Document type | Source | Link to | Key metadata | Sensitivity | Retention | Keep paper original? | Expires? | MVP preset? |
|---|---|---|---|---|---|---|---|---|
| SSS / PhilHealth / Pag-IBIG employer registration certificates | Received | Company | Employer no., date | Internal | **Est** permanent | Yes | No | **Yes**: one type, "Government employer registration", with an agency field |
| Monthly contribution reports and payment proof (SSS contribution/loan collection lists, PhilHealth, Pag-IBIG remittances) | Created / received | Payroll run, period | Agency, period, PRN/reference, amount, no. of employees | Confidential | SSS: "true and accurate work records … for such period … as the Commission may prescribe" and an **Annual Register of New and Separated Employees** submitted every January [S22] **VF**. Fixed period for SSS, PhilHealth and Pag-IBIG: **OQ** | Not needed if e-filed (**Rec**) | No | **Yes** |
| SSS clearance / certificate of coverage | Received | Company | Year | Internal | **Est** | Yes | Needed before each **annual** business permit (RA 11199 IRR) [S22] **VF** | Under "Permit/Clearance" |
| Payroll register, payslips, daily time records | Created | Employee, payroll run | Employee ID, period, gross, deductions | **SPI** (salary + government IDs) | DOLE: employment records (payrolls, DTRs) kept at the workplace for **at least 3 years** [S23] **VF** (DOLE checklist, read via search excerpt). Counted from last entry: **Sec** [S24]. Section number in the Omnibus Rules: **OQ** | DOLE may ask for hard copies at inspection **Sec** [S24] → keep paper or printable copies | No | **Yes** ("Payroll record") |
| **BIR 2316**: Certificate of Compensation Payment / Tax Withheld | Created | Employee, fiscal year | Employee TIN, year, compensation, tax withheld, signed date | **SPI** (tax return-like, government-issued ID numbers) [S25] | 5 yrs [S1]; employer submits scanned duplicate originals to BIR by **28 Feb** [S11] **Sec** | Yes, signed duplicate originals | No | **Yes** |
| Employee 201 file: application/CV, employment contract, job offer, government IDs and numbers (TIN, SSS, PhilHealth, Pag-IBIG), NBI/police clearance, medical/pre-employment exam, diplomas/TOR, evaluations, disciplinary notices (NTE/decision), resignation/clearance, quitclaim, Certificate of Employment | Received / created | Employee | Employee, doc subtype, date, expiry (IDs, clearances) | **SPI** for IDs, health, offences/proceedings, education (DPA s.3(l)) [S25] | 3 yrs minimum for employment records [S23]. Claims-related: **OQ** | Yes for signed contracts, quitclaims, disciplinary decisions (**Rec**) | Some (IDs, clearances, medicals) | **Yes**: "Employee document" with subtype + SPI flag. Contents are **Est** (typical PH practice) |
| DOLE establishment registration / OSH reports | Received / created | Company | Report type, year | Internal | **OQ** | Yes | **OQ** | No (MVP) |

### D. Accounting and bookkeeping firm pack
| Document type | Source | Link to | Key metadata | Sensitivity | Retention | Keep paper original? | Expires? | MVP preset? |
|---|---|---|---|---|---|---|---|---|
| **Engagement letter** (audit, bookkeeping, tax compliance) | Created, signed by client | Customer (client), project/engagement | Client TIN, service, period covered, fee, signed date, renewal date | Confidential | Contract: 10 yrs (art. 1144) [S31] | **Yes** | Often annual | **Pack** |
| Client document request / monthly client submissions (sales and purchase invoices, 2307s received, payroll, bank statements, receipts) | Received from client | Client, period, engagement | Client, period (month/quarter), category, completeness status | Confidential / SPI (payroll) | Client's own 5-yr duty [S1]; the firm's period: **OQ** | Originals usually go back to the client (**Est**) | No | **Pack** ("Client submission" + period) |
| **Audit working papers** | Created | Engagement | Client, FY, WP ref., preparer/reviewer, report date | Confidential; "confidential and privileged", property of the CPA absent a written agreement (RA 9298 s.29) [S26] **VF** | **RR 7-2024**: the CPA keeps electronic copies of AFS and WPs **5 years** from the ITR due date or actual filing date, whichever is later **Sec** [S6]. ISA 230 guidance: ordinarily **≥5 years from the auditor's report date** **Sec** [S27]. PSA 230 / PSQM 1 wording: **OQ** | Electronic accepted by RR 7-2024 **Sec** | No | **Pack** |
| Management representation letter | Received (signed by client) | Engagement | FY, date, signatories | Confidential | With WPs | **Yes** | No | **Pack** |
| Auditor's report + signed AFS | Created | Engagement, client | FY, report date, opinion type | Confidential until filed | With WPs; client keeps its own | **Yes** | No | **Pack** |
| Client filing confirmations (eFPS/eBIRForms, eAFS, SEC eFAST) | Received | Client, return | Form, period, reference no. | Confidential | 5 yrs | No | No | **Pack** |
| Client authorization / SPA / Board resolution allowing the firm to file | Received | Client | Scope, validity | Confidential | Duration + **Est** | **Yes** (notarized) | Yes | **Pack** |
| Client KYC (IDs, SEC/DTI docs, beneficial owners) | Received | Client | Client, ID type, expiry | **SPI** | If the firm is an AMLA covered person (e.g. it manages client money or acts as company service provider): **5 years** from the transaction or account closure (RA 9160 s.9(b); RA 10365) **VF** [S29][S30]. Pure accounting/audit work may fall outside: **OQ** | Copy | IDs expire | **Pack** |
| BOA/PRC Certificate of Registration (firm) and CPA accreditation | Received | Company (firm) | Cert. no., validity | Internal | **Est** | Yes | **Every 3 years** **Sec** [S28] | **Pack** |
| BIR Tax Agent Accreditation | Received | Company/staff | Accreditation no., validity | Internal | **Est** | Yes | Yes. Basis NIRC s.6(G)/s.244, RR 11-2006 as amended **Sec** [S33]. Current RR and validity: **OQ** | **Pack** |
| SEC accreditation of external auditor (where applicable) | Received | Company | Group/category, validity | Internal | **Est** | Yes | Yes (**OQ** period) | No |

---

## 3. Recommended MVP preset document types (Rec)

### General SME: 18 presets
| # | Preset | Metadata fields |
|---|---|---|
| 1 | Sales Invoice (issued) | Invoice no., invoice type (cash/charge/service/billing), date, buyer name, buyer TIN, gross, VAT, VAT class, branch code |
| 2 | Invoice (purchase) | Supplier name, supplier TIN, invoice no., date, gross, VAT, VAT class, "converted OR" flag, "e-invoice" flag |
| 3 | Payment Receipt (OR/CR/AR) | Receipt no., kind (OR/CR/AR/payment), date, amount, payer/payee, linked invoice no., "not valid for input tax" flag, "legacy OR before 27 Apr 2024" flag. Search aliases: "Official Receipt", "OR" |
| 4 | BIR 2307 (Creditable Withholding Certificate) | Direction (received/issued), payor TIN, payee TIN, period from/to, ATC, income payment, tax withheld, date received |
| 5 | Tax Return / BIR Filing | Form no. (pick-list: 1601-C, 0619-E, 1601-EQ, 1604-C, 1604-E, 2550Q, 2551Q, 1701Q, 1701, 1702Q, 1702, other), period, amount due, filing ref., date filed |
| 6 | Tax Payment Proof | Form, period, amount, bank/channel, reference no., date |
| 7 | Financial Statements (AFS) | FY end, audited Y/N, auditor, report date, eAFS ref., SEC eFAST ref. |
| 8 | Books of Accounts | Book type (journal/ledger/subsidiary), format (manual/loose-leaf/CAS), FY, registration ref. |
| 9 | BIR Notice / Assessment | Notice type, date received, tax type, period, amount, response deadline, status |
| 10 | Bank Statement | Bank, account (last 4 digits), period |
| 11 | Contract / Agreement | Parties, counterparty TIN, type (sales/supply/lease/employment/service), start, end, value, notarized Y/N, renewal notice date |
| 12 | BIR Certificate of Registration (COR) | TIN, branch code, RDO, tax types, registration date, address |
| 13 | DTI Business Name Certificate | BN, BN no., territorial scope, registration date, **expiry date** |
| 14 | SEC Registration Documents | Subtype (Certificate of Incorporation / Articles / By-laws / amendment), SEC reg. no., date |
| 15 | SEC General Information Sheet | Year, annual meeting date, eFAST ref., date filed |
| 16 | Business Permit / Clearance | Subtype (Mayor's/Business Permit, Barangay Clearance, FSIC, Sanitary Permit, SSS clearance, other), issuing office/LGU, permit no., issue date, **valid until**, branch |
| 17 | Government Employer Records | Agency (SSS/PhilHealth/Pag-IBIG/DOLE), subtype (registration, monthly contribution/remittance, annual register), period, reference no., amount |
| 18 | Employee Document | Employee, subtype (contract, government ID, clearance, medical, 201 item, payslip, 2316), date, expiry, **SPI flag (default on)** |

"Payroll Record" (register/DTR) can stay inside #17/#18 subtypes or become a 19th preset. **Rec:** use subtypes in the MVP. Keep "Supporting Document (PO/DR/SOA)" as a generic fallback type.

### Optional accounting-firm pack: 8 presets
| Preset | Metadata fields |
|---|---|
| Engagement Letter | Client, client TIN, service type, period covered, fee, signed date, renewal date |
| Client Submission | Client, period (month/quarter/year), category (sales, purchases, 2307, payroll, bank), received date, completeness status |
| Audit Working Paper | Client, FY, WP reference, preparer, reviewer, auditor's report date, archive date |
| Management Representation Letter | Client, FY, date, signatories |
| Auditor's Report | Client, FY, report date, opinion type |
| Filing Confirmation (client) | Client, form/system (BIR, eAFS, SEC eFAST), period, reference no., date |
| Client Authorization / SPA | Client, scope, notarized Y/N, valid until |
| Professional Accreditation | Body (BOA/PRC, BIR tax agent, SEC), holder, cert. no., **valid until** |

**Sensitivity defaults (Rec):** permits and registrations "Public/Internal"; tax records "Confidential"; employee, payroll, 2316 and KYC documents "SPI" with restricted access. Under DPA s.3(l), SPI includes health, education, offences and government-issued numbers such as SSS numbers, and tax returns [S25].

**Expiry-reminder candidates (next release):** DTI BN (5 yrs), Mayor's permit (yearly, January), FSIC (about 1 yr), sanitary permit (31 Dec), SSS clearance (before permit renewal), GIS (30 days after the annual meeting), AFS (SEC schedule), BOA/PRC (3 yrs), contracts and engagement letters (end date), employee IDs, clearances and medicals.

---

## 4. Open questions

**For a tax adviser:**
1. Does a scanned image satisfy s.235 for **purchase invoices and 2307s**, or is the paper original required for the full 5 years? RR 7-2024 states a hard-copy rule for manual and loose-leaf **books** [S6]. RR 2-2015 refers to retaining **hard copies** of withholding certificates [S11]. RA 8792 s.13 points the other way.
2. Does RR 5-2014's electronic-storage standard (imaging, indexing, integrity) survive under RR 7-2024? (Carried over from R02.)
3. Do e-invoices under RMC 98-2026 carry their own storage format or retention duty for issuer and buyer? The secondary summary is silent [S9].
4. Should Officetul default retention to 5 years, or to 10 years given s.222(a)? (Rec: 10-year "review after" date, never destroy.)
5. Confirm the current form-title list and deadlines against the BIR Forms page. It is JavaScript-rendered and could not be read.

**For a lawyer:**
6. Section number and trigger for DOLE's 3-year employment-records rule. The current DOLE issuance is unclear; one commentary cites DO 238-23 [S24].
7. Retention periods for PhilHealth (RA 7875/11223) and Pag-IBIG (RA 9679) employer records. None found.
8. Whether an accounting firm doing only bookkeeping or tax work is an AMLA "covered person". This decides the 5-year KYC rule. Also whether RA 11521 changed it.
9. Whether PSA 230 and PSQM 1 as adopted by AASC/BOA keep ISA 230's "ordinarily ≥5 years" wording, and how that sits beside RR 7-2024's 5-years-from-ITR rule.
10. The current BIR tax-agent accreditation regulation and its validity period (*Maestro v. CIR*, CTA, 2025–26, was reportedly challenging it) [S33].
11. FSIC validity text in the 2019 Revised IRR of RA 9514. Whether non-food establishments need sanitary permits; this varies by LGU.
12. Whether permits must be displayed in original. COR display is confirmed [S6]; permits are by LGU ordinance.

---

## 5. Sources (all accessed 2026-10-18)

| ID | Source | URL | Type |
|---|---|---|---|
| S1 | RA 11976 (EOPT Act), full text: ss.235, 237, 48, 51 | https://lawphil.net/statutes/repacts/ra2024/ra_11976_2024.html | Primary |
| S2 | BIR, RR 7-2024 digest (issued 11 Apr 2024) | https://bir-cdn.bir.gov.ph/BIR/pdf/RR%207-2024.pdf | Primary |
| S3 | BIR, RR 11-2024 (13 Jun 2024), amended s.8 transitory text. Read from a copy downloaded from this URL earlier in this research session | https://bir-cdn.bir.gov.ph/BIR/pdf/RR%2011-2024.pdf | Primary |
| S4 | P&A Grant Thornton, amended transitory provisions | https://www.grantthornton.com.ph/technical-alerts/amended-transitory-provisions-on-invoicing-requirements-under-eopt-act/ | Secondary |
| S5 | P&A Grant Thornton, RMC 77-2024 clarifications | https://www.grantthornton.com.ph/insights/articles-and-updates1/tax-notes/clarification-on-the-invoicing-requirements-per-rr-no-7-2024-as-amended-by-rr-no-11-2024/ | Secondary |
| S6 | P&A Grant Thornton, EOPT preservation of books and registration (RR 7-2024 ss.4–5); KPMG EOPT effectivity | https://www.grantthornton.com.ph/insights/articles-and-updates1/tax-notes/eopt-is-here-updates-on-the-preservation-of-book-of-accounts-and-changes-in-taxpayer-registration/ ; https://kpmg.com/ph/en/insights/2024/01/special-intax-january-2024-issue-1-volume-3.html | Secondary |
| S7 | BIR, RR 26-2025 digest (e-invoicing deadline 31 Dec 2026) | https://bir-cdn.bir.gov.ph/BIR/pdf/RR%20No.%2026-2025%20Digest.pdf | Primary |
| S8 | BIR press release, 23 Sep 2026 (RMC 98-2026) | https://bir-cdn.bir.gov.ph/BIR/pdf/PR101SEP2326.pdf | Primary |
| S9 | P&A Grant Thornton, RMC 98-2026 e-invoicing guidelines | https://www.grantthornton.com.ph/technical-alerts/tax-alert/2026/bir-issues-guidelines-on-electronic-invoicing-retains-31-dec-2026-deadline/ | Secondary |
| S10 | NIRC (RA 8424), ss.203, 222 | https://lawphil.net/statutes/repacts/ra1997/ra_8424_1997.html | Primary |
| S11 | RR 2-2015 text (Lawyerly mirror): 2307/2316 scanned submission, 28 Feb | https://dev.lawyerly.ph/laws/67777 | Primary text, third-party host |
| S12 | P&A Grant Thornton, RR 16-2021 (scanned 2307/2316) | https://www.grantthornton.com.ph/insights/articles-and-updates1/tax-notes/amendments-on-manner-of-submission-of-copies-of-bir-forms-2307-2316/ | Secondary |
| S13 | JuanTax, BIR filing deadlines; GT on quarterly VAT and optional 2550M | https://help.juan.tax/en/articles/310-when-to-file-a-list-of-bir-tax-deadlines ; https://www.grantthornton.com.ph/insights/articles-and-updates1/tax-notes/optional-filing-and-payment-of-monthly-vat-returns-now-allowed | Secondary |
| S14 | PwC Tax Alert 21 (RMC 39-2026, eAFS attachments 2026) | https://www.pwc.com/ph/en/tax/tax-publications/tax-alerts/2026/tax-alert-21.html | Secondary (search excerpt only) |
| S16 | Revised Corporation Code (RA 11232), ss.73, 177 | https://lawphil.net/statutes/repacts/ra2019/ra_11232_2019.html | Primary |
| S17 | P&A Grant Thornton, SEC MC 9-2026 (AFS/GIS 2026) | https://www.grantthornton.com.ph/insights/articles-and-updates1/tax-notes/sec-mc-no-9-2026-guidelines-on-the-filing-of-afs-and-gis/ | Secondary |
| S18 | DTI BNRS FAQ: 5-year validity, renewal windows | https://bnrs.dti.gov.ph/faq | Primary |
| S19 | Local Government Code (RA 7160), ss.152(c), 161, 165–167 | https://lawphil.net/statutes/repacts/ra1991/ra_7160_1991.html | Primary |
| S20 | DOH IRR, ch. III "Food Establishments", PD 856 (SC E-Library) | https://elibrary.judiciary.gov.ph/thebookshelf/showdocs/10/39219 | Primary |
| S21 | FSIC: Angono LGU service page; Triple i Consulting | https://angono.gov.ph/bureau-fire-protection-issuance-fire-safety-inspection-certificate-fsic-business-permit/ ; https://www.tripleiconsulting.com/?p=1070 | Secondary (search excerpts) |
| S22 | IRR of RA 11199 (Social Security Act of 2018), Rules 32 and 39 | https://lawphil.net/statutes/repacts/ra2019/irr_11199_2019.html | Primary |
| S23 | DOLE-BWC Labor Inspection Checklist ("kept at workplace for at least 3 yrs") | https://bwc.dole.gov.ph/wp-content/uploads/2024/07/Labor_Inspection_Checklist.pdf | Primary (read via search excerpt; direct fetch blocked) |
| S24 | Respicio & Co., payroll records retention | https://www.respicio.ph/commentaries/how-long-must-employers-keep-payroll-records-under-philippine-labor-law | Secondary |
| S25 | Data Privacy Act (RA 10173), s.3(l) | https://lawphil.net/statutes/repacts/ra2012/ra_10173_2012.html | Primary |
| S26 | Philippine Accountancy Act (RA 9298), s.29 | https://lawphil.net/statutes/repacts/ra2004/ra_9298_2004.html | Primary |
| S27 | IAS Plus, new IAASB ISA 230 (5-year retention guidance) | https://www.iasplus.com/en/news/2005/October/news2422 | Secondary |
| S28 | PRC Resolution 27 s.2024 (CPD, 3-year accreditation); jur.ph BOA firm registration | https://www.prc.gov.ph/sites/default/files/2024-27%20published%20cpa.pdf ; https://jur.ph/law/facts/governing-public-accountancy-registration-firms-partnerships | Primary (excerpt) / Secondary |
| S29 | AMLA (RA 9160), s.9(b) | https://lawphil.net/statutes/repacts/ra2001/ra_9160_2001.html | Primary |
| S30 | RA 10365 (AMLA amendments: covered persons incl. accountants) | https://lawphil.net/statutes/repacts/ra2013/ra_10365_2013.html | Primary |
| S31 | Civil Code (RA 386), art. 1144 | https://lawphil.net/statutes/repacts/ra1949/ra_386_1949.html | Primary |
| S33 | Tax-agent accreditation: *Maestro v. CIR* summary; RR 11-2006 amendments summary | https://batasnatin.com/laws/emelino-t-maestro-v-commissioner-of-internal-revenue-cta-case-no-11309-1st-1 ; https://jur.ph/law/facts/amending-revenue-regulations-tax-practitioners-accreditation | Secondary |

RA 8792 s.13 is cited through R01. IDs S15 and S32 are not used.
