# 16 — ERP Integration and Data Model

## 1. Purpose

Define **what Officetul Documents stores** (its tables, keys and rules), **how other Officetul apps connect to it** (Links, record panels, permissions, events), and **how stored files and database rows stay consistent**. Uses the agreed terms in [GLOSSARY.md](../GLOSSARY.md).

## 2. Status

**Owner agreed (Round 12, D-069 to D-073, D-075), 2026-10-20.** Research: [R17](../research/R17-data-model-and-erp-integration.md) (2026-10-20). The design is based on assumptions, because the Officetul code has not been seen; it is checked against the code before any build (D-013).

## 3. Executive Summary

- **Documents is a native Officetul app** (D-002, D-010). It shares Officetul's database, users, roles and company model. Other apps reach it only through one small published contract.
- **The database itself blocks the worst mistakes:**
  - a record can't point at another company's row (composite company keys);
  - a Document can't have two Official versions (a unique index);
  - a Version's content can't be changed (a write-once rule with a trigger backstop). *(D-075)*
- **Links survive.** When an Officetul record is deleted, its Documents stay, and the Link is marked "record deleted". *(D-069)*
- **One stored copy per company per file.** Duplicates filed on purpose share one copy, which is never shared across companies. *(D-070)*
- **Standard details are real fields; type-specific details are flexible** and filter by exact value in the MVP. *(D-071)*
- **The expenses report trusts only Official, non-Trash Documents.** *(D-072)*
- **Stored files can be recovered from accidents but can still be deleted on purpose:** storage versioning, no Object Lock on originals. *(D-073)*

## 4. Business Context

The selling point is the one-stop ERP: a receipt lives with its expense, and a contract with its customer and project (D-019). That only works if:
- every Officetul app can link records to Documents the same way;
- Links don't break or leak when records change;
- the data model is strict enough that a small team can't accidentally mix two companies' data.

## 5. Verified Research Findings

| Finding | Source |
|---|---|
| Odoo stores attachments as `res_model` + `res_id`; deleting a record moves its Documents to Trash | R17 §3 |
| Frappe stores `attached_to_doctype` + `attached_to_name`; it blocks deleting a record with links, and otherwise deletes the attachments | R17 §3 |
| Odoo and Frappe add a real database column per custom field; a shared multi-company table can't do that per company | R17 §2 |
| Laravel polymorphic relations can't have real foreign keys; `enforceMorphMap` stops class names being stored | R17 §3 |
| Laravel model events don't fire on mass updates and deletes | R17 §4 |
| S3 lifecycle rules filter by leading key prefix, and the clean-up rules can't use tags, so the storage class must come first in the key | R17 §5.1 |
| S3 versioning keeps deleted or overwritten objects recoverable; compliance-mode Object Lock can't be deleted even by the root user | R17 §5.1 |
| MySQL 8.4 needs `UNIQUE (tenant_id, id)` before composite foreign keys can point at it | R17 §2 |

## 6. Technical Analysis

### 6.1 Main tables *(D-075; R17 §2)*

| Glossary term | Table | Key rules |
|---|---|---|
| Inbox item / Capture record | `inbox_items`, `capture_records` | Email headers kept in storage; review state |
| Document | `documents` | Company, Folder, Document type, standard detail fields, `details` JSON, Trash marker |
| Version | `versions` | Write-once content; state (Draft, Official, Superseded, Withdrawn); at most one Official per Document; Revision label |
| Original file | `original_files` | SHA-256; one per company per fingerprint (D-070); storage version ID |
| Derived file | `derived_files` | Kind and tool version; regenerable |
| Paper original | `paper_originals` | Location and custodian (they change, so not on the Version) |
| Folder / access | `folders`, `folder_grants` | Parent pointer plus `acl_folder_id`; Restricted and Confidential flags (D-053, D-055) |
| Link | `linked_records`, `document_links`, `link_types` | One stub per Officetul record; "Grants view" off by default; "record deleted" flag (D-069) |
| Related document | `document_relations` | Labelled, e.g. "amends" (D-040) |
| Activity log | `activity_events`, payloads, seals | As Section 09 |
| Fixity check | `fixity_checks` | Insert-only results (D-061) |

Every table carries `tenant_id` and a ULID key. Child tables point at parents with `(tenant_id, parent_id)` foreign keys.

### 6.2 Details *(D-071)*

| Kind | Examples | Filtering in the MVP |
|---|---|---|
| Standard (typed columns) | Document date, reference number, amount, expiry date, event date | Full: ranges, sorting |
| Type-specific (JSON, validated per Document type) | Payee TIN, ATC, permit number, bond amount | Exact value only (multi-valued index); ranges later |

### 6.3 The contract with other Officetul apps *(D-075; R17 §4)*

Each Officetul app that has records worth linking (Accounting, Projects, CRM…) registers a **linkable record provider** that answers:
1. **What is this record called?** (e.g. "Expense #881 · Meralco")
2. **Can this user open it?** (needed for link visibility, D-053)
3. **Which of these records can this user see?** (a database subquery, for lists and search)
4. **Which Link types does it offer?** (e.g. "receipt for", "invoice for"; "Grants view" always starts off)

Events then run both ways:

| Event | Effect |
|---|---|
| A record is deleted | Its Links are marked "record deleted"; Documents stay (D-069) |
| A Document's Official version is withdrawn | The owning app is told; the expenses report shows the expense as missing again (D-072) |
| A record is renamed | The Link's label snapshot is refreshed |

