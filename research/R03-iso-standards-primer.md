# R03: ISO standards primer for a Philippine SME DMS

> **Status note (2026-10-19):** **ISO certification is a future goal; the senior partner decides which standard and when (D-028). It does not block this project. The Documents app is built ISO-ready in the meantime (D-011).** The findings below are unchanged research as of 2026-10-18. Where this note says "the partner's firm", read "the company behind Officetul".

**Purpose:** Background for the conversation with the senior partner who "is interested in ISO certification". It sets out what each relevant standard is, what certification means, the situation in the Philippines, and what a DMS can and cannot do.
**Researched:** 2026-10-18. Every source was accessed on 2026-10-18 unless noted otherwise. Citations such as [S1] point to the source list at the end.
**Labels:** **Verified fact** means a primary or authoritative source confirms it. **Estimate** is an inference or an indicative figure. **Open question** means the source could not be found or the sources disagree.

> **Method note (read first).** www.iso.org returned HTTP 403 (bot protection) to automated access. Edition, date, stage and replacement data therefore come mainly from the **ISO Open Data "deliverables metadata" dataset** [S1]. It is published by ISO under the ODC-By 1.0 licence, updated daily, and the file used here was last modified on 2026-10-14. The figures were cross-checked against the same catalogue pages served from ISO's `committee.iso.org` mirror. The catalogue URLs are cited in their canonical `www.iso.org/standard/<id>.html` form. The iso.org footer states that ISO content may not be used to prompt AI tools, except for content on the Open Data page. For that reason no ISO text is reproduced here, and scope is paraphrased in a few words only. If this document is shared outside the team, a human should re-check the catalogue pages in a browser.

---

## 0. TL;DR for busy readers

- **ISO 9001 has a new edition.** **ISO 9001:2026 (edition 6) was published on 2026-09-16** and **ISO 9001:2015 is withdrawn**. Accredited certificates to the 2015 edition stay valid until **30 Sept 2029**. **New initial certifications must be to the 2026 edition from 31 March 2028** at the latest. [S1][S2][S3][S17] — **Verified fact**
- **ISO/IEC 27001:2022 plus Amd 1:2024 (the climate amendment) is current.** No newer edition is registered in ISO's data. [S1][S4][S5] — **Verified fact**
- **The IAF no longer exists.** It **ceased operations on 1 January 2026**. Its successor is the **Global Accreditation Cooperation Inc. (Global ACI)**, and the **"IAF MLA" is now the "Global ACI MRA"**. [S22][S18][S19] — **Verified fact**
- **PAB has no ISO 27001 accreditations listed.** The Philippine Accreditation Bureau (PAB, under DTI) is the national accreditation body. Its public directory lists **22 accredited management-system certification scopes, covering QMS, EMS, OH&S, FSMS, HACCP and EOMS, and none for ISMS (ISO/IEC 27001)**. ISO 27001 certificates issued in the Philippines must therefore come from certification bodies accredited by foreign accreditation bodies. [S25] — **Verified fact** for the directory contents; the inference is an **Estimate**.
- **Software is never "ISO 27001 certified".** Organizations are certified. A DMS can supply evidence for parts of clause 7.5 ("documented information") and some Annex A controls. It cannot supply leadership, risk assessment, internal audit, management review or the certification audit itself.
- **ISO 30301 (records management system) is about to be replaced.** A 3rd edition is at the approval stage (ISO/PRF 30301, stage 50.00), and 30301 is **not** in the Global ACI MRA scope. [S1][S7][S20] — **Verified fact**

---

## 1. Standards: edition, status, certifiability

ISO stage codes used below [S2][S7]: 60.60 = published; 90.93 = confirmed at systematic review; 90.92 = to be revised; 90.60 = close of review; 50.00 = final text registered for approval; 40.60 = close of DIS voting; 95.99 = withdrawn.

