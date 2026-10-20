# R10 — Data Privacy Act duties for a SaaS Documents app (PIC vs PIP)

- **Status:** First pass, complete. Research only, **not legal advice**. Prepared for a Philippine lawyer to confirm.
- **Researched:** 2026-10-19. **All sources accessed 2026-10-19.**
- **Question:** What do RA 10173, its IRR and NPC issuances require of (a) Officetul's client businesses as **personal information controllers (PICs)** and (b) the vendor as **personal information processor (PIP)**? What does that mean for Documents-app features, contracts and operations?
- **Labels:** **Verified fact** (read in the primary text), **Secondary** (law-firm or tracker commentary, or a search excerpt only), **Open question**, **Recommendation** (proposed, not approved). Source IDs `[S#]` point to §13.

**Access caveat.** privacy.gov.ph returned **HTTP 403** to automated fetching again. Every NPC text below was read from **Internet Archive** copies: the IRR snapshot of 2024-11-12, the NPC issuances list of 2026-10-12, the advisory-opinions list of 2026-10-11, and the PDFs captured on those dates. NPC Advisory 2021-01 was read from a DataGuidance mirror because the archived NPC copy returned 404. RA 10173 and the Rules on Evidence were read on lawphil.net. The CPRA was read in a ChanRobles reproduction (as in R08).

---

## 1. Who is PIC and who is PIP

| Point | Detail | Label |
|---|---|---|
| PIC definition | A PIC "controls the collection, holding, processing or use of personal information", including one who "instructs another… to collect, hold, process, use, transfer or disclose personal information on his or her behalf". The definition **excludes** anyone who "performs such functions as instructed by another". RA s.3(h) | **Verified fact** [S1] |
| PIP definition | A PIP is any person "to whom a personal information controller may outsource the processing of personal data". RA s.3(i) | **Verified fact** [S1] |
| Storage provider = PIP | "When a PIC engages a service provider for the purpose of **storing** personal data… the service provider acts as a PIP." NPC Circular 2023-06 s.10 | **Verified fact** [S4] |
| Mapping | Client business = **PIC** for documents it uploads or emails in. Officetul = **PIP** for that content. Officetul is a **PIC** for its own account, billing and marketing data about client users. | **Recommendation** (lawyer to confirm) |
| PIP must comply directly | "The personal information processor shall comply with all the requirements of this Act and other applicable laws." RA s.14. This is repeated in IRR s.45. | **Verified fact** [S1][S2] |
| PIC stays accountable | The PIC remains responsible for data "transferred to a third party for processing, whether domestically or internationally". RA s.21, IRR s.50. In *In re FLI* (NPC 19-910), the NPC held that a PIC "cannot surrender its accountability… to the PIP". This is cited in AO 2022-024. | **Verified fact** [S1][S2][S20] |
| Sensitive data in scope | Sensitive personal information (SPI) includes health and education data; "any proceeding for any offense committed or alleged"; and government-issued identifiers, including social security numbers, health records, licences and **tax returns**. RA s.3(l). Privileged information is defined in s.3(k). | **Verified fact** [S1] |

### 1.1 The outsourcing agreement (IRR ss.43–44)

The contract must exist, and IRR s.44 lists its contents. NPC AO 2020-018 adds that "the applicable terms and conditions" of a cloud service, such as Gmail or OneDrive for the University of the Philippines, "shall suffice for as long as proper safeguards… are in place". The "contract **or other legal act**" can therefore be Officetul's standard terms plus a DPA addendum. A separately negotiated contract is not required. **Verified fact** [S2][S19]

| IRR s.44 item | Required content |
|---|---|
| (a) | Subject matter and duration; nature and purpose; types of personal data; categories of data subjects; the PIC's rights and obligations; **the geographic location of the processing** |
| (b)(1) | Process **only on the PIC's documented instructions**, including transfers to another country |
| (b)(2) | Confidentiality obligation on everyone authorised to process |
| (b)(3) | Appropriate security measures; comply with the Act, the IRR and NPC issuances |
| (b)(4) | **No sub-processor "without prior instruction from the PIC"**, with the same obligations flowed down |
| (b)(5) | Assist the PIC with data subject requests |
| (b)(6) | Assist the PIC with compliance generally |
| (b)(7) | **At the PIC's choice, delete or return all personal data at the end of services, including existing copies**, unless storage is authorised by law |
| (b)(8) | Provide compliance information; **allow and contribute to audits and inspections** by the PIC or its mandated auditor |
| (b)(9) | **Immediately inform the PIC** if an instruction would infringe the Act or NPC rules |

**Label:** **Verified fact** [S2]. The NPC reproduced the list in AO 2017-015, adding that outsourcing contracts are "subject to the review of the Commission" [S15].

**Related rules:**

- IRR s.26(f) says a PIC may engage only PIPs that "provide sufficient guarantees". **Verified fact** [S2]
- Circular 2023-06 s.13: PIP processing "must be covered by the appropriate and necessary agreement that contains an equivalent" of the PIC's **security-clearance** rule. **Verified fact** [S4]
- Circular 2023-06 s.14: PIP access is governed by "strict procedures contained in formal contracts", and the NPC may weigh those terms when it judges the PIC. **Verified fact** [S4]
- A **data sharing agreement (DSA)** is a different instrument, for PIC-to-PIC sharing. It is **optional** (NPC Advisory 2025-01). Sharing "incidental to a subcontracting agreement" between PIC and PIP is excluded from "data sharing" (Circular 2022-04 s.2(E)). Officetul therefore needs an **outsourcing agreement, not a DSA**. **Verified fact** [S12][S7]

