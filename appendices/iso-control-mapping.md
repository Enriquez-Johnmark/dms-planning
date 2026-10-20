# ISO Control Mapping (certification-support traceability)

**Purpose:** show which ISO requirements the Documents app can support with evidence, and what still needs people, policy or other systems. It references clause and control **titles only**; no standard text is reproduced. Titles are from public sources cited in [R03](../research/R03-iso-standards-primer.md) §5.

**Scope assumed:** ISO/IEC 27001:2022 for the company, covering Officetul's development, hosting and support (Section 04 recommendation, not yet decided). Rows for ISO 9001:2026 apply only if goal A is chosen. ISO 9001:2026 subclause numbering was not verified, so those rows cite the topic.

**Ownership:** ISO certification is a future goal; the senior partner decides the standard and timing (D-028). This mapping is reference material for that decision and for building ISO-ready controls (D-011). It is not a project gate.

**Status values:** Unverified → Planned → Implemented → Validated.

## ISO/IEC 27001:2022, management-system clauses

| Requirement (title) | Applicability | Business process | Documents-app capability | Non-software control | Evidence | Gap | Owner | Status |
|---|---|---|---|---|---|---|---|---|
| 4 Context of the organization (incl. scope) | Applies | Management | Hosts the scope statement | Leadership defines scope, interested parties, climate consideration (Amd 1) | Approved scope document | Scope and legal entity not defined | Senior partner | Unverified |
| 5 Leadership (incl. policy, roles) | Applies | Management | Hosts the policy with approval record (next release) | Management commitment; roles assigned | Signed policy; role assignments | No ISMS roles yet | Senior partner | Unverified |
| 6 Planning (risk assessment and treatment, SoA, objectives) | Applies | Risk management | Stores the risk register and Statement of Applicability; versions | **Risk decisions are made by people** | Risk assessment, treatment plan, SoA | Not started | Owner + senior partner | Unverified |
| 7.2 / 7.3 Competence and awareness | Applies | HR, training | Stores training records (Employee Document type) | Training programme | Training records | — | Owner | Unverified |
| **7.5 Documented information** | Applies | Document control | **Core fit:** identification, versions, official-version marker, access control, approvals (next release), retention (next release) | Document-control procedure | Version history; approval records; access log | Approvals and retention not in MVP | Owner | Planned |
| 8 Operation | Applies | All in scope | Holds operating procedures | Processes actually run | Records of operation | — | Owner | Unverified |
| 9.1 Monitoring and measurement | Applies | Management | Stores metrics reports | Metrics chosen and reviewed | Reports | — | Owner | Unverified |
| 9.2 Internal audit | Applies | Audit | Stores the audit programme and reports | **Independent internal auditor** | Audit reports | No auditor | Senior partner | Unverified |
| 9.3 Management review | Applies | Management | Stores minutes | **Management meeting** | Minutes, decisions | — | Senior partner | Unverified |
| 10 Improvement (nonconformity, corrective action) | Applies | Quality | Stores corrective action records | Corrective action process | Records | — | Owner | Unverified |

## ISO/IEC 27001:2022 Annex A, controls the app directly supports

| Control (title) | Documents-app capability | Non-software control | Evidence | Gap / dependency | Status |
|---|---|---|---|---|---|
| A.5.9 Inventory of information and other associated assets | Document and folder inventory per tenant | Asset register | Inventory report | — | Planned |
| A.5.12 Classification of information | Sensitivity field with defaults per type (D-024, D-026) | Classification policy | Type configuration; labelled documents | — | Planned |
| A.5.13 Labelling of information | Visible sensitivity label in UI and exports | — | Screenshots, exports | Export labelling not yet designed | Unverified |
| A.5.14 Information transfer | Scan-to-email inbox with TLS and approved senders (D-015); share links (next release) | Transfer policy | Configuration; logs | Share links later | Planned |
| A.5.15 Access control | Folder permissions plus opt-in link visibility (D-020) | Access-control policy | Permission configuration | Designed (D-053 to D-055) | Planned |
| A.5.18 Access rights | Grant/revoke logged in the activity log | Periodic access review | Review records | Access-review report not yet scoped | Unverified |
| A.5.33 Protection of records | Original kept plus hash (D-026); legal hold and retention (next release) | Records policy | Hash verification report | Retention later | Planned |
| A.5.34 Privacy and protection of PII | SPI flag and restricted defaults | DPA compliance programme (Section 11) | Configuration; outsourcing agreement | R10 pending | Planned |
| A.5.37 Documented operating procedures | Hosts SOPs with versions | — | SOP library | — | Planned |
| A.8.3 Information access restriction | Permission-safe search, snippets and previews (D-026) | — | Tests | Designed (D-052, D-056) | Planned |
| A.8.5 Secure authentication | Uses Officetul login; MFA | — | Configuration | Depends on Officetul (questionnaire Q7) | Unverified |
| A.8.10 Information deletion | Soft delete; controlled disposal with evidence (next release) | Disposal procedure | Disposal records | Later | Unverified |
| A.8.12 Data leakage prevention | Download and share restrictions per sensitivity | — | Configuration | Not yet designed | Unverified |
| A.8.13 Information backup | Backup of files and database; restore tests (Section 15) | Backup procedure | Restore-test records | Phase 10 | Unverified |
| A.8.15 Logging | Activity log, protected (Section 09) | Log review | Log samples | Designed (D-060 to D-062) | Planned |
| A.8.24 Use of cryptography | Encryption in transit and at rest | Key management | Configuration | Phase 9/10 | Unverified |

## ISO 9001:2026 (only if goal A is chosen)

| Topic | Documents-app capability | Non-software control | Status |
|---|---|---|---|
| Documented information (create, update, control) | Same as 27001 7.5 | Quality manual / procedures | Unverified |
| Control of externally provided documents | Supplier documents linked to supplier records (D-019) | Supplier management | Unverified |

## Product-feature checklists (not certification)

- **ISO 15489-1:2016** (records concepts) and **ISO 16175-1:2020 / ISO/TS 16175-2:2020** (functional requirements for records software): use as a checklist when designing retention, disposal and metadata in the next release.
- **ISO 19005 (PDF/A):** export format for long-term preservation and court e-filing (law pack). Validate with veraPDF.