| Standard | Current edition (ISO ref) | Pub. date | Amendments | Status / stage | Type | Label | Source |
|---|---|---|---|---|---|---|---|
| **ISO 9001** Quality management systems — Requirements | **ISO 9001:2026**, edition 6 | 2026-09-16 | none yet | Published (60.60). Replaces ISO 9001:2015 and 9001:2015/Amd 1:2024, which are both withdrawn (95.99). | **Certifiable MSS (requirements)** | Verified fact | [S1][S2][S3] |
| **ISO/IEC 27001** ISMS — Requirements | **ISO/IEC 27001:2022**, edition 3 | 2022-10-25 | **Amd 1:2024 "Climate action changes"** (pub. 2024-02-23) | Published (60.60). No successor project appears in ISO data. | **Certifiable MSS (requirements)** | Verified fact | [S1][S4][S5] |
| **ISO 30301** Management systems for records — Requirements | **ISO 30301:2019**, edition 2 | 2019-01-31 (catalogue shows 2019-02) | Amd 1:2024 climate action changes (pub. 2024-02-23) | Published but **to be revised (90.92)**. **ISO/PRF 30301 (edition 3) is at 50.00**, registered 2026-09-07; the DIS was approved 2026-06-16. | **Requirements MSS.** Its abstract lists self-declaration, external confirmation, or certification by an external party as ways to demonstrate conformity. | Verified fact | [S1][S6][S7] |
| **ISO 15489-1** Records management — Concepts and principles | **ISO 15489-1:2016**, edition 2 | 2016-04-07 | none | Published, **confirmed (90.93)** | Guidance (concepts and principles); not certifiable | Verified fact | [S1][S8] |
| **ISO 16175-1** Functional requirements for software managing records, Part 1 | **ISO 16175-1:2020**, edition 2 | 2020-07-03 | none | Published; **systematic review closed (90.60)**, outcome not yet shown | Technical: model functional requirements for software. **No accredited product-certification scheme was found** (see Open questions). | Verified fact (status); Open question (scheme) | [S1][S9] |
| **ISO/TS 16175-2** Part 2: guidance for selecting, designing, implementing and maintaining records software | **ISO/TS 16175-2:2020**, edition 2 | 2020-10-05 | none | Published, confirmed (90.93). Replaced the old 16175-2:2011 and 16175-3:2010. | Technical Specification (guidance) | Verified fact | [S1][S10] |
| **ISO 19005-1** PDF/A-1 (PDF 1.4) | ISO 19005-1:2005, edition 1 | 2005-09-28 | Cor 1:2007, Cor 2:2011 | Published, confirmed (90.93) | Technical file-format standard | Verified fact | [S1][S11a] |
| **ISO 19005-2** PDF/A-2 (ISO 32000-1) | ISO 19005-2:2011, edition 1 | 2011-06-20 | none | Published, confirmed (90.93) | Technical file format | Verified fact | [S1][S11b] |
| **ISO 19005-3** PDF/A-3 (embedded files) | ISO 19005-3:2012, edition 1 | 2012-10-15 | none | Published, confirmed (90.93) | Technical file format | Verified fact | [S1][S11c] |
| **ISO 19005-4** PDF/A-4 (PDF 2.0, ISO 32000-2) | ISO 19005-4:2020, edition 1 | 2020-11-17 | none | Published but **to be revised (90.92)**. **ISO/DIS 19005-4.2 (edition 2) is at 40.60**, close of DIS voting. | Technical file format | Verified fact | [S1][S11d][S11e] |
| **ISO/IEC 27701** Privacy information management systems — Requirements and guidance | **ISO/IEC 27701:2025**, edition 2 | 2025-10-14 | none | Published (60.60). Replaces 27701:2019, which is withdrawn. | **Now a standalone requirements MSS.** ISO's FAQ says it can be used alone; the 2019 edition was an extension of 27001. | Verified fact | [S1][S12] |
| **ISO/IEC 27002** Information security controls | **ISO/IEC 27002:2022**, edition 3 | 2022-02-15 | none | Published (60.60). See Open questions on a possible successor. | **Guidance** (reference control set used with 27001); not certifiable | Verified fact | [S1][S13] |
| **ISO/IEC 27017** Cloud security controls based on 27002 | **ISO/IEC 27017:2026**, edition 2 | 2026-07-27 | none | Published (60.60). Replaces 27017:2015. | **Guidance / code of practice**; not a standalone certifiable MSS | Verified fact | [S1][S14] |
| **ISO/IEC 27018** PII protection in public clouds acting as PII processors | **ISO/IEC 27018:2025**, edition 3 | 2025-08-26 | none | Published (60.60). Replaces 27018:2019. | **Guidelines**; not a standalone certifiable MSS | Verified fact | [S1][S15] |

### Points the brief asked about specifically