---

## 2. Security requirements

### 2.1 Statute and IRR

- **RA s.20 duties.** The Act puts these duties on the **PIC**: s.20(a)–(c) measures, s.20(c)(1)–(4) network safeguards, a security policy, vulnerability management and breach monitoring. Under s.20(d) the PIC "must further ensure that third parties processing… on its behalf" implement them. **Verified fact** [S1]
- **The IRR extends them to PIPs.** IRR ss.25–28 say "personal information controllers **and personal information processors**". **Verified fact** [S2]

| IRR section | Key requirements (all apply to PICs and PIPs, "where appropriate") |
|---|---|
| s.25 | Organisational, physical and technical measures for availability, integrity and confidentiality. People acting under authority process only on instructions. |
| s.26 | (a) DPO or compliance officer; (b) data protection policies with privacy by design and by default; (c) **records of processing**; (d) staff selection, perpetual confidentiality and **training**; (e) access-management, monitoring and incident policies, data-subject-rights procedures and a **retention schedule** |
| s.27 | Physical access control; policies for transfer, removal, **disposal and re-use of electronic media**; protection against disasters and power problems |
| s.28 | Security policy; network safeguards; ensure **confidentiality, integrity, availability and resilience**; monitoring and vulnerability handling; **timely restore** after an incident; **regular testing** of controls; **(g) "Encryption of personal data during storage and while in transit, authentication process"** and access controls |
| s.29 | The NPC judges adequacy by data sensitivity, risk, organisation size, best practice and cost |

**Label:** **Verified fact** [S2]

### 2.2 NPC Circular 2023-06 (Security of Personal Data in the Government and the Private Sector)

- **Number, dates and scope.** The circular is dated **1 December 2023**. It was published in the Daily Tribune on **15 March 2024** and took **effect on 30 March 2024**, per the NPC FAQ. It **repeals Circular 16-01** (s.39). The 12-month transitory period (s.37) has ended. It applies to "all natural or juridical persons engaged in the processing of personal data" (s.1). Most clauses say "**PIC or PIP**". **Verified fact** [S4][S5]

| s. | Requirement | Applies to | Documents-app relevance |
|---|---|---|---|
| 4 | Designate and register a DPO; register systems; **inventory** of systems; **PIA**; privacy management programme; **periodic training**; comply with NPC orders | PIC **and its PIP** | Vendor needs its own DPO, PIA and training programme |
| 5 | PIA content: data inventory with **repository locations** and media; lifecycle "from collection to disposal" | PIC and PIP | Vendor publishes its storage locations and data flows |
| 7 | Privacy by design and **by default**. PIA **off-the-shelf software**. Functions without a lawful basis "must be switched off" | PIC or PIP | Clients must PIA Officetul. Optional data-hungry features (AI or OCR extraction, sharing links) should default off |
| 9 | Keep data only as long as necessary; **PIC** documents a **retention policy** and tells data subjects | PIC | Product must let the PIC apply retention |
| 11 | Industry-standard protection; enforced **password policy** | PIC or PIP | Password rules in the app |
| 12 | Databases accessed or modified only through **authorised software** | PIC or PIP | Vendor support access through approved tooling only, never ad-hoc database clients |
| 13 | **Access control policy**, "need to know"; authentication and monitoring; **security clearance** filed with the PIC's DPO; the PIP contract carries an equivalent | PIC or PIP | Folder and matter permissions; vendor staff clearances |
| 16 | "**Secure authentication mechanisms, such as multifactor authentication or secure encrypted links**" for personnel online access to **SPI, privileged information and high volumes**, controlled by a system management tool | PIC or PIP | **MFA must be available (and enforceable per tenant) for client users. MFA is mandatory for vendor admins.** |
| 17 | Only known, configured devices; endpoint security; USB restrictions | PIC or PIP | Vendor endpoint policy |
| 18 | Remote disconnect or wipe of **PIC-owned** mobile devices | PIC | A future mobile app should support session revoke and cache wipe |
| 19 | For **physical** files, an access log (who, when, where, copies made) | PIC or PIP | Optional: register paper originals and record check-outs |
| 21 | **Business continuity plan**: backup, restoration and "remedial time"; **periodic testing**; disaster recovery; crisis communications | PIC or PIP | Backups, documented restore tests, RPO/RTO in the DPA |
| 22 | Telecommuting policy and training | PIC or PIP | Vendor remote-work policy |
| **23** | **Email:** "use secure transmission and reception of email messages, including attachments"; may scan outgoing mail for personal data | PIC or PIP | **Per-tenant scan inbox: TLS in transit, sender authentication, quarantine** |
| 24 | Where reasonable, stop personnel copying or printing personal data into productivity software | PIC or PIP | Per-role download, print and export permissions |
| 25 | Removable media regulated; files on it encrypted | PIC or PIP | Encrypted export bundles |
| 26 | **No fax** | PIC or PIP | — |
| 28, 30 | Disposal policy: retention, local laws, deletion or anonymisation technique, **documentation before deletion**; secure wiping and media destruction | PIC or PIP | Deletion certificates and disposal log |
| 29 | **Log retention:** security and authentication logs kept longer than general logs; incident logs kept as long as the NPC requires; **log backup and archive** | PIC or PIP | Immutable audit and security logs with a defined retention |
| 31 | A disposal provider must contractually follow the PIC's procedures | PIP | Contract-end deletion |
| 32 | **Threat monitoring and vulnerability management** using sources such as NCERT; feed results into the PIA | PIC or PIP | Vendor vulnerability management programme |
| 34 | NPC "may accept" **independent verification or certification** of the PIC or PIP | — | ISO/IEC 27001/27701 or SOC 2 would help, but none is required (FAQ IX) |
| 35 | Sanctions: compliance orders, cease-and-desist orders, processing bans, fines | PIC or PIP | — |