A **nightly consistency check** catches mass deletes that skip events. Record pages show a **Documents panel**: attach, see linked Documents, with the same "no hints" rule (D-056). Attaching from a record and the later document-request feature (D-022) share one "filing intent" mechanism.

### 6.4 Storage layout and consistency *(D-073, D-075; R17 §5)*

| Storage area | Versioning | Lock | Clean-up |
|---|---|---|---|
| Quarantine | Off | None | Abandoned uploads after 1–7 days; infected files after 30 days (D-047) |
| Originals | **On** (30–90-day recovery window) | None (D-073) | Old versions expire after the window |
| Derived | Off | None | Deleted with the Original file |
| Exports | Off | None | Expire after 7 days |
| Log digests | On | Compliance | Kept 10 years (D-062) |

**Rules:**
- Keys hold only random IDs: no file names, no fingerprints, no company names.
- **Object before row, row before delete.** A file is stored before its database row is written. Its bytes are deleted only after the commit that removes the last row pointing at them.
- A daily sweep finds stored files with no row and raises an alert.

### 6.5 Expenses report rule *(D-072)*

An expense counts as covered when it has a Link to a Document of the right type (Invoice or Payment Receipt, D-023) that:
- has an **Official version**;
- is **not in Trash**;
- is visible to the viewer.

Only roles that can see all finance Folders get the report (D-056).

## 7. Alternatives and Trade-offs

| Question | Options | Choice |
|---|---|---|
| Record deleted | Trash the Documents (Odoo) / block the delete (Frappe) / **keep and flag** | Keep and flag: real bills must not vanish, and duplicates must still be fixable |
| Linking | Bare polymorphic columns / **registry of linked records with real foreign keys** | Registry: database-checked, survives deletion, no class names stored |
| Custom details | A column per field / attribute rows / **JSON validated per type** | JSON: works in a shared multi-company table; exact filters now, ranges later |
| Duplicate bytes | Copy per Document / **one per company** / one across companies | One per company: saves space without revealing that two companies hold the same file |
| Original file protection | **Versioning only** / Object Lock | Versioning: lawful deletion stays possible; lock with legal hold comes with retention |
| Folder tree | Closure table / nested set / **parent pointer + access shortcut** | Parent pointer: simplest, and recursive queries cover the rest |

## 8. Recommendation

Adopt §6 as the MVP data model and integration contract. Confirm it in the D-013 code review before building, especially the key types, the module structure, how records are deleted, and how record access is decided.

## 9. Cost and Operational Implications

- **Size (Estimate):** at 100 companies × 50,000 Documents, about 35 GB of core tables and about 9.5 TB of originals plus about 4 TB of derived files. In year 1 it is a small fraction of that.
- **Versioning** keeps deleted or overwritten originals for the 30–90-day window, costing storage for those extra copies (Phase 10 sizing).
- **Each Officetul app** that wants Links must implement the provider: a small piece of work per app, owned by that app's maintainer.

## 10. Risks and Limitations

| ID | Risk | Mitigation |
|---|---|---|
| RK-51 | **Officetul's key types or module layout** differ from the assumptions | Design review before build (D-013); questionnaire Q25–Q30 |
| RK-52 | **Turning on the morph map** affects every Officetul app that already stores class names | Check existing morph columns; migrate them in one planned step |
| RK-53 | **Mass deletes in other apps** skip events and leave Links unflagged | Nightly consistency check |
| RK-54 | **"Permanently deleted" file still exists** because another Document shares it | Certified deletion (next release) reports shared copies |
| RK-55 | **Link labels name people** (e.g. "Expense #881 · Juan dela Cruz") and outlive an erasure request | Privacy review; blank the label on erasure (OQ-029) |

## 11–12. Questions and Decisions

Grilling Round 12, Q6–Q10 and Q12.

## 13. Confirmed Decisions

| ID | Decision |
|---|---|
| D-069 | Deleted records: Documents stay; Link marked "record deleted" |
| D-070 | One stored copy per company per file; never across companies |
| D-071 | Standard details typed; type-specific details filter by exact value in the MVP |
| D-072 | Expenses report counts only Official, non-Trash Documents |
| D-073 | Storage versioning for originals; no Object Lock on originals |
| D-075 | Technical defaults (shared with Section 13) |

## 14. Open Questions

- OQ-025: how Officetul decides who may open a record.
- OQ-028: MySQL version and upgrade effort.
- OQ-029: Link labels that name people, and erasure.
- Officetul code checks (R17 §11), now in the [ERP technical questionnaire](../questionnaires/erp-technical-questionnaire.md) Q25–Q30:
  - key types;
  - module structure;
  - existing attachments;
  - morph columns;
  - how records are deleted;
  - whether there is an Expense model.
- Does a local data-centre store (if chosen in Phase 10) support versioning and lifecycle rules like S3?

## 15. Dependencies

Depends on D-019 to D-023, D-035 to D-043, D-046, D-047, D-052 to D-061. Feeds Section 14 (storage sizing), Section 15 (backup and restore), Section 17 (MVP scope) and Section 20 (architecture).

## 16. Sources

R17 (50 sources, accessed 2026-10-20), especially the MySQL 8.4 reference manual, the Laravel 13.x documentation, the Odoo and Frappe documentation and source, and the AWS S3 documentation.

## 17. Next Steps

- [x] Link behaviour on record deletion agreed (D-069)
- [x] Stored-copy, details and report rules agreed (D-070 to D-072)
- [x] File protection agreed (D-073)
- [x] Technical defaults agreed (D-075)
- [ ] Questionnaire answers Q25–Q30
- [ ] Design review against the Officetul code (D-013)