- **ISO 9001 revision.** The revision has been **published and is no longer at DIS/FDIS stage**. ISO 9001:2026 (edition 6) was published on 2026-09-16, and the 2015 edition with its 2024 climate amendment is withdrawn [S1][S2][S3]. — **Verified fact**
  - The transition rules come from the mandatory Global ACI document **TECH-3-TR 2029-09-30 (M)**, issued 4 Sept 2026 with an implementation date of 16 Sept 2026 [S17]. — **Verified fact**
    - The transition period is **three years, ending 30 Sept 2029**.
    - Accreditation bodies must be ready by **31 Mar 2027**.
    - Certification bodies must submit transition declarations by **30 Jun 2027**.
    - Accreditation bodies must complete their decisions by **30 Sept 2027**.
    - **New and initial certifications must be to the 2026 edition only from 31 Mar 2028.**
    - Certified clients transition at a surveillance, recertification or special audit **before 30 Sept 2029**. After that date, 2015 certificates are no longer valid.
  - Global ACI describes the changes as **minimal** [S17]. — **Verified fact**. They include:
    - alignment with the ISO Harmonized Structure;
    - new wording on documented information, which clarifies "shall be available";
    - "quality culture and ethical behaviour" added to leadership and awareness;
    - risks and opportunities split into separate subclauses;
    - Annex B removed.
  - ISO's own page says certified organizations should contact their certification body about timing [S2]. — **Verified fact**
- **ISO/IEC 27001:2022/Amd 1:2024.** This amendment exists. It was published 2024-02-23 and is one page long [S1][S5]. — **Verified fact**
  - It is one of the "climate action" amendments to management system standards. It adds climate-change considerations to clauses 4.1 and 4.2 [S23]. — **Verified fact**
  - The IAF treated it as a clarification, with **no full transition programme** [S23]. — **Verified fact**
- **The 27001:2013 to 2022 transition is over.** Under IAF MD 26, 2013-based certificates expired after **31 Oct 2025** [S32]. — **Verified fact**. The source is an accredited certification body (NSAI) citing IAF MD 26. The NSAI page refers to the Irish adoption of the standard, I.S. EN ISO/IEC 27001:2017.
- **ISO 30301 is a certifiable requirements standard, and accreditation infrastructure exists for it.** ISO/IEC TS 17021-14:2022 sets competence requirements for auditing and certifying records management systems against ISO 30301 [S1]. — **Verified fact**
  - However:
    1. **30301 is not in the Global ACI MRA scope** [S20]. — **Verified fact**
    2. The 2019 edition is about to be replaced [S7]. — **Verified fact**
    3. No accreditation body was found that publicly lists an accredited 30301 scope. — **Open question**
  - Some certification bodies, such as MSECB, advertise ISO 30301 certification, but MSECB's brochure contains **no accreditation claim** [S41]. — **Verified fact**
  - **Conclusion (Estimate):** accredited 30301 certification is rare worldwide, and no source was found showing it is available in the Philippines.
- **ISO/IEC 27701 is outside the Global ACI MRA.** 27701 is not in the MRA scope list [S20], and Global ACI has issued no transition document for 27701:2025 [S21]. — **Verified fact**
  - Transition rules for 27701:2019 to 2025 certificates are set nationally. — **Open question**. Secondary sources disagree: one says UKAS requires transition by 31 Oct 2027, another says Oct 2028. Neither primary source could be loaded.
  - Certification against 27701:2019 is still being issued in PH. BSI Group Philippines certified BPI to 27001:2022 and 27701:2019, announced 3 Sept 2026 [S28]. — **Verified fact**

---

## 2. What "certification" actually means

**Who and what is certified**
- **An organization's management system** is certified, not a product. A **certification body** audits the management system against a **requirements** standard such as ISO 9001, ISO/IEC 27001, ISO/IEC 27701 or ISO 30301. These requirements standards are the ones whose scope says the document "specifies requirements" for a management system [S1]. — **Verified fact**
- **Certification bodies must be competent and impartial.** ISO/IEC 17021-1 sets requirements for their competence, consistency and impartiality, and describes management system certification as **third-party conformity assessment** [S1]. — **Verified fact**
  - For ISMS, ISO/IEC 27006-1:2024 adds further requirements [S1][S20]. — **Verified fact**
- **ISO itself does not certify anyone** [S16]. ISO says certification is voluntary, and that a certificate from an *accredited* certification body adds confidence because an accreditation body has independently confirmed that body's competence [S2][S4]. — **Verified fact**
  - ISO asks that certificates cite the full reference, for example "certified to ISO/IEC 27001:2022" [S4]. — **Verified fact**

**What "ISO 27001 certified software" really means**
- **A product or software is not "ISO 27001 certified".** 27001's requirements concern an organization's ISMS "within the context of the organization" [S1][S4]. — **Verified fact**
- A vendor can truthfully say "*our company* holds an ISO/IEC 27001:2022 certificate covering scope X", for example "development and operation of the DMS SaaS". It cannot truthfully say "the DMS is ISO 27001 certified". — **Estimate** (wording guidance derived from [S4] and [S1])
- **27002, 27017 and 27018 are guidance and control sets, not requirements standards** [S1][S13][S14][S15]. — **Verified fact**
  - In practice, certification bodies often reference 27017/27018 controls *within* a 27001 certificate scope rather than issuing standalone accredited certificates. Neither standard appears in the Global ACI MRA scope list [S20]. — **Estimate** (the MRA scope list is primary; the industry practice is not primary-verified)