**Labels:**

- Rows: **Verified fact** [S4]
- The "Documents-app relevance" column: **Recommendation**

**Notes:**

- **No cloud-specific section exists.** "Data Center" is defined to include virtual centres "owned and controlled by the PIC or not" (s.3(G)). The FAQ says online access covers "documents stored in the cloud". **Verified fact** [S4][S5]
- The FAQ also says a username, password and approver "may not be sufficient". **Verified fact** [S5]
- **"High volume" is undefined.** The FAQ points to the Circular 2022-04 registration thresholds as a "baseline". **Verified fact** [S5]
- Circular 2023-06 itself does not mention encryption at rest. That duty comes from **IRR s.28(g)**. NPC Circular 16-03 s.7 also lists backups, "access control and secure log files" and encryption. **Verified fact** [S2][S6]

---

## 3. Breach management

| Point | Detail | Label |
|---|---|---|
| Governing circular | **NPC Circular 16-03** (15 Dec 2016) is still listed on the NPC issuances page as of 2026-10-12. **No successor circular was found.** | **Verified fact** [S3][S6] |
| 2026 procedural advisory | NPC Advisory **2026-02** (11 May 2026), titled "Clarification on the Submission of Personal Data Breach Notification through DBNMS". Commentary says: requests go through the NPC's breach-notification portal (DBNMS); filing a request does not suspend Circular 16-03 duties; an exemption request cannot be combined with a postponement or alternative-notification request; NPC silence is not approval. It is not said to amend 16-03. | Title and date: **Verified fact** [S3]. Number and content: **Secondary** [S23] (archived PDF returned 404, not read) |
| Who notifies | The **PIC** notifies the NPC and data subjects. The duty "remains with the personal information controller even if the processing… is outsourced". 16-03 s.15 | **Verified fact** [S6] |
| PIP duty | The PIC must "use contractual or other reasonable means" so the PIP reports "upon the knowledge of, or reasonable belief" of a breach (16-03 s.16). The PIP keeps its own incident policy, response team and documentation (ss.4, 5, 8(G), 9). | **Verified fact** [S6] |
| **Clock** | 72 hours "upon knowledge of or the reasonable belief by the personal information controller **or personal information processor**" (16-03 s.17(A); IRR s.38(a)). **The PIP's discovery may start the PIC's 72 hours.** | **Verified fact** [S2][S6] |
| Trigger | SPI or identity-fraud data (which expressly includes "copies of identification documents… PhilHealth, SSS, GSIS, TIN"), plus likely unauthorised acquisition, plus real risk of serious harm (16-03 s.11). A DMS of scanned IDs and tax returns will meet the first limb in most incidents. | **Verified fact** [S6] |
| No delay allowed | If **≥100 data subjects** are affected, or the SPI disclosure will cause harm, notify within 72 hours on the information available. The **full report is due within 5 days** (s.17(C)). Failure to notify is **presumed** after 5 days (s.20). | **Verified fact** [S6] |
| Contents (NPC) | **Nature:** how the breach happened and the vulnerability, a chronology, the number of subjects or records, likely consequences, DPO contact. **Data involved.** **Measures taken:** containment, recovery, mitigation, data-subject notice, prevention (s.17(D)). | **Verified fact** [S6] |
| Contents (data subjects) | Nature, data involved, measures, harm reduction, a contact person, assistance offered. Notice is **individual** unless the NPC approves alternative means (s.18). | **Verified fact** [S6] |
| Register and annual report | Document **all** security incidents and breaches, including those not notifiable. Submit an **annual summary** to the NPC (IRR s.41(b); 16-03 s.22). An NPC advisory granted extra time for **2025 Annual Security Incident Reports**. | **Verified fact** [S2][S6][S3] |
| Does a PIP file its own annual report? | 16-03 s.22 does not name the filer. | **Open question** |

---

## 4. Registration and DPO

| Point | Detail | Label |
|---|---|---|
| Mandatory registration | A PIC **or PIP** must register all data processing systems if it has **≥250 employees**, **or** processes SPI of **≥1,000 individuals**, **or** its processing is likely to pose a risk. Systems with automated decision-making or profiling must always register (Circular 2022-04 s.5). Otherwise the entity files a **sworn declaration** (s.6). | **Verified fact** [S7] |
| **SaaS clause** | "A PIC who uses a system as a service shall register the same indicating the fact that processing is done through a service provider. **A PIP who uses its own system as a service to process personal data must register with the Commission.**" (s.5(B)) | **Verified fact** [S7] |
| Officetul's position | Across all tenants, Officetul will almost certainly hold SPI of ≥1,000 individuals. That alone makes registration mandatory, and s.5(B) points the same way. | **Recommendation:** register as PIP. Lawyer to confirm whether s.5(B) applies regardless of thresholds. |
| Mechanics | Register within **20 days** of a new system starting, and update within **10 days** of changes (s.7). Certificate valid **1 year** (s.14), renewable 30 days before expiry (s.18). DPO uses a **dedicated** email address (s.8). The **Seal of Registration** must be displayed, including on the website (s.32). Failing to register or update true details: **₱50,000–₱200,000** (Circular 2022-01). | **Verified fact** [S7][S8] |
| DPO for PIPs | "Any natural or juridical person… involved in the processing" designates a DPO (IRR s.26(a)). Circular 2023-06 s.4(A) requires the PIC "**and its PIP**" to designate **and register** a DPO. | **Verified fact** [S2][S4] |
| DPO profile | The DPO "should be a full-time or organic employee". A contract DPO should have a term of at least 2 years. Avoid conflicts of interest (NPC Advisory 2017-01). Circular 2022-04 allows a **Common DPO** across related companies (s.8). | **Verified fact** [S9][S7] |
| Client registration | Clients that cross the thresholds must list Officetul as a service provider. Law firms holding court-proceeding SPI of ≥1,000 people cross it. | **Verified fact** (rule) [S7]. **Recommendation:** provide a "registration info pack". |

