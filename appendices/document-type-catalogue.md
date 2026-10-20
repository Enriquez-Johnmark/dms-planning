# Document-Type Catalogue (proposed)

**Status:** agreed by the project owner 2026-10-19 (D-023 to D-026), in [Section 03](../sections/03-philippine-sme-workflows.md). Sources and legal detail: [R07](../research/R07-doc-types-general-sme-and-accounting.md) (general SME, accounting) and [R08](../research/R08-doc-types-law-and-construction.md) (law, construction).

Each preset is configuration data that clients can extend. It is not a separate feature. Field names are business-level; the data model comes in Phases 6 and 9.

## Fields on every document type

| Field | Why |
|---|---|
| Title, document date, document type | Basic findability |
| Linked Officetul records (many-to-many, labelled) | D-019 |
| Sensitivity: Public / Internal / Confidential / **SPI** (preset default per type) | Data Privacy Act; suggests a default Folder when filing, never grants or removes access (D-098) |
| **Paper original kept? Where? Custodian** | D-008: never advise destroying originals. Several rules require paper originals (R08 §3.5). |
| Expiry / valid-until date (types that expire) | Captured now; reminders in the next release |
| Event date (e.g. completion, matter closed, fiscal year end) | Future retention rules count from events, not upload dates (R08 §3.7) |
| Revision label (free text, e.g. "Rev C") and status (e.g. Draft, For Approval, For Construction, Superseded) | Separate from the system's "official version" marker (R08 §3.3) |
| Optional **notarial details** group: Doc No., Page No., Book No., Series of | On every notarized PH document; searchable (R08 §3.10) |

## MVP presets: every SME (22)

| # | Preset | Typical Officetul links | Key extra fields | Default sensitivity | Paper original | Expires |
|---|---|---|---|---|---|---|
| 1 | **Sales Invoice (issued)** | Sales invoice, customer | Invoice no., type, buyer TIN, gross, VAT, VAT class | Confidential | Keep per BIR (copy) | — |
| 2 | **Invoice (purchase)** | Expense/bill, supplier | Supplier TIN, invoice no., gross, VAT, e-invoice flag, converted-OR flag | Confidential | Open question (tax adviser) | — |
| 3 | **Payment Receipt (OR / CR / AR)**; searchable as "Official Receipt", "OR" | Payment, expense, sales invoice | Receipt kind, linked invoice no., "not valid for input tax" flag, legacy-OR flag | Confidential | Open question | — |
| 4 | **BIR 2307** | Customer or supplier, payment | Direction, payor/payee TIN, period, ATC, tax withheld | Confidential | Open question (RR 2-2015 mentions hard copies) | — |
| 5 | Tax Return / BIR Filing | Company | Form no. (pick-list), period, amount, filing ref. | Confidential | — | — |
| 6 | Tax Payment Proof | Tax return | Form, period, amount, channel, reference | Confidential | — | — |
| 7 | Financial Statements (AFS) | Company | FY end, audited, auditor, eAFS / eFAST refs | Confidential | — | — |
| 8 | Books of Accounts | Company | Book type, format (manual / loose-leaf / CAS), FY | Confidential | **Yes for manual and loose-leaf** (RR 7-2024) | — |
| 9 | BIR Notice / Assessment | Company | Notice type, received, period, amount, **response deadline** | Confidential | Yes | Deadline |
| 10 | Bank Statement | Bank account | Bank, last 4 digits, period | Confidential | — | — |
| 11 | BIR Certificate of Registration (2303) | Company, branch | TIN, branch code, RDO, tax types | Internal | **Yes (displayed)** | — |
| 12 | DTI Business Name Certificate | Company | BN no., scope, registration date | Internal | — | **5 years** |
| 13 | SEC Registration Documents | Company | Subtype, SEC no., date | Internal | — | — |
| 14 | SEC General Information Sheet | Company | Year, meeting date, filed date | Internal | — | Annual |
| 15 | **Permit / Licence** | Company, branch, project | Subtype (Mayor's/Business Permit, Barangay, FSIC, Sanitary, PCAB, professional licence…), issuer, no., issue date | Internal | Often displayed | **Yes** |
| 16 | Government Employer Records | Company, payroll period | Agency (SSS / PhilHealth / Pag-IBIG / DOLE), subtype, period, ref., amount | Confidential | — | — |
| 17 | **Employee Document** | Employee | Subtype (contract, ID, clearance, medical, 201 item, payslip, 2316) | **SPI** | — | IDs, clearances |
| 18 | **Contract / Agreement** | Customer, supplier, project, employee | Parties, type, start, end, value, notarized flag | Confidential | **Yes (signed original)** | End date |
| 19 | Notarized Document (other) | Any | Notarial details group, instrument type | Confidential | Yes | — |
| 20 | Correspondence | Any | From, to, date received/sent, channel | Internal | — | — |
| 21 | Bond / Guarantee / Insurance Policy | Project, supplier, company | Issuer, policy/bond no., amount, beneficiary | Confidential | Yes | **Yes** |
| 22 | Supporting Document (PO, DR, SOA, quotation, photo, other) | Any | Subtype, reference no. | Internal | — | — |

## Optional pack: accounting and bookkeeping firms (8, configuration only)

Engagement Letter · Client Submission (period, category, completeness) · Audit Working Paper · Management Representation Letter · Auditor's Report · Filing Confirmation · Client Authorization / SPA · Professional Accreditation (expires).

## Later packs (need new features, not only presets)

| Pack | Needs | Main document types |
|---|---|---|
| **Law** | A Matter record, court deadlines, privilege and ethical walls, PDF/A e-filing export, notarial register and monthly transmittal, client-file turnover | Pleading, Court Order/Decision, Evidence/Annex, Affidavit, Notarial Register entry |
| **Construction** | Drawing register with revision chain, transmittals, RFI/submittal logs, change orders with written approval, daily log and photos, closeout checklist | Drawing, Specification, RFI, Submittal, Change Order, Progress Billing / SWA, Logbook, As-Built, Certificate of Completion |
| **Government contractor** (sub-pack) | PhilGEPS bid checklist, NOA–NTP timeline, warranty and security tracking | Bid documents, NOA, NTP, Performance Security |