- **PDF/A (ISO 19005) is a file-format specification.** Conformance is checked per file with validators such as veraPDF, an open-source validator for all PDF/A parts [S38]. It is not organizational certification. — **Verified fact**

**Accreditation chain (Philippines)**
- **PAB is the national accreditation body.** Executive Order 802 (2009) recognised the Philippine Accreditation Office, now the **Philippine Accreditation Bureau (PAB)** attached to DTI, as the **national accreditation body** and the central body that accredits inspection, testing and certifying bodies [S24]. — **Verified fact**
- **PAB accredits management-system certification bodies to (PNS) ISO/IEC 17021-1.** Philippine government tenders require "DTI-PAB accredited" certification bodies for ISO 9001 [S33][S34]. — **Verified fact**
- **The Bureau of Philippine Standards (BPS, DTI) is the National Standards Body.** It develops and promulgates Philippine National Standards [S26]. — **Verified fact**. BPS does **not** accredit or certify; that is PAB's and the certification bodies' role (inference from [S24][S26]). — **Estimate**
- **IAF MLA is now the Global ACI MRA.**
  - The IAF website now states that **IAF ceased operations on 1 January 2026** and points to Global ACI [S22]. — **Verified fact**
  - Global ACI has assumed the former roles of IAF and ILAC [S18]. — **Verified fact**
  - The mutual recognition arrangement is now the **Global ACI Multilateral Recognition Arrangement (MRA)**. Its signatories recognise each other's accreditations as equivalent, which is the basis for "certified once, accepted everywhere" [S19]. — **Verified fact**
  - The MRA's management-system scope covers ISO 9001, 14001, **27001 (with 27006-1)**, 13485, 50001, 45001, 37301, 37001, 22000 and some sector schemes, per Global ACI-FMRA-001 v3.3, issued 5 Aug 2026 [S20]. — **Verified fact**
  - Older documents and many Philippine sources still say "IAF MLA". Treat that as the same concept. — **Estimate**
- **PAB's MRA scopes: Open question.**
  - A DTI page (reported via search, since the page returned 404 / DNS failure on 2026-10-18) states that PAB is an IAF/APAC MLA signatory for QMS, EMS, FSMS and product certification [S27]. — **Open question**: could not be loaded on the access date.
  - PAB's current Global ACI MRA scope could not be confirmed. The Global ACI register uses a dynamic filter that could not be read. — **Open question**

---

## 3. Philippines landscape

### 3.1 ISO Survey counts

- **The ISO Survey moved to IAF CertSearch.** From the 2025 cycle the ISO Survey is compiled from IAF CertSearch data, and results are downloaded free *after logging in* to IAF CertSearch [S16]. — **Verified fact**
- **Philippines-specific counts for ISO 9001 and ISO/IEC 27001 could not be retrieved.** The per-country dataset sits behind the CertSearch login, and no public copy with PH rows was found. — **Open question**. The next step is for a human to create a free CertSearch account and download the latest survey file.
- **Global context, ISO Survey 2024** (published Sept 2025, via a secondary summary) [S36] — **Verified fact** (secondary reproduction of ISO Survey figures):

  | Standard | Certificates 2023 | Certificates 2024 |
  |---|---|---|
  | ISO 9001 | 837,978 | 1,474,118 |
  | ISO/IEC 27001 | 47,291 | 96,709 |

  - The jump reflects the switch to CertSearch data collection, not real growth alone.
  - ISO's own pages say ISO 9001 has "more than one million certificates in 189 countries" [S2], and that 27001 had over 70,000 certificates in 150 countries in the 2022 Survey [S4].
- Treat any PH number quoted by consultants as unverified until it is checked against the CertSearch download. — **Estimate**

### 3.2 Certification bodies operating in the Philippines (examples)

PAB's public directory of management-system certification bodies was read on 2026-10-18 [S25]. It listed 22 accreditation entries, all marked "Accredited". Its scheme filter offers **only EOMS, EMS, OH&SMS, FSMS, HACCP and QMS, with no ISMS** [S25]. — **Verified fact**