---

## 5. Data subject rights that hit a DMS

| Right (RA s.16 / IRR s.34) | What it requires | DMS support (**Recommendation**) |
|---|---|---|
| Access, s.16(c) | Contents, sources, **recipients**, manner of processing, reasons for disclosure, and the "**date when… last accessed and modified**" | **Search all documents tagged to or mentioning a person.** Per-document access and modification log. Share and recipient log. Export of the bundle. |
| Rectification, s.16(d) | Correct the data. Keep "both the new and the retracted information" accessible. Inform past recipients on request. | Versioning that keeps prior versions. Annotation for "corrected". Recipient list. |
| Erasure or blocking, s.16(e) / IRR s.34(e) | Suspend, withdraw, **block**, remove or destroy, on the stated grounds | A **"blocked" state** (restricted, not deleted). Hard delete with a disposal log. Delete across versions, OCR text, thumbnails and search index. |
| Portability, s.18 / IRR s.36 | A structured, commonly used copy, where processing is electronic and based on consent or contract | Export in "XML, JSON, CSV etc." per Advisory 2021-01 s.11(D). Original files plus metadata. |
| Object, IRR s.34(b) | Stop processing unless a subpoena, an obvious purpose or a legal obligation applies | Same as blocking |

**Label:** Rights text: **Verified fact** [S1][S2].

**Procedure (NPC Advisory 2021-01):**

- PICs must ensure "by contractual or other reasonable means" that PIPs "cooperate and coordinate" on requests.
- Reply within **30 working days**, extendable by **15 working days** for complex requests.
- No fee, except reasonable copying costs.
- Verify the requester's identity.
- PICs may not retain data "for the sole purpose of making it available for potential future requests".

**Label:** **Verified fact** [S10]

**Legal retention beats erasure:**

- An erasure request "may be denied" where the data is still needed for a legal obligation, legal claims, or legitimate business purposes "consistent with the applicable industry standard for… retention" (Advisory 2021-01 s.10). **Verified fact** [S10]
- AO 2018-039 says the right "is not absolute and is subject to existing laws… governing the retention period". **Verified fact** [S17]
- The product therefore needs **legal hold or blocking that overrides delete**. Every refused request should be recorded with its ground. **Recommendation**

**Redaction:**

- No DPA provision expressly requires redaction tooling. Access copies must not expose third parties: portability "shall not adversely affect the rights and freedoms of others" (Advisory 2021-01 s.11(C)).
- **Recommendation:** add redact-on-export later. It is not an MVP legal requirement.

---

## 6. Retention and disposal

| Source | Rule | Label |
|---|---|---|
| RA s.11(e) | Keep only as long as needed for the purpose, legal claims, legitimate business purposes, "or as provided by law" | **Verified fact** [S1] |
| IRR s.19(d) | The same grounds. Legitimate business retention must be "consistent with standards followed by the applicable industry". Dispose "in a secure manner that would prevent further processing". | **Verified fact** [S2] |
| IRR s.19(e)(3) | "Personal data shall **not be retained in perpetuity** in contemplation of a possible future use yet to be determined" | **Verified fact** [S2] |
| IRR s.26(e)(5); 2023-06 s.9 | The **PIC** keeps a retention schedule or policy, reviews it, and informs data subjects | **Verified fact** [S2][S4] |
| 2023-06 ss.28–31 | Disposal policy; deletion or anonymisation techniques; **documentation before deletion**; secure wiping; a disposal provider follows the PIC's procedures | **Verified fact** [S4] |
| RA s.27 | Crime of improper disposal: knowingly or negligently abandoning personal information in a publicly accessible place or in trash. SPI: 1–3 years and ₱100,000–₱1M. | **Verified fact** [S1] |
| AO 2020-017 | At the end of a software contract, the form of disposal proof (for example, a certificate) is "best left with the parties". A waiver of claims does not erase DPA liability, and confidentiality survives a vendor's dissolution. | **Verified fact** [S18] |

**What "never auto-delete" means:**

- **During the subscription**, never auto-deleting is consistent with the law. Officetul is a PIP acting on the PIC's instructions, and **retention decisions belong to the PIC**. **Recommendation** (lawyer to confirm)
- The PIC still carries the s.11(e) and IRR s.19(e)(3) duty not to keep data forever. **Until retention rules ship**, the app should give PICs:
  - manual and bulk disposal
  - a disposal log showing who, what, when and on what basis
  - "blocked" and legal-hold states
  - documented purge from backups within a stated window

  **Recommendation**
