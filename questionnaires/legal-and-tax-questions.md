# Questions for a Philippine Lawyer and Tax Adviser

**Purpose:** the brief for the one or two professional sessions agreed in D-027. The answers decide what the Documents app may claim, which documents must stay on paper, and what our contracts must say.

**From:** the project owner. **To:** a Philippine lawyer (part A) and a tax adviser (part B). **How the answers are used:** they update Sections 03, 10 and 11 in this planning repository, and the app's claims and default settings.

## Context

We sell **Officetul**, a cloud ERP for Philippine SMEs (law, accounting, construction and others). We're adding a **Documents app**: clients scan or upload documents, which are kept unchanged with a SHA-256 fingerprint, made searchable with OCR, linked to ERP records, and access-logged. We **never advise clients to destroy paper originals**. Our research (cited, but not legal advice) is in Sections 10 and 11.

## How to answer

Please mark each answer **confirmed**, **likely** or **unsure**. Short answers are fine.

## A. Lawyer: electronic records and authenticity

### A1. Is a PDF scan of a paper document a "duplicate" admissible like the original under Rule 130 s.4 (2019 Rules), or do *NPC v. Codilla* and *MCC v. Ssangyong* still make it secondary evidence?

>

### A2. Can RA 8792 s.13 (electronic retention) satisfy a legal duty to keep **paper-origin** documents, or only born-electronic ones?

>

### A3. May a private company issue a document titled "Certified True Copy" of its own records? What wording avoids falsification risk (RPC art. 171(7), 172)? Is our proposed "Copy issued from [Company] records by [officer]: not a notarial or government certification" safe?

>

### A4. Please review our claims list (Section 10 §10). Are "tamper-evident for changes after upload" and "designed to help you authenticate records under the Rules on Electronic Evidence" safe?

>

### A5. Is an in-app approval click with MFA and an activity log enough to be an electronic signature under RA 8792 s.8? What is missing?

>

### A6. Do the Rules on Electronic Notarization (A.M. No. 24-10-14-SC) say anything about e-notarized PDFs stored in a third-party system, or their printouts?

>

## A. Lawyer: data privacy

### A7. Is Officetul a personal information processor for all client documents, including those arriving by email before anyone opens them?

>

### A8. Do click-through terms plus a data processing addendum satisfy IRR s.44 (per Advisory Opinion 2020-018)? May we use a general sub-processor authorisation with notice and objection?

>

### A9. Must the company register with the NPC as a processor regardless of headcount (Circular 2022-04 s.5(B))? Which systems must it list? Must a processor file the annual security incident report?

>

### A10. Is 24 hours a defensible breach-notice period from us to the client, given the 72-hour clock can start on our knowledge (IRR s.38(a))?

>

### A11. Is "never auto-delete during the subscription" acceptable before clients have a retention tool? At contract end, how long may backups keep their data?

>

### A12. Is there any reason (DPA, BSP, DICT, client contracts) to prefer hosting in the Philippines over Singapore?

>

## A. Lawyer: industry

### A13. Law firms: does storing client files with a cloud provider "reveal" confidences under CPRA s.27–28? Is client consent needed? Is there a minimum period for keeping client files, and may a firm keep copies after turning files over (s.56)?

>

### A14. Are accounting firms (bookkeeping and tax only) or law firms (company formation) AMLA covered persons with 5-year KYC record duties?

>

### A15. Retention periods for DOLE employment records, PhilHealth and Pag-IBIG employer records.

>

### A16. Do LGUs accept a digital construction logbook or an e-signed Certificate of Completion, or only signed and sealed paper?

>

### A17. The privacy rules set no period for keeping logs (NPC Circular 2023-06 s.29). Is **2 years** right for our security log (logins, refused access, staff access)? May we keep a short "exit record" (company name, dates, export fingerprint, deletion certificate) after deleting a client's data?

>

### A18. We plan to keep **daily fingerprints (hashes) of each client's activity log for 10 years** in storage nobody can delete. Are such hashes personal data, and is the 10-year period defensible?

>

### A19. Does a fingerprint kept with an independent party (an AWS locked bucket, a foreign time-stamp service) help show integrity under the Rules on Electronic Evidence, Rule 7 s.2?

>

### A20. We group repeated views of the same document by one person into one log entry per 10 minutes, but log every download. Is that acceptable for law-firm clients?

>

### A21. What is the "specified period required by the NPC" for keeping incident logs (2023-06 s.29)?

>

### A22. Should our support-access rules (client approval, 4-hour limit, emergency "break-glass" access with notice within 24 hours) be written into the data processing addendum? Does emergency access without prior approval need the client's advance consent in the contract?

>

## B. Tax adviser

### B1. Can scanned images replace paper **purchase invoices** and **BIR 2307s** for the 5-year retention period, or must the paper be kept? (RR 7-2024; RR 2-2015 mentions hard copies of withholding certificates.)

>

### B2. Does RR 5-2014's electronic-storage standard (imaging, indexing, integrity) still apply under RR 7-2024?

>

### B3. Do e-invoices under RMC 98-2026 carry their own storage format or retention duty for the issuer and the buyer?

>

### B4. What default "review after" period should the app suggest for tax records: 5 years, or 10 given NIRC s.222(a)?

>

### B5. Is "helps organise records you must keep for the BIR" a safe description? Anything we should never say?

>

## Anything else?

Anything we didn't ask that a document app for Philippine SMEs must get right?

>