| Certification body (as listed by PAB) | PAB-accredited schemes listed |
|---|---|
| Bureau Veritas S.A. | QMS |
| Certification International Philippines Inc. (CIPI) | QMS, EMS, OH&SMS, FSMS, HACCP |
| DQS Certification Phil's. Inc. | QMS |
| Prime Certification and Inspection Asia Pacific Inc. | FSMS, HACCP |
| Seasafe Quality Assurance Inc. | QMS |
| SGS Philippines, Inc. | QMS, EMS, HACCP |
| SOCOTEC Certification Philippines Inc. | QMS, EMS, OH&SMS |
| TÜV NORD Philippines, Inc. | QMS |
| TÜV Rheinland Philippines, Inc. | QMS |
| TÜV SÜD PSB Philippines, Inc. | QMS, HACCP, EOMS |
| WQA Phils Inc. | QMS |

Source: [S25]. — **Verified fact**

**ISO/IEC 27001 in the Philippines**
- **BSI Group Philippines Inc. issues 27001 and 27701 certificates.** It certified BPI to ISO/IEC 27001:2022 and ISO/IEC 27701:2019, announced 2026-09-03 [S28]. — **Verified fact**
- BSI does not appear in PAB's directory, so its accreditation must come from a non-PAB accreditation body. — **Estimate**. Which body (for example UKAS or ANAB) was not verified.
- Implication: buyers in the PH should check the accreditation mark on any 27001 certificate, for example via IAF CertSearch. — **Estimate**

---

## 4. Certification process and timeline

**Typical cycle** (accredited certification body sources) — **Verified fact** [S30][S31]:
1. **Application and contract.** Includes a proposal covering scope, fee and planning [S30].
2. **Stage 1 audit.** Checks management-system design and readiness for stage 2, and is largely a documentation review [S30][S31].
3. **Stage 2 audit.** Assesses whether the system is effective and implemented. Any major nonconformities must be corrected before certification [S30][S31].
4. **Certification decision** by the certification body, separate from the auditors, then the certificate is issued [S30].
5. **Surveillance audits at least annually** [S30][S31].
6. **Recertification** before the end of the **three-year** certificate validity [S30][S31].

**Readiness and timing details**
- **NQA's readiness bar:** the system must have run for **at least three months** before assessment, including a management review and a full cycle of internal audits [S31]. — **Verified fact** (one certification body's published requirement; others may differ)
- **A PH government buyer's timetable** for ISO 9001 certification [S34]. — **Verified fact**:
  - stage 2 report due ≤14 days after the stage 1 report;
  - certificate due ≤30 days after a clean stage 2;
  - surveillance audits ≤12 months apart.
- **Audit days:** under the IAF MD 5 methodology for QMS, audit time is driven by "effective number of personnel" and risk. Surveillance audits are typically about one-third of initial audit time and rarely under one day [S40]. — **Verified fact** (secondary training copy of IAF MD 5:2019; the current issue was not checked)
- ISMS audit time follows ISO/IEC 27006-1 instead [S1][S20]. — **Verified fact**
- **End-to-end timeline for an SME from zero to certificate:** no authoritative published figure was found. Practitioner sources commonly suggest about 6–12 months. — **Estimate, unverified**

**Cost**
- **No credible published SME cost range for the Philippines was found.** — **Open question**
- The only primary price data are **Philippine government procurement ceilings** (Approved Budget for the Contract), which cover **certification body fees only**, for public agencies rather than SMEs. — **Estimate** (indicative only):
  - Insurance Commission (2018): **PhP 469,000 ceiling for three years** of new ISO 9001 certification plus pre-audit [S34].
  - PIDS (2024): **PhP 400,000 ceiling** for one recertification audit plus two surveillance audits [S33].
  - These figures exclude internal staff time, consultants and tools.

---

## 5. What a DMS can and cannot contribute

Clause and control **titles only** are given below. No requirement text is reproduced.

### 5.1 Where a DMS directly helps — **Estimate** (mapping is analysis; titles are **Verified fact**)

**Clause 7.5 "Documented information"**
- Applies to both standards.
- ISO/IEC 27001:2022 has 7.5.1 General, 7.5.2 Creating and updating, and 7.5.3 Control of documented information [S37].
- ISO 9001:2015 had the same pattern [S39].
- ISO 9001:2026 keeps documented-information requirements with clarified wording [S17]. Its exact 2026 subclause numbering was not verified.
- **DMS contributions:**
  - identification (title, date, author, reference) and format;
  - review and approval workflow;
  - version control and change history;
  - controlled distribution and access;
  - storage and preservation, for example PDF/A;
  - retention and disposition;
  - control of external documents.

**ISO/IEC 27001:2022 Annex A controls relevant to documents** (titles verified in a public Statement of Applicability [S35])