- **At contract end**, IRR s.44(b)(7) requires delete **or** return "at the choice of the PIC", **including existing copies**. Officetul therefore cannot keep a lapsed tenant's documents indefinitely. **Verified fact** [S2]

  **Recommendation:**
  - a defined export window, for example 30–90 days in read-only or export-only mode
  - then deletion, including search indexes, OCR text and derivatives
  - backups expire within a stated period
  - a deletion certificate is issued
  - legally required vendor records, such as invoices, are kept separately

---

## 7. Cross-border transfer and hosting outside the Philippines

| Point | Detail | Label |
|---|---|---|
| No prohibition | "The DPA and its IRR do not prohibit the transmission and processing of personal data outside the country", including "cloud infrastructures" (AO 2017-012). The DPA "does not serve as a barrier" if safeguards give "comparable protection" (AO 2022-024). | **Verified fact** [S14][S20] |
| Accountability | The PIC uses "contractual or other reasonable means to provide a comparable level of protection" (RA s.21(a); IRR s.50) | **Verified fact** [S1][S2] |
| Contract duties | The contract must state the **geographic location of processing** (IRR s.44(a)). Transfers to another country happen only on documented instructions (s.44(b)(1)). Registration lists proposed transfers outside the Philippines (IRR s.47(a)(5)). | **Verified fact** [S2] |
| **NPC Advisory 2024-01** (30 May 2024), Model Contractual Clauses for Cross-Border Transfers | It **imposes nothing**. MCCs are "intended for voluntary adoption and do[] not impose any additional rights or obligations". The NPC "does not require" them and "will not accept any requests for review" of conformity. It points to the GPA comparison tables (ASEAN MCCs, EU SCCs, CoE, RIPD, Argentina, NZ, UK IDTA) and the ASEAN–EU Joint Guide. The extracted text numbers its sections 1, 2, 3, 6, 7, 8, so sections 4–5 appear to be missing. | **Verified fact** [S11] |
| ASEAN MCCs | Covered by NPC Advisory 2021-02 (title confirmed in the NPC list) | **Verified fact** (title) [S3] |
| Extraterritorial reach | The Act applies to PICs and PIPs outside the Philippines that use Philippine equipment or have a Philippine office (s.4) and in s.6 cases. A Philippine vendor hosting in Singapore remains fully bound. | **Verified fact** [S1] |
| Sector localisation flags | Government: DICT's Cloud First Policy (DC 2017-002 as amended by DC 2020-010) reportedly keeps government data under Philippine law and government control "regardless of location", with data-classification tiers. Banks and non-banks under BSP supervision: BSP Circular 1137 on outsourcing and IT risk management (amending Circulars 808 and 951) reportedly imposes due-diligence and outsourcing controls. Neither text was read. | **Secondary** [S24][S25]. **Open question** for any client that is a government contractor or BSP-supervised. |
| Government contractors | Contractors handling SPI of ≥1,000 people for a government agency must register their systems (RA s.24) | **Verified fact** [S1] |

**Recommendation (hosting).**

- Whichever region is chosen, name it in the DPA.
- Give advance notice before moving data or adding a region.
- Offer the ASEAN MCC module as an optional rider for clients that want it.
- Singapore hosting is legally permissible under the DPA. A Philippine data centre is not legally required for Officetul's target segments.
- Whether a Philippine data centre is a **sales advantage** is a separate, commercial question.

---

## 8. Penalties and enforcement relevant to a processor

| Exposure | Detail | Label |
|---|---|---|
| Crimes naming PIPs | s.31 (malicious disclosure) and s.32 (unauthorised disclosure) expressly cover "personal information controller **or personal information processor** or any of its officials, employees or agents". ss.25–29 apply to "persons" generally: unauthorised processing, negligent access, improper disposal, unauthorised purpose, intentional breach. Unauthorised processing of SPI: 3–6 years and ₱500,000–₱4M. | **Verified fact** [S1] |
| Officers | For a juridical person, the penalty falls on "responsible officers… who participated in, or by their gross negligence, allowed" the crime (s.34) | **Verified fact** [S1] |
| Large scale | Maximum penalty where ≥100 persons are affected (s.35) | **Verified fact** [S1] |
| Concealment | Concealing a breach involving SPI: 1.5–5 years and ₱500,000–₱1M (s.30) | **Verified fact** [S1] |
| Administrative fines (Circular 2022-01) | Applies to **PICs and PIPs**. **Grave:** 0.5–3% of prior-year **annual gross income**, for s.11 principles or s.16 rights infractions affecting >1,000 subjects, or any repeat infraction. **Major:** 0.25–2%, for the same with ≤1,000 subjects. The security, processor-oversight and notification infractions in the major tier are worded as "by a PIC". Cap: **₱5M per single act.** | **Verified fact** [S8] |
| Other liability | IRR s.51(a): anyone "involved in the processing" who fails to comply is liable. Circular 2023-06 s.35: PIC **or PIP** faces cease-and-desist orders, processing bans and fines. | **Verified fact** [S2][S4] |
| NPC fines against a PIP | No NPC decision fining a **processor** was found. The searches of the NPC advisory-opinion list and secondary sources returned only frameworks. | **Open question** |

---

## 9. Law firms

