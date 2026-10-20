# R01 — Philippine SME market and demand drivers for document management

- **Status:** First pass, complete for Section 01. Research only, not legal advice.
- **Researched:** 2026-10-18. All sources accessed 2026-10-18.
- **Question:** How big is the Philippine SME market, how digital is it, and which regulations create document-management demand in each candidate segment (accounting/bookkeeping, law firms, construction)?
- **Scope:** This note frames the opportunity. It does **not** conclude that a DMS should be built.

**Labels used** (from the [README](../README.md#statement-labels)): **Verified fact**, **Estimate**, **Working assumption**, **Open question**, **Recommendation**. Source IDs `[S#]` point to the list at the end.

**Access caveat:** dti.gov.ph, psa.gov.ph, privacy.gov.ph and sc.judiciary.gov.ph blocked automated fetching (Cloudflare or 403 errors). For those sources, figures come from the search engine's excerpt of the official page and are marked *(excerpt)*. Re-check them in a browser before quoting them to the partner. Statute text from lawphil.net and BIR PDFs from bir-cdn.bir.gov.ph was read directly.

---

## 1. MSME landscape

### 1.1 Official size definitions

| Basis | Micro | Small | Medium | Large | Label / source |
|---|---|---|---|---|---|
| Total assets, excluding land (Magna Carta for MSMEs, RA 9501 s.3) | ≤ ₱3M | ₱3,000,001–15M | ₱15,000,001–100M | above medium | **Verified fact** [S3][S4]. Lawphil's text prints the medium ceiling as "P1000,000,000", an apparent typo. DTI AO 09 s.2008 gives ₱100M. |
| Employment (PSA statistical practice) | <10 | 10–99 | 100–199 | 200+ | **Verified fact** *(excerpt)* [S55] |
| BIR taxpayer class by gross sales (RR 8-2024, under EOPT) | <₱3M | ₱3M–<₱20M | ₱20M–<₱1B | ≥₱1B | **Verified fact** [S21] |

- The DTI/PSA counts below use **employment** size. The BIR class drives tax duties: for example, micro taxpayers are exempt from mandatory e-invoicing. A firm can be "small" under one scheme and "micro" under another. **Verified fact** [S3][S21]
- DTI has said it is looking at revising the MSME definition, and a 2023 Senate bill proposed new asset thresholds. No enacted amendment was found. **Open question** [S55]

### 1.2 Counts and employment (PSA 2023 List of Establishments, as published by DTI)

| Size | Enterprises | % of total | Employment share | Label |
|---|---|---|---|---|
| Micro | 1,127,058 | 90.43% | 33.95% | **Verified fact** *(excerpt)* [S1] |
| Small | 109,912 | 8.82% | 26.26% | **Verified fact** *(excerpt)* [S1] |
| Medium | 4,763 | 0.38% | 6.77% | **Verified fact** *(excerpt)* [S1] |
| **All MSMEs** | **1,241,733** | **99.63%** | **66.97% (6,351,466 jobs)** | **Verified fact** *(excerpt)* [S1] |
| Large | 4,640 | 0.37% | 33.03% (3,132,499 jobs) | **Verified fact** *(excerpt)* [S1][S53] |
| Total | 1,246,373 | 100% | — | **Verified fact** *(excerpt)* [S1] |

- **Caveats.** The employment shares cover employment *in establishments on the List of Establishments*, not total national employment. The List also undercounts informal businesses. 2023 is the latest year found; no 2024 DTI release turned up. **Verified fact / Open question** [S1][S2]
- The top five MSME sectors are wholesale/retail (604,313), accommodation and food (190,899), manufacturing (140,180), other services (87,197), and financial/insurance (52,924). Together they make up 86.61% of MSMEs. **Verified fact** *(excerpt)* [S1]

### 1.3 Candidate-segment counts

| Segment | Best figure found | Label |
|---|---|---|
| Professional, scientific & technical activities (PSIC Section M), MSMEs | About **19,900** (1.60% of MSMEs) | **Estimate**: the share comes from a Statista chart of DTI data [S8, secondary] applied to [S1]. Low confidence. |
| Construction MSMEs | About **5,100** (0.41% of MSMEs) | **Estimate**: same method and same weakness [S8] |
| Legal activities (PSIC 691), formal sector | **970** establishments (18.9% of Section M); Section M total 5,145 (2022 ASPBI, preliminary). NCR holds 45.1% of Section M. | **Verified fact** *(excerpt)* [S5] |
| Architectural & engineering activities (PSIC 711), formal sector | 579 establishments (2022 ASPBI, preliminary) | **Verified fact** *(excerpt)* [S5] |
| Accounting, bookkeeping, auditing; tax consultancy (PSIC 692) | No establishment count found. 23,678 workers (16.9% of Section M) in 2022. | **Verified fact** for workers *(excerpt)* [S5]; count is an **Open question** |
| Construction, formal sector | **2,512** establishments (2024 ASPBI, final); NCR 935 (37.2%) | **Verified fact** *(excerpt)* [S6] |

- **Why ASPBI counts are small.** ASPBI's "formal sector" includes corporations, partnerships, cooperatives and foundations of any size. It includes single proprietorships only if they have 10 or more workers or have branches [S7]. Sole-practitioner law offices, freelance bookkeepers and small contractors are therefore largely **excluded**. **Verified fact** *(excerpt)*
- Counting the people who practise these professions needs PRC/IBP rosters, the PCAB licence list and the BIR accredited-agent list. None was pulled in this pass. **Open question**

---

## 2. Digitalisation signals

| Indicator | Value | Date / method | Label |
|---|---|---|---|
| Establishments that owned and used computers | 94.9% (268,314 of 282,865) | PSA SICT 2019, run as an ASPBI rider, so it covers the **formal** frame only | **Verified fact** *(excerpt)* [S9] |
| Establishments with internet access | 81.0% (227,623) | PSA SICT 2021, final | **Verified fact** *(excerpt)* [S10] |
| MSMEs using "advanced digital tools" / no digital tools | 6% / 23% | DTI e-Commerce Office baseline survey, Sept 2020. Sample and method not retrieved (site unreachable). | **Verified fact** *(excerpt)*; dated [S11] |
| Small businesses receiving at least half of sales in cash | 77%, the highest of the markets surveyed | CPA Australia Asia-Pacific Small Business Survey, 2025 results. PH sample size not stated. | **Verified fact** (secondary, tier 3–4) [S12] |
| Small businesses saying technology investment improved profitability | 68% (survey average 56%) | Same survey | **Verified fact** (secondary) [S12] |
| World Bank Enterprise Survey, Philippines | 1,002 firms (5+ employees), April 2023–February 2024 | The country profile reviewed did not surface cloud or document-management indicators | **Verified fact** [S13] |

- **Cloud adoption:** no official or credible figure was found for Philippine SME cloud use or for paper-based operations. Vendor and telco claims exist but do not name their surveys. **Open question**
- PSA SICT releases after 2021 were not found. **Open question**
- **Working assumption:** most micro and small firms have a computer and internet but run on paper, cash, email attachments and consumer file-sharing tools. This fits the data above but is not measured by it. Interviews must test it.

---

## 3. Regulatory drivers by segment

### 3a. Accounting/bookkeeping, and all BIR-registered businesses

| Driver | What it requires | Status | Label |
|---|---|---|---|
| **Retention period**: NIRC s.235 as amended by **RA 11976 (EOPT)** s.33 | Books of accounts, subsidiary books and "other accounting records" must be kept **5 years**, counted from the day after the filing deadline (or from the actual filing date, if late) for the year of the last entry | Act approved 5 Jan 2024; effective 15 days after publication | **Verified fact** [S14] |
| Previous rule (RR 17-2013) | **10 years**; the auditing CPA also kept audited FS for 10 years; records had to be kept until any pending protest or refund case was resolved | Replaced by the EOPT 5-year rule | **Verified fact** [S15] |
| RR 7-2024 (EOPT implementing rules) | Manual and loose-leaf books kept in **hard copy**; computerized books kept **electronically**; records kept past 5 years if material to a pending protest or refund; auditing CPA keeps electronic copies of FS and working papers for 5 years | Issued 11 Apr 2024 | **Verified fact** that RR 7-2024 covers preservation [S16]. The details come from Grant Thornton (secondary) [S17] |
| "Other accounting records" | Includes invoices, receipts, vouchers, returns and source documents (RR 5-2014 definition) | — | Secondary [S17] |
| CAS / computerized books (CBA) | EOPT changes to a CAS or CBA count as a **major enhancement**: the taxpayer must update its system registration and get a new Acknowledgement Certificate, which replaced the Permit to Use | Reconfiguration due 31 Dec 2024, extendable up to 6 months | **Verified fact** [S18] |
| Registering books via ORUS | Loose-leaf books registered within 15 days after year-end; computerized books within 30 days. RMC 06-2025 extended the 2025 deadlines because of ORUS login problems. | — | Secondary (KPMG) [S22] |
| **E-invoicing** (RR 11-2025 as amended by **RR 26-2025**; **RMC 98-2026**) | By **31 Dec 2026**, these must issue e-invoices: e-commerce taxpayers classed small, medium or large; LTS and EOPT large taxpayers; **CAS/CBA users with invoicing software**. **Micro taxpayers are exempt.** Exporters, incentivised RBEs and POS users follow later. Electronic sales reporting is a separate, later requirement. | RMC 98-2026 issued 22 Sep 2026, effective immediately. The rules on e-invoicing service providers were still pending as of 23 Sep 2026. | **Verified fact** [S19][S20]. Deloitte reports that covered taxpayers were "unable to move forward pending clear guidance" [S23] |
| Corporate records (Revised Corporation Code s.73) | Keep and "carefully preserve" all corporate information, including "a record of all business transactions", minutes and resolutions. Records are open to inspection "regardless of the form in which they are stored". | No retention period found in the part read | **Verified fact** [S25] |
| Electronic retention (**E-Commerce Act, RA 8792 s.13**) | A legal duty to keep originals is met by an electronic copy that stays accessible, accurately represents the original, and identifies its originator, addressee and date/time. A **third-party service** may be used. | In force | **Verified fact** [S24] |
| Civil Code art. 1144 | Actions on a **written contract** prescribe in **10 years** | In force | **Verified fact** [S26] |

- **Working assumption:** the EOPT cut from 10 to 5 years *reduces* the tax reason to keep records. Contract and litigation exposure (10 years under art. 1144) still pushes firms to keep records longer.
- **Open question (for a lawyer):** whether *scanned* manual or loose-leaf books satisfy BIR, given RR 7-2024's hard-copy rule and RA 8792 s.13. Both sides of this tension are documented; the answer is not.
- **Working assumption:** e-invoicing at CAS users will create a new kind of record to retain (structured e-invoices). Whether customers expect a DMS or their invoicing system to hold these is untested.

### 3b. Law firms

| Driver | What it requires | Status | Label |
|---|---|---|---|
| SC e-filing: A.M. No. 25-05-16-SC, then **A.M. No. 25-09-16-SC** (2025 Transitory Rules) | Lawyers file in the Supreme Court through **eCourt PH** on the Philippine Judiciary Platform, in PDF/A, at most 100 MB per file, one file per pleading. Paper only in exceptional cases. | Hybrid filing from 1 Jul 2025; mandatory for covered pleadings from 1 Oct 2025; transitory rules effective 30 Oct 2025 | Secondary (law-firm summary, news) [S28][S29] |
| E-filing in lower courts | Reports of email PDF filing in first- and second-level courts, but nothing current or authoritative was found | — | **Open question** |
| **2004 Rules on Notarial Practice** (A.M. No. 02-8-13-SC) | Permanently bound notarial register; every month, certified copies of entries and duplicate originals go to the Clerk of Court | In force | Secondary [S30][S31] |
| **2025 amendment** to the notarial rules | Duplicate originals **emailed as PDFs**, one file per instrument, as direct attachments (no cloud links, no ZIP). Notaries may not charge for digitizing. Signatories receive an electronic copy within 5 days. The notary keeps an original of each contract. | En banc resolution 4 Mar 2025; reported effective **21 Jun 2025** | Secondary (news, law firm) [S30][S31] |
| **Rules on Electronic Notarization**, A.M. No. 24-10-14-SC | Separate commission for electronic notaries; only **SC-accredited** electronic notarization facilities may be used; records go to an SC central notarial database. Paper documents stay under the 2004 Rules. | Promulgated 4 Feb 2025. Applications opened 10 Aug 2026. The SC eNotarization service goes live **19 Oct 2026** (the day after this note). | Secondary (news, law firm) [S32][S33] |
| 2020 Interim Rules on Remote Notarization (A.M. No. 20-07-04-SC) | Allowed only where a party was under COVID community quarantine | Effective 16 Aug 2020; current status not confirmed | Secondary [S34]; **Open question** |
| **Rules on Electronic Evidence** (A.M. No. 01-7-01-SC) | E-documents are the functional equivalent of paper (R3 s.1). Authentication by digital signature, approved security procedure, or other proof of integrity (R5 s.2). Evidentiary weight depends on how reliably the document was generated and **stored** and on the **integrity of the system** (R7 s.1). Electronic business records kept in the regular course are a hearsay exception (R8). | Effective 1 Aug 2001 | **Verified fact** [S27] |
| **CPRA** (A.M. No. 22-09-01-SC) | Canon III: confidentiality, ss. 27–32; inventory and accounting of client funds and property, ss. 49–50; turnover on termination, s. 56 | Adopted 11 Apr 2023; reported effective 30 May 2023 | Section mapping is secondary [S36][S37]; official text blocked [S35] |

- **Open question:** whether the CPRA sets a minimum period for keeping client files. Secondary summaries show none. Confirm against the official text and with a Philippine lawyer.
- **Working assumption:** client matter files hold **sensitive personal information** under the DPA (offences, court proceedings, government IDs, tax returns; s.3(l)) [S44]. This makes law-firm documents a high-risk data category (see 3d).

### 3c. Construction and project-based firms

| Driver | What it requires | Status | Label |
|---|---|---|---|
| **RA 12009** New Government Procurement Act | PhilGEPS is the single portal, with e-bidding and e-signatures under RA 8792 (s.20). Audit trail of e-transactions (s.22). Bid-conference video kept **at least 5 years** (s.38). Eligibility documents may be submitted electronically (s.52). PhilGEPS links to PCAB licensing (s.23). | Approved 20 Jul 2024; effective 13 Aug 2024; IRR (GPPB Res. 02-2025) effective 25 Feb 2025 | **Verified fact** [S38][S39] (IRR dates *(excerpt)*) |
| General retention of procurement records under RA 12009 | None found in the portion of the Act read (about 74%) | — | **Open question** |
| **Civil Code art. 1723** | Engineer/architect liable if the structure collapses within **15 years** of completion because of defective plans or ground; contractor liable for defective construction or materials. Action within 10 years of the collapse. | In force | **Verified fact** [S26] |
| National Building Code (PD 1096) | Signed and sealed construction logbook and Certificate of Completion to the Building Official; Certificate of Occupancy before use | In force | Secondary [S41] |
| PCAB licence renewal | Latest audited FS and ITR; renewal schedule set by licence number; ₱5,000 fee for late filing | DTI notice dated 2018; current list not verified | **Verified fact** but dated [S40] |
| OSH Law (RA 11058) | A Construction Safety & Health Program for every project (DO 13-98). DO 198-18 reportedly replaced by DO 252-25, effective 16 May 2025. | — | Secondary, tier 5 [S42][S43]; record-retention periods are an **Open question** |

- **Working assumption:** the strongest construction need is **revision control** of drawings, specifications and submittals, plus long-tail retention (art. 1723 implies 15–25 years). Short statutory retention is not the main driver. Validate through interviews.

### 3d. Data Privacy Act (all segments)

| Driver | Requirement | Label |
|---|---|---|
| RA 10173 s.11(e) | Keep personal data only as long as needed for its purpose, for legal claims, or as the law provides. This creates **disposal** obligations as well as retention ones. | **Verified fact** [S44] |
| RA 10173 s.20 | "Reasonable and appropriate" organisational, physical and technical measures, including against **accidental loss or destruction** (natural dangers). The level is scaled to organisation size and cost. | **Verified fact** [S44] |
| NPC Circular 16-03 | Notify NPC and data subjects within **72 hours** of knowing of a breach; full report within 5 days. Delay is not allowed if at least 100 data subjects are affected. | **Verified fact** *(excerpt)* [S45] |
| NPC Circular 2023-06 (security of personal data) | Covers the private sector. Need-to-know access; **MFA** for online access to sensitive data; encrypted removable media; business continuity plan covering backup and restore. Effective 30 Mar 2024; repeals Circular 16-01. | **Verified fact** *(excerpt)* [S46] |
| NPC Circular 2022-04 (registration) | Register all data processing systems if the organisation has **≥250 employees**, **or** processes SPI of **≥1,000 individuals**, **or** its processing is likely to pose a risk. Automated decision-making requires registration in all cases. | **Verified fact** *(excerpt)* [S47]. Sources give the effective date as both Jan 2022 and Jan 2023: **Open question** |
| NPC enforcement scale | **11,524** registered entities in 2025, up from 4,307 in 2023; **942** breach notifications and **3,538** complaints handled since 2022 | **Verified fact** *(excerpt)* [S48] |

- **Estimate:** a law or accounting firm with 1,000 or more individual clients probably crosses the SPI registration threshold, even when it is micro by headcount (TINs, tax returns and case details count as SPI) [S44][S47]. Confirm per firm.
- No NPC breach statistics broken down by organisation size were found. **Open question**

---

## 4. Evidence of pain

| Evidence | Type | Label |
|---|---|---|
| Typhoon Ondoy (2009): Malabon–Navotas prosecutor's case records "100% lost" | News, government office | Anecdotal [S49] |
| Typhoon Yolanda (2013): about 95% of a Tacloban court's case files under water and mud | News, government | Anecdotal [S50] |
| Typhoon Kristine damaged BIR accountable forms at a Bicol RDO (RMC 18-2026) | Regulator notice about the BIR's own forms, not taxpayers' | Anecdotal (secondary copy) [S51] |
| BIR examination rules: books may be inspected at the taxpayer's place of business or at the BIR, generally once per taxable year, with exceptions | Regulation | Secondary [S17] |
| 942 breach notifications handled by NPC since 2022 (all sizes) | Regulator | **Verified fact** *(excerpt)* [S48] |

- **No credible source** was found that measures document loss, retrieval time, or audit-preparation burden at Philippine **private SMEs**. The evidence above concerns government offices and is anecdotal. **Open question**

## 5. IT budgets and software spend

- **No credible data found** on IT budgets or software spending by Philippine SMEs. GlobalData reports enterprise-wide ICT growth (10.4% CAGR forecast for 2024–29). It is paywalled, not split by firm size, and from a secondary source [S52]. **Open question**
- **Working assumption** (from the project context): budgets are limited. The 77% cash-heavy sales figure [S12] is consistent with low spending on SaaS, but it does not measure it.

---

## Key implications for Section 01

- **Working assumption:** the addressable market for document management is far smaller than "1.24M MSMEs". The formal-sector cores of the three segments total thousands of establishments (for example 970 legal, 2,512 construction), plus an uncounted long tail of sole practitioners.
- **Recommendation:** frame the opportunity around **dated, external triggers**, not generic "digitalisation". The triggers are BIR e-invoicing for CAS users (31 Dec 2026), SC mandatory e-filing (since Oct 2025), the 2025 notarial PDF transmittal rule, and eNotarization go-live (19 Oct 2026).
- **Recommendation:** treat the law-firm segment as the one with the clearest *document-specific* regulatory change in 2025–26: e-filing, notarial PDFs, e-notarization and SPI-heavy files. Verify the details against official SC texts before relying on them.
- **Working assumption:** for accounting firms, the EOPT 5-year rule *reduces* tax-driven retention. Demand is more likely to come from client-document collection, e-invoice records and audit readiness than from retention length. Untested.
- **Working assumption:** for construction, the value is likely revision control and long liability tails (art. 1723). Retention law is weak and fragmented.
- **Recommendation:** any product claim must stop at "supports" compliance. RA 8792 s.13 and Electronic Evidence R7 make storage integrity and audit trails *relevant*, but no law found makes a DMS sufficient for compliance (consistent with PROJECT-CONTEXT "must not assume").
- **Recommendation:** treat the DPA (s.20 security, 72-hour breach notice, 2023-06 MFA and backup, s.11(e) disposal) as a baseline requirement for any product that stores client files, whichever segment is chosen.
- **Working assumption:** there is no evidence base on willingness to pay. Pricing must come from interviews and the competitor research (R02).

## Evidence gaps that only customer interviews can fill

1. How documents are stored today: paper, shared drives, Google Drive or Dropbox, email, Viber or Messenger. The share of each.
2. Actual incidents: lost files, flood or fire loss, BIR or client document requests that could not be met, version mix-ups. Frequency and cost.
3. Who owns records management in a 5–50-person firm, and what time it takes each week.
4. Whether firms keep records longer than the legal minimum, and why (liability, habit, client demands).
5. How law firms are handling eCourt PH and the notarial PDF transmittals now: tools, pain, workarounds.
6. Whether CAS users expect their e-invoicing or accounting system, or a separate DMS, to hold e-invoices and supporting documents.
7. For construction: how drawings and revisions are controlled, who approves them, and what owners and government clients demand at turnover.
8. Current software spend, the budget holder, and the acceptable monthly price per user or per firm.
9. Data-residency and privacy expectations, including whether clients ask about NPC registration and security.
10. Willingness to switch from free tools, and the trigger that would make a firm switch.

## Sources

All accessed **2026-10-18**. Tier: 1 = government, regulator or court; 3 = credible research; 4 = industry analysis; 5 = practitioner or community.

| ID | Title | URL | Tier / note |
|---|---|---|---|
| S1 | DTI, 2023 Philippine MSME Statistics | https://dti.gov.ph/businesses/msmes/msmeresources/msme-statistics | 1; excerpt only (fetch failed) |
| S2 | DTI, 2022 Philippine MSME Statistics | https://dti.gov.ph/resources/msme-statistics | 1; excerpt |
| S3 | RA 9501, Magna Carta for MSMEs (lawphil) | https://lawphil.net/statutes/repacts/ra2008/ra_9501_2008.html | 1 |
| S4 | DTI Administrative Order No. 09 s.2008 | https://issuances-library.senate.gov.ph/sites/default/files/2022-11/Department%20of%20Trade%20and%20Industry%20Administrative%20Order%20No.%2009%2C%20Series%20of%202008.pdf | 1; excerpt |
| S5 | PSA, 2022 ASPBI – Professional, Scientific and Technical Activities: Preliminary Results | https://psa.gov.ph/content/2022-annual-survey-philippine-business-and-industry-aspbi-professional-scientific-and | 1; excerpt |
| S6 | PSA, ASPBI – Construction Sector (2024 final; 2021/2022 results) | https://psa.gov.ph/statistics/construction/aspbi | 1; excerpt |
| S7 | PSA, 2024 ASPBI Technical Notes | https://psa.gov.ph/system/files/ssd/3.%202024%20ASPBI%20Technical%20Notes%20H_rev_16Jun2026_for%20upload_1.pdf | 1; excerpt |
| S8 | Statista, Business industries of the MSME sector, Philippines | https://www.statista.com/statistics/1016349/business-industries-msme-sector-philippines | 4; secondary, chart shares only |
| S9 | PSA, 2019 SICT – All Establishments: Final Results | https://psa.gov.ph/content/2019-survey-information-and-communication-technology-sict-all-establishments-final-results | 1; excerpt |
| S10 | PSA, 2021 SICT – All Establishments: Final Results | https://psa.gov.ph/content/2021-survey-information-and-communications-technology-sict-all-establishments-final-0 | 1; excerpt |
| S11 | DTI e-Commerce Office, MSME digitalization baseline survey | https://ecommerce.dti.gov.ph/madali/baseline_survey.html | 1; excerpt, site unreachable |
| S12 | CPA Australia, Strong growth and optimism among small businesses in the Philippines | https://www.cpaaustralia.com.au/about-cpa-australia/media/media-releases/strong-growth-optimism-philippines | 3–4 |
| S13 | World Bank Enterprise Surveys, Pilipinas 2023 Country Profile | https://espanol.enterprisesurveys.org/content/dam/enterprisesurveys/documents/country-profiles/Philippines-2023-TGL.pdf | 1–3 |
| S14 | RA 11976, Ease of Paying Taxes Act (lawphil) | https://lawphil.net/statutes/repacts/ra2024/ra_11976_2024.html | 1 |
| S15 | BIR, RR 17-2013 digest | https://bir-cdn.bir.gov.ph/BIR/pdf/75723rr13_17-revised.pdf | 1 |
| S16 | BIR, RR 7-2024 digest | https://bir-cdn.bir.gov.ph/BIR/pdf/RR%207-2024.pdf | 1 |
| S17 | Grant Thornton PH, EOPT is here: preservation of books of accounts | https://www.grantthornton.com.ph/insights/articles-and-updates1/tax-notes/eopt-is-here-updates-on-the-preservation-of-book-of-accounts-and-changes-in-taxpayer-registration/ | 4 |
| S18 | BIR, RR 11-2024 digest | https://bir-cdn.bir.gov.ph/BIR/pdf/RR%2011-2024.pdf | 1 |
| S19 | BIR, RR 26-2025 digest | https://bir-cdn.bir.gov.ph/BIR/pdf/RR%20No.%2026-2025%20Digest.pdf | 1 |
| S20 | BIR press release, "BIR sets electronic invoicing rules ahead of December 31 compliance deadline" (23 Sep 2026) | https://bir-cdn.bir.gov.ph/BIR/pdf/PR101SEP2326.pdf | 1 |
| S21 | BIR, RR 8-2024 (taxpayer classification) | https://bir-cdn.bir.gov.ph/BIR/pdf/RR%208-2024.pdf | 1; excerpt |
| S22 | KPMG PH, Special InTAX January 2025 | https://kpmg.com/ph/en/insights/2025/01/special-intax-january-2025-issue-1-volume-1.html | 4; excerpt |
| S23 | Deloitte SEA, Countdown to compliance: embracing e-invoicing | https://www.deloitte.com/southeast-asia/en/services/tax/perspectives/countdown-to-compliance-embracing-e-invoicing.html | 4 |
| S24 | RA 8792, Electronic Commerce Act (lawphil) | https://lawphil.net/statutes/repacts/ra2000/ra_8792_2000.html | 1 |
| S25 | RA 11232, Revised Corporation Code (lawphil) | https://lawphil.net/statutes/repacts/ra2019/ra_11232_2019.html | 1 |
| S26 | RA 386, Civil Code (lawphil) | https://lawphil.net/statutes/repacts/ra1949/ra_386_1949.html | 1 |
| S27 | A.M. No. 01-7-01-SC, Rules on Electronic Evidence (lawphil) | https://lawphil.net/courts/supreme/am/am_01-7-01_2001.html | 1 |
| S28 | Aureada Law, 2025 Transitory Rules on Electronic Filing and Service | https://www.aureadalaw.com/post/a-new-era-of-e-filing-supreme-court-issues-2025-transitory-rules-on-electronic-filing-and-service | 5 |
| S29 | Daily Tribune, SC goes full digital filing (20 Jun 2025) | https://tribune.net.ph/2025/06/20/sc-goes-full-digital-filing | 4 |
| S30 | Newsbytes.PH, SC amends notarial rules to require digital reporting (12 Jun 2025) | https://newsbytes.ph/2025/06/12/sc-amends-notarial-rules-to-require-digital-reporting/ | 4 |
| S31 | Alburo Law, New rules on notarial practice (OCA Circular 197-2025) | https://www.alburolaw.com/new-rules-on-notarial-practice-oca-circular-197-2025/ | 5 |
| S32 | Newsbytes.PH, SC to operationalize eNotarization service on Oct 19 (19 Aug 2026) | https://newsbytes.ph/2026/08/19/sc-to-operationalize-enotarization-service-on-oct-19/ | 4 |
| S33 | Cruz Marcelo, SC promulgates Rules on Electronic Notarization | https://cruzmarcelo.com/supreme-court-promulgates-rules-on-electronic-notarization/ | 5 |
| S34 | Cruz Marcelo, 2020 Interim Rules on Remote Notarization | https://cruzmarcelo.com/supreme-court-issues-2020-interim-rules-on-remote-notarization-of-paper-documents/ | 5 |
| S35 | Supreme Court, A.M. No. 22-09-01-SC (CPRA) | https://sc.judiciary.gov.ph/22-09-01-sc-code-of-professional-responsibility-and-accountability/ | 1; blocked, not read |
| S36 | Digest.ph, CPRA summary | https://www.digest.ph/laws/code-of-professional-responsibility-and-accountability | 5 |
| S37 | Batas Natin, CPRA Canon III s.28 and s.49 notes | https://batasnatin.com/laws/cpra-c3-section-28-protecting-client-confidences-five-exceptions | 5; excerpt |
| S38 | RA 12009, New Government Procurement Act (lawphil) | https://lawphil.net/statutes/repacts/ra2024/ra_12009_2024.html | 1; about 74% read |
| S39 | GPPB Resolution No. 02-2025 (IRR of RA 12009) | https://www.dbm.gov.ph/wp-content/uploads/Issuances/2025/GPPB-Resolution/IRR-RA-12009-Resolution-No-02-2025.pdf | 1; dates from excerpt |
| S40 | DTI, DTI urges contractors to register/renew PCAB licenses (2018) | https://www.dti.gov.ph/archives/regional-archives/region-6-news-regional-archives/dti-urges-contractors-register-renew-pcab-licenses/ | 1; dated |
| S41 | DPWH, Revised IRR of PD 1096 | https://www.dpwh.gov.ph/DPWH/files/nbc/IRR.pdf | 1; not read, content from secondary excerpts |
| S42 | Open Exam Prep, RA 11058, DO 198-18 & DO 252-25 | https://open-exam-prep.com/study-guides/ph-dole-cosh-so2/ch2/ra-11058-dole-198-18 | 5 |
| S43 | AEDO Construction, CSHP requirement Philippines | https://aedoconstruction.com/blog/construction-safety-health-program-cshp-requirement-philippines/ | 5 |
| S44 | RA 10173, Data Privacy Act (lawphil) | https://lawphil.net/statutes/repacts/ra2012/ra_10173_2012.html | 1 |
| S45 | NPC Circular 16-03, Personal Data Breach Management | https://privacy.gov.ph/wp-content/uploads/2022/01/sgd-npc-circular-16-03-personal-data-breach-management.pdf | 1; excerpt |
| S46 | NPC Circular 2023-06 and FAQ | https://privacy.gov.ph/wp-content/uploads/2024/12/v12-19-2024_FAQ-NPC-Circular-2023-06_NNJ_JDN.pdf | 1; excerpt |
| S47 | NPC Circular 2022-04 (Registration) | https://privacy.gov.ph/wp-content/uploads/2023/05/Circular-2022-04-1.pdf | 1; excerpt |
| S48 | NPC, PAW 2025: NPC champions global privacy in borderless digital era | https://privacy.gov.ph/paw-2025-npc-champions-global-privacy-in-borderless-digital-era/ | 1; excerpt |
| S49 | Philstar, Records at Malabon-Navotas fiscal's office "100% lost" (2009) | https://www.philstar.com/metro/2009/10/18/514803/records-malabon-navotas-fiscals-office-100-lost/amp/ | 4; anecdotal |
| S50 | Inquirer, DOJ faces tough task of rebuilding court records in Yolanda-hit areas | https://newsinfo.inquirer.net/531901/doj-faces-tough-task-of-rebuilding-court-records-in-yolanda-hit-areas/amp | 4; anecdotal |
| S51 | Batas Natin, RMC No. 18-2026, damaged accountable forms | https://batasnatin.com/laws/rmc-no-18-2026-damaged-various-copies-of-usedissued-accountable-forms | 5; secondary copy of a BIR RMC |
| S52 | GlobalData, Philippines enterprise ICT revenue forecast | https://www.globaldata.com/media/technology/it-services-to-drive-philippines-overall-enterprise-ict-revenue-at-10-4-cagr-through-2029-forecasts-globaldata/ | 4 |
| S53 | PSA CAR, MSMEs in the Cordillera, 2023 ULE | https://rssocar.psa.gov.ph/system/files/attachment-dir/CAR-SSR-2025-26_MSMEs_in_the_Cordillera_2023_ULE.pdf | 1; excerpt |
| S55 | eFOI, Classification of MSMEs (PSA employment bands); Senate document on MSME definitions | https://foi.gov.ph/requests/classification-of-msmles ; https://legacy.senate.gov.ph/lisdata/1704014256!.pdf | 1; excerpt |