| Control | DMS contribution |
|---|---|
| A.5.9 Inventory of information and other associated assets | Document and repository inventory |
| A.5.12 Classification of information; A.5.13 Labelling of information | Classification metadata and visible labels |
| A.5.14 Information transfer | Controlled sharing and external links |
| A.5.15 Access control; A.5.18 Access rights; A.8.3 Information access restriction | Role-based permissions and access reviews |
| A.8.5 Secure authentication | SSO and MFA |
| A.5.33 Protection of records | Immutability, retention locks, audit trail |
| A.5.34 Privacy and protection of PII | PII tagging and access limits |
| A.5.37 Documented operating procedures | Hosting the SOP library |
| A.8.10 Information deletion | Retention-driven disposal with evidence |
| A.8.12 Data leakage prevention | Download and share restrictions |
| A.8.13 Information backup | Backups |
| A.8.15 Logging | Access and change audit logs |
| A.8.24 Use of cryptography | Encryption at rest and in transit |

**Records standards**
- **ISO 15489-1** and **ISO 16175-1/-2** provide records concepts and model functional requirements for software that manages records [S8][S9][S10]. They are a good **feature checklist** for the product. — **Estimate**
- **ISO 19005 (PDF/A)** suits long-term preservation renditions [S11a–e]. — **Estimate**

### 5.2 What a DMS cannot do (requires people, policy and the certification body) — **Estimate** (derived from the scope of requirements MSS [S1][S4] and the certification process [S30][S31])
- **Define the management system.** This means context, scope, interested parties, and the quality, ISMS or privacy policy and objectives.
- **Leadership commitment and assigned roles.** The partner's management must own these.
- **Risk assessment and treatment.** For 27001 this includes the Statement of Applicability and the risk treatment plan. A DMS can *store* them but cannot *decide* them.
- **Competence, training and awareness** of staff.
- **Operating the processes** themselves, whether quality or security.
- **Internal audit and management review.** These need people with independence and authority. A DMS can schedule them and hold the records.
- **Corrective action and continual improvement.**
- **The certification audit and decision.** Only an accredited certification body does this. A tool vendor cannot "grant" or "guarantee" certification.

---

## 6. Three possible goals and what each implies

| | **(A) Partner's own firm gets certified** | **(B) Product supports customers' certification** | **(C) Vendor gets ISO/IEC 27001 certified** |
|---|---|---|---|
| **Likely standard** | ISO 9001:2026 if it is about quality/process and tenders; ISO/IEC 27001:2022 if clients demand security assurance | Feature design informed by 9001/27001 clause 7.5, 27001 Annex A, ISO 15489-1 and ISO 16175 | ISO/IEC 27001:2022 (+Amd 1); optionally 27701:2025 for privacy, with 27017/27018 controls referenced in scope |
| **Who is certified** | The partner's firm | Nobody: the customers get certified, the product helps | The vendor company that builds and runs the DMS |
| **What it requires** | A management system run by the firm's people. Certification body audits: stage 1, stage 2, annual surveillance, recertification in year 3 [S30][S31]. | Product features plus honest marketing claims, e.g. "supports clause 7.5 evidence", **not** "ISO-certified DMS" [S4] | A full ISMS covering development, hosting, support and suppliers, audited by a certification body accredited for ISMS |
| **PH specifics** | For ISO 9001, PAB-accredited certification bodies exist (table in §3.2). Government tenders often require PAB accreditation [S33][S34]. Starting now means certifying to **9001:2026**; 2015-edition initial certifications end by 31 Mar 2028 [S17]. | DMS buyers pursuing 9001 or 27001 are the target market. PH certificate counts are unknown (Open question). | **PAB lists no ISMS accreditation** [S25], so a certification body accredited elsewhere is needed; BSI Philippines is one active example [S28]. ISO offers an SME handbook for 27001 [S4]. |
| **Relevance to building a DMS** | Indirect. The DMS could be the firm's own document-control tool, but certification is mostly organizational work. | **Direct.** This is a product and positioning decision. | **Direct, if selling to banks, BPOs and regulated buyers.** It is a cost and commitment decision for the vendor. |
| **Main risk** | Treating a tool purchase as "getting ISO" | Over-claiming ("ISO-certified software") | Underestimating ongoing cost: annual surveillance and a 3-year cycle |

Labels: rows citing sources are **Verified fact** for the cited element. The framing, relevance and risk assessments are **Estimates**.

---

## 7. Key implications for the senior partner conversation (**Recommendation**)