| Point | Detail | Label |
|---|---|---|
| Law firms are PICs | A law firm decides what client and third-party data it collects and why, which meets RA s.3(h). Circular 2022-04 adds that a sole practitioner is an "Individual Professional" and the "de facto DPO" (s.2(I), s.5(C)). | Rule: **Verified fact** [S1][S7]. Application to firms: **Recommendation** |
| Privileged information | Processing privileged information is prohibited unless an exception in s.13 applies, for example "all parties to the exchange" consent (s.13(a)) or legal claims (s.13(f)). A PIC "may invoke the principle of privileged communication" (s.15). AO 2018-017: it must be privileged under the Rules of Court and "relate to information about individuals". | **Verified fact** [S1][S16] |
| s.13(f) guidance | NPC Advisory 2024-02, "Personal Data Processing Based on Section 13(f)". Commentary says no pending case is needed. | Title: **Verified fact** [S3]. Content: **Secondary** |
| Evidence privilege | Rule 130 s.24(b) protects an attorney **"or other persons assisting the attorney"** from being examined about facts learned "in such capacity" | **Verified fact** [S22] |
| CPRA | s.27: a lawyer "shall maintain the confidences of the client, and shall respect data privacy laws". s.28: must not reveal confidences, "including data from the client's files", except on five grounds, the first being **written informed consent**. s.30 lists who inside the firm may receive matters (partners, staff and others), but **does not mention outside service providers**. | **Verified fact** (reproduction) [S21] |
| NPC opinions on law firms or cloud files | A keyword review of every advisory-opinion title in the NPC list (snapshot 2026-10-11) found **none on law firms**. On cloud, it found AO 2017-012 (offshore and cloud processing allowed) and AO 2020-018 (Gmail and OneDrive terms can serve as the outsourcing "legal act"). | **Verified fact** (absence, by title only) [S13][S14][S19] |
| Whether storing files with Officetul "reveals" confidences under CPRA s.28 | It is unsettled whether this needs client consent, or whether the vendor counts as a person "assisting the attorney" | **Open question** |

**Recommendation (law-firm features).**

- Per-matter restricted access and a "privileged" flag (see R08).
- Vendor support staff cannot open document content without a tenant-granted, time-limited, logged "support access" grant.
- Engagement-letter clause text that firms can adopt to disclose cloud storage.

---

## 10. Requirements table (mapping = **Recommendation**)

| Requirement | Who | Source | Documents-app feature or vendor process |
|---|---|---|---|
| Outsourcing contract with IRR s.44 contents | PIC (ensure), PIP (sign) | IRR ss.43–44; AO 2020-018 | Standard DPA addendum accepted at sign-up |
| Process only on documented instructions | PIP | IRR s.44(b)(1) | Tenant admin settings = instructions; no secondary use of content (no training of AI models on it) |
| Sub-processor control | PIP | IRR s.44(b)(4) | Published sub-processor list, advance notice of changes, right to object |
| Geographic location stated | PIP | IRR s.44(a) | Region named per tenant; change only with notice |
| Encryption at rest and in transit | PIC and PIP | IRR s.28(g); 16-03 s.7 | TLS everywhere; encrypted storage and backups; encrypted export bundles (2023-06 s.25) |
| MFA or strong authentication for SPI | PIC and PIP | 2023-06 s.16; FAQ VI | Tenant-enforceable MFA; mandatory MFA for vendor admins |
| Need-to-know access and clearances | PIC and PIP | 2023-06 s.13 | Role and folder permissions; privileged flag; vendor "support access" grants |
| Authorised software only | PIC and PIP | 2023-06 s.12 | No direct production database access outside approved, logged tools |
| Download, print and export controls | PIC and PIP | 2023-06 s.24 | Per-role permissions; watermarking optional |
| Audit logging and log retention | PIC and PIP | 2023-06 s.29; RA s.16(c)(7) | Immutable access and modification log per document and per person; security logs kept longer |
| Backup, restore and BCP tested | PIC and PIP | 2023-06 s.21; IRR s.28(e) | Daily backups, quarterly restore test record, stated RPO and RTO |
| Vulnerability management | PIC and PIP | RA s.20(c)(3); 2023-06 s.32; 16-03 s.6(D) | Scanning, patch SLAs, annual penetration test |
| Secure email intake | PIC and PIP | 2023-06 s.23 | Per-tenant inbox: TLS, SPF/DKIM/DMARC checks, sender allow-list, malware scan, quarantine |
| Breach: PIP informs PIC | PIP (by contract) | 16-03 ss.16–17; IRR s.38 | Contractual notice within 24 hours of awareness; incident details pack; affected-tenant scoping |
| Breach: notify NPC and subjects in 72 hours | PIC | RA s.20(f); 16-03 ss.15–18 | Per-tenant export of affected documents and data subjects to support notices |
| Incident register and annual report | PIC (PIP documents its own) | IRR s.41; 16-03 s.22 | Vendor incident register; tenant-facing incident history |
| PIA, inventory, records of processing | PIC and PIP | IRR s.26(c); 2023-06 ss.4–5, 7 | Vendor PIA; "PIA support pack" for clients |
| Registration and DPO | PIC and PIP | 2022-04 ss.5, 7, 8; 2023-06 s.4 | Vendor registers as PIP with its own DPO; "registration info pack" for clients |
| Data subject access and portability | PIC (PIP assists) | RA ss.16, 18; Adv. 2021-01 | Person search; access-log report; JSON/CSV and original-file export |
| Rectification with history | PIC | RA s.16(d) | Versioning that keeps retracted versions |
| Erasure or blocking vs legal retention | PIC | RA s.16(e); Adv. 2021-01 s.10; AO 2018-039 | Blocked state; legal hold overrides delete; denial-reason log |
| Retention policy and secure disposal | PIC | RA s.11(e); IRR s.19(d); 2023-06 ss.9, 28–30 | Interim bulk disposal plus disposal log; later retention rules; purge from derivatives and backups |
| Delete or return at end | PIP | IRR s.44(b)(7); AO 2020-017 | Export window, then certified deletion, including backups after N days |
| Audits | PIP | IRR s.44(b)(8); 2023-06 s.34 | Security whitepaper, certifications or reports; audit on notice |
| Staff confidentiality and training | PIC and PIP | RA s.20(e); IRR s.26(d); 2023-06 s.4(F) | Signed NDAs, an acceptable use policy (s.15), annual training |