1. **First, ask which goal the partner means: A, B or C.** "Interested in ISO certification" maps to three very different investments. A and C are organizational programs with recurring audit costs. B is a product-positioning choice.
2. **If the partner means A or B for quality, say "ISO 9001:2026", not "9001:2015".** The 2015 edition is withdrawn, and new initial certifications to it stop by 31 Mar 2028 [S17]. Content built around 2015 clause wording will need updating.
3. **If the partner means C, or B aimed at security-conscious buyers, ISO/IEC 27001:2022 is the benchmark.** PAB currently lists no ISMS-accredited certification bodies, so plan for a foreign-accredited one and verify accreditation on IAF CertSearch.
4. **Do not promise "ISO-certified software".** The defensible claims are:
   - "our company is certified to ISO/IEC 27001:2022 for scope X" (goal C); or
   - "the product helps you meet documented-information and records controls" (goal B).
5. **Records-management standards (15489-1, 16175, PDF/A) are good product requirements but poor certification targets for an SME.** ISO 30301 is about to be replaced, and accredited 30301 certification is rarely available (Open question for PH).
6. **Get real PH numbers before sizing the market.** Pull the free ISO Survey dataset from IAF CertSearch for Philippines ISO 9001 and 27001 counts.

---

## 8. Open questions

1. **PH certificate counts:** what are the Philippines counts for ISO 9001 and ISO/IEC 27001 in the latest ISO Survey? This needs an IAF CertSearch login [S16].
2. **PAB's current MRA scopes:** what are they under Global ACI, and does PAB plan to add ISMS (ISO/IEC 27001 / 27006-1) accreditation? The DTI "About PAB" pages returned 404 or DNS errors on 2026-10-18.
3. **27701 transition:** what deadline applies to 27701:2019 certificates moving to 27701:2025? It is not covered by the Global ACI MRA, and secondary sources disagree (2027 vs 2028).
4. **Accredited ISO 30301 certification:** does any accreditation body accredit certification bodies for it, especially in APAC or the PH? When will ISO 30301 edition 3 publish? It is at 50.00 as of 2026-09-07.
5. **27002 successor:** ISO Open Data lists a successor ID (94703) for ISO/IEC 27002:2022, but no record for that ID exists in the dataset. Is a 27002 revision formally registered?
6. **16175-1 review:** what was the outcome of the systematic review of ISO 16175-1:2020 (stage 90.60)? Is there any accredited product-conformity scheme for ISO 16175-1? None was found.
7. **Real SME costs:** what are actual certification body quotes for a PH SME of the partner's size, for 9001:2026 and for 27001:2022? No credible published figure was found.
8. **The partner's own situation:** does their firm already hold any ISO certificate, and are customers or tenders asking for one? This is a question for the partner, not for research.

---

## Sources (all accessed 2026-10-18)

**ISO**
- [S1] ISO Open Data — Deliverables metadata (JSONL, ODC-By 1.0; file Last-Modified 2026-10-14). https://isopublicstorageprod.blob.core.windows.net/opendata/_latest/iso_deliverables_metadata/json/iso_deliverables_metadata.jsonl ; licence page: https://www.iso.org/open-data.html
- [S2] ISO 9001:2026 catalogue page. https://www.iso.org/standard/88464.html (read via committee.iso.org mirror)
- [S3] ISO 9001:2015 catalogue page (withdrawn). https://www.iso.org/standard/62085.html
- [S4] ISO/IEC 27001:2022 catalogue page. https://www.iso.org/standard/82875.html
- [S5] ISO/IEC 27001:2022/Amd 1:2024. https://www.iso.org/standard/88435.html
- [S6] ISO 30301:2019. https://www.iso.org/standard/74292.html
- [S7] ISO/PRF 30301 (edition 3). https://www.iso.org/standard/89772.html
- [S8] ISO 15489-1:2016. https://www.iso.org/standard/62542.html
- [S9] ISO 16175-1:2020. https://www.iso.org/standard/74294.html
- [S10] ISO/TS 16175-2:2020. https://www.iso.org/standard/74293.html
- [S11a] ISO 19005-1:2005. https://www.iso.org/standard/38920.html
- [S11b] ISO 19005-2:2011. https://www.iso.org/standard/50655.html
- [S11c] ISO 19005-3:2012. https://www.iso.org/standard/57229.html
- [S11d] ISO 19005-4:2020. https://www.iso.org/standard/71832.html
- [S11e] ISO/DIS 19005-4.2. https://www.iso.org/standard/86943.html
- [S12] ISO/IEC 27701:2025. https://www.iso.org/standard/85819.html
- [S13] ISO/IEC 27002:2022. https://www.iso.org/standard/75652.html
- [S14] ISO/IEC 27017:2026. https://www.iso.org/standard/82878.html
- [S15] ISO/IEC 27018:2025. https://www.iso.org/standard/88150.html
- [S16] ISO — The ISO Survey. https://www.iso.org/the-iso-survey.html