---

## 11. Minimum contents of Officetul's outsourcing agreement / DPA addendum (**Recommendation**)

1. **Roles:** client = PIC, Officetul = PIP for tenant content. Officetul = PIC for account and billing data.
2. **Processing description** (IRR s.44(a)): purposes, document categories (IDs, tax, payroll, health, court records), categories of data subjects, duration = term plus exit period, **hosting region(s)**.
3. **Instructions:** process only per the agreement and tenant settings. No use of content for Officetul's own purposes. Flag unlawful instructions immediately (s.44(b)(9)).
4. **Confidentiality:** binding on staff and contractors, and surviving termination (RA s.20(e)).
5. **Security schedule:**
   - encryption in transit and at rest
   - MFA
   - access control and security clearances (2023-06 s.13)
   - logging and log retention
   - backups and restore testing
   - business continuity plan with RPO and RTO
   - vulnerability management
   - email-intake controls
   - staff training
6. **Sub-processors:** list (hosting, email intake, OCR or AI if any), flow-down of equal obligations, advance notice of changes, right to object or terminate (s.44(b)(4)).
7. **Cross-border:** named locations; no new country without notice; ASEAN MCC rider on request (Advisory 2024-01 is voluntary).
8. **Breach:** notify the PIC **without undue delay and within 24 hours** of awareness. Provide the 16-03 s.17(D) details as they become known. Cooperate, preserve logs (2023-06 s.29), and make no public statement about tenant data without the PIC's consent.
9. **Data subject rights:** tools and assistance within timelines that let the PIC meet 30 working days (Advisory 2021-01). Forward misdirected requests within 2 business days.
10. **Compliance assistance:** support PIAs, registration and NPC inquiries. Officetul is registered as a PIP and has a DPO.
11. **Audit:** information on request, third-party reports or certifications, on-site or remote audit on reasonable notice (s.44(b)(8)).
12. **Retention during term:** Officetul never deletes tenant content except on the PIC's instruction or at exit. The PIC owns retention decisions.
13. **Exit:** export window and format; deletion of all copies, including backups within N days; a deletion certificate; legally mandated vendor records excepted (s.44(b)(7)).
14. **Government and legal requests:** notify the PIC unless prohibited; disclose the minimum.
15. **Liability and indemnity** allocation, and survival clauses (AO 2020-017).
16. **Law-firm rider (optional):** privileged-matter handling, a ban on vendor content access without a grant, and acknowledgement of CPRA s.27–28 duties.

---

## 12. Questions for a Philippine lawyer

1. Is Officetul a PIP for all tenant content, including documents that arrive by email to the tenant inbox before any user opens them?
2. Can Officetul's click-through terms plus a DPA addendum satisfy IRR s.44, given AO 2020-018? Does IRR s.44(b)(4) ("prior instruction") allow a **general** authorisation of sub-processors with notice and objection?
3. Does Circular 2022-04 s.5(B) require Officetul to register as a PIP regardless of headcount? Which "data processing systems" must it list?
4. Does a PIP have its own duty to file the Annual Security Incident Report?
5. Since the 72-hour clock can start on the PIP's knowledge (IRR s.38(a)), what PIP-to-PIC notice period is defensible: 24 hours, or "immediately"?
6. Is "never auto-delete during the term" consistent with IRR s.19(e)(3) if the PIC has no retention tool yet? What interim tools are enough?
7. At exit, how long may Officetul keep backups containing tenant data? Does "storage authorised by law" cover anything for a vendor?
8. For law firms: does storing client files with a cloud PIP "reveal" confidences under CPRA s.28? Is client consent (s.28(a)) needed, or is the vendor a person "assisting the attorney" (Rule 130 s.24(b))? How does RA s.13(a) ("all parties to the exchange") apply?
9. Singapore versus a local data centre: any DPA, sector or client-contract reason to prefer local hosting? Do BSP Circular 1137 or DICT Cloud First rules reach Officetul when a client is BSP-supervised or a government contractor?
10. Are Officetul's officers exposed under RA s.34 for a tenant breach caused by a sub-processor?
11. Must Officetul offer redaction or person-level search to be "assisting" under IRR s.44(b)(5), or is export enough?
12. Any NPC decision so far that fines or orders a **processor** directly?

---

## 13. Sources (all accessed 2026-10-19)

| ID | Source | URL | Type |
|---|---|---|---|
| S1 | RA 10173, Data Privacy Act of 2012 | https://lawphil.net/statutes/repacts/ra2012/ra_10173_2012.html | Primary |
| S2 | IRR of RA 10173 (2016), Internet Archive copy of 2024-11-12 | https://web.archive.org/web/20241112161706/https://privacy.gov.ph/implementing-rules-regulations-data-privacy-act-2012/ | Primary (archive) |
| S3 | NPC Advisories & Circulars list, Internet Archive copy of 2026-10-12 | https://web.archive.org/web/20261012063023/https://privacy.gov.ph/pips-and-pics/advisories-circulars/ | Primary (archive) |
| S4 | NPC Circular 2023-06, Security of Personal Data | https://web.archive.org/web/20261012063023/https://privacy.gov.ph/wp-content/uploads/2024/03/NPC-Circular-Repeal-16-01-Signed.pdf | Primary (archive) |
| S5 | NPC FAQ on Circular 2023-06 | https://web.archive.org/web/20261012063023/https://privacy.gov.ph/wp-content/uploads/2024/12/v12-19-2024_FAQ-NPC-Circular-2023-06_NNJ_JDN.pdf | Primary (archive) |
| S6 | NPC Circular 16-03, Personal Data Breach Management | https://web.archive.org/web/20261012063023/https://privacy.gov.ph/wp-content/uploads/2022/01/sgd-npc-circular-16-03-personal-data-breach-management.pdf | Primary (archive) |
| S7 | NPC Circular 2022-04, Registration and DPO | https://web.archive.org/web/20261012063023/https://privacy.gov.ph/wp-content/uploads/2023/05/Circular-2022-04-2.pdf | Primary (archive) |
| S8 | NPC Circular 2022-01, Administrative Fines | https://web.archive.org/web/20261012063023/https://privacy.gov.ph/wp-content/uploads/2022/08/NPC-CIRCULAR-NO.-2022-01-GUIDELINES-ON-ADMINISTRATIVE-FINES-dated-08-AUGUST-2022-w-SGD.pdf | Primary (archive) |
| S9 | NPC Advisory 2017-01, Designation of DPOs | https://web.archive.org/web/20261012063023/https://privacy.gov.ph/wp-content/uploads/2022/01/NPC-Advisory-2017-01-sgd.pdf | Primary (archive) |
| S10 | NPC Advisory 2021-01, Data Subject Rights (DataGuidance mirror; NPC archive copy returned 404) | https://www.dataguidance.com/sites/default/files/npc-advisory-2021-01-final.pdf | Primary text (mirror) |
| S11 | NPC Advisory 2024-01, Model Contractual Clauses | https://web.archive.org/web/20261012063023/https://privacy.gov.ph/wp-content/uploads/2024/06/Published-NPC-Advisory-No.-2024-01-Contractual-Clauses-for-Cross-Border-Transfers_30May24.pdf | Primary (archive) |
| S12 | NPC Advisory 2025-01, DSA clarification | https://web.archive.org/web/20261012063023/https://privacy.gov.ph/wp-content/uploads/2025/07/SGD-2025-01-DSA-Clarification.pdf | Primary (archive) |
| S13 | NPC Advisory Opinions list, Internet Archive copy of 2026-10-11 | https://web.archive.org/web/20261011080925/https://privacy.gov.ph/pips-and-pics/advisory-opinions/ | Primary (archive) |
| S14 | NPC AO 2017-012, Processing outside the Philippines | https://web.archive.org/web/20261011080925/https://privacy.gov.ph/wp-content/uploads/2022/01/NPC_AdvisoryOpinionNo._2017-012.pdf | Primary (archive) |
| S15 | NPC AO 2017-015, Subcontracting or outsourcing | https://web.archive.org/web/20261011080925/https://privacy.gov.ph/wp-content/uploads/2022/01/NPC_AdvisoryOpinionNo._2017-015.pdf | Primary (archive) |
| S16 | NPC AO 2018-017, Trade secrets / privileged communication | https://web.archive.org/web/20261011080925/https://privacy.gov.ph/wp-content/uploads/2022/01/AdOpNo.2018-017.pdf | Primary (archive) |
| S17 | NPC AO 2018-039, Erasure vs retention | https://web.archive.org/web/20261011080925/https://privacy.gov.ph/wp-content/uploads/2022/01/AONo_2018-039.pdf | Primary (archive) |
| S18 | NPC AO 2020-017, Termination of services / dissolution | https://web.archive.org/web/20261011080925/https://privacy.gov.ph/wp-content/uploads/2022/01/Redacted-Advisory-Opinion-No.-2020-017.pdf | Primary (archive) |
| S19 | NPC AO 2020-018, Outsourcing (cloud terms as legal act) | https://web.archive.org/web/20261011080925/https://privacy.gov.ph/wp-content/uploads/2022/01/Redacted-Advisory-Opinion-No.-2020-018.pdf | Primary (archive) |
| S20 | NPC AO 2022-024, Free flow of data | https://web.archive.org/web/20261011080925/https://privacy.gov.ph/wp-content/uploads/2023/09/Advisory-Opinion-No.-2022-024-sgd_Redacted.pdf | Primary (archive) |
| S21 | CPRA, A.M. No. 22-09-01-SC (ChanRobles reproduction) | https://chanrobles.com/rulesofcourt/2023codeofprofessionalresponsibility.php | Primary (reproduction) |
| S22 | 2019 Amendments to the Revised Rules on Evidence, A.M. No. 19-08-15-SC | https://lawphil.net/courts/rules/am_19-08-15-sc_2019.html | Primary |
| S23 | Baker McKenzie, "Philippines: Data Breach Notification Procedure Further Clarified" (Jun 2026) | https://www.bakermckenzie.com/en/insight/publications/2026/06/philippines-data-breach-notification-procedure-further-clarified | Secondary |
| S24 | W.Media, "Philippine Government Pushes Cloud First Policy" (search excerpt only) | https://w.media/philippine-government-pushes-cloud-first-policy/ | Secondary (excerpt) |
| S25 | Outsource Accelerator, "BSP amends IT outsourcing risk management rules" (search excerpt only) | https://news.outsourceaccelerator.com/bsp-amends-it-outsourcing-risk-management-rules/ | Secondary (excerpt) |