**Global ACI and IAF**
- [S17] Global ACI-TECH-3-TR 2029-09-30 (M), Transition Requirements for ISO 9001:2026, v1.1, issued 2026-09-04. https://sys.global-aci.org/uploads/documents/TECH-3-TR_2029-09-30_(M)_ISO_9001_2026_Transition_v1_12.pdf
- [S18] Global Accreditation Cooperation Inc. — Home. https://global-aci.org/en/home/
- [S19] Global ACI — About the Global ACI MRA. https://global-aci.org/en/about-the-global-aci-mra/
- [S20] Global ACI — Scopes, and Global ACI-FMRA-001 v3.3 (2026-08-05). https://global-aci.org/en/scopes/ ; https://sys.global-aci.org/uploads/documents/FMRA-001_Global_Accreditation_Cooperation_MRA_Status_v3_3_2026-09-28.pdf
- [S21] Global ACI — Technical Documents (Transition). https://global-aci.org/en/global_aci-documents/?cat_id=13
- [S22] IAF legacy website (states IAF ceased operations 2026-01-01). https://iaf.nu/en/home/
- [S23] IAF-ISO Joint Communiqué on Climate Change amendments (Feb 2024). https://iaf.nu/wp-content/uploads/2024/02/IAF-ISO-Joint-Communique-_2024F.pdf

**Philippines**
- [S24] Executive Order No. 802 (2009), Lawphil. https://www.lawphil.net/executive/execord/eo2009/eo_802_2009.html
- [S25] DTI-PAB — Management System Certification Bodies directory (pages 1–5). https://pabaccreditation.dti.gov.ph/723ed643a71e686c56c6fb8c1c3e7a41190ac9740d3108c806342707f4428728767599da8b05774d408dca4ff000fef55e
- [S26] DTI Bureau of Philippine Standards — Home. https://bps.dti.gov.ph/
- [S27] DTI-PAB "About us" (seen only as search snippet; page unreachable 2026-10-18). https://dti.gov.ph/pab/about-us
- [S28] BPI press release, 2026-09-03 (BSI Group Philippines; ISO/IEC 27001:2022 and 27701:2019). https://www.bpi.com.ph/about-bpi/news/bpi-sets-benchmark-among-philippine-universal-banks-with-dual-iso-certification

**Certification process and cost**
- [S30] EY CertifyPoint — Certification approach and conditions (Feb 2025). https://www.ey.com/content/dam/ey-unified-site/ey-com/en-gl/services/consulting/documents/ey-certifypoint-certification-approach-and-conditions-v1.pdf
- [S31] NQA — Certification process. https://www.nqa.com/en-in/resources/knowledge-hub/certification-process
- [S32] NSAI — ISO/IEC 27001:2022 transition policy. https://www.nsai.ie/certification/management-systems/iso-iec-27001-information-security-management-system/iso-iec-270012022-transition-policy/
- [S33] PIDS — Terms of Reference, Certifying Body for recertification and surveillance audits (ISO 9001:2015), 2024. https://pidswebs.pids.gov.ph/CDN/document/1721284931_6698b943b46f1.pdf
- [S34] Insurance Commission — RFQ, Third-Party Certifying Body for QMS certification (2018). https://www.insurance.gov.ph/wp-content/uploads/2022/02/RFQ_ISO2018-compressed.pdf

**Secondary and supporting**
- [S35] Euronext Clearing — public Statement of Applicability, ISO/IEC 27001 (Annex A titles). https://www.euronext.com/sites/default/files/2025-12/Euronext%20Clearing%20SoA%20-%20ISO27001.pdf
- [S36] ISBL — "ISO Survey 2024" summary (secondary, Spanish; cites ISO Survey/CASCO Sept 2025). https://isbl.eu/wp-content/uploads/2025/10/ISO-SURVEY-2024.pdf
- [S37] Advisera — ISO 27001 clause 7.5 Documented information (subclause titles). https://advisera.com/iso27001/clause-7-5-documented-information/
- [S38] veraPDF. https://verapdf.org/
- [S39] ISMS.online — ISO 9001 clause 7.5 (refers to 9001:2015). https://isms.online/iso-9001/clause-7-5-documented-information
- [S40] IAS-hosted training copy of IAF MD 5:2019 (audit time). https://cdn-v2.iasonline.org/wp-content/uploads/2021/02/6.-IAF-MD5.pdf
- [S41] MSECB — ISO 30301 certification brochure (no accreditation claim found). https://msecb.com/wp-content/uploads/ISO-30301-MS-Certification-MSECB.pdf
