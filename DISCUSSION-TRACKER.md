# Discussion Tracker

Status of each section. A section closes only after owner/partner review, never because a file exists.

**Statuses:** Not started → Researching → Draft ready → In discussion → Revising → Closed (or Waived by owner)

| # | Section | Phase | Status | Research doc | Meeting guide | Last update | Blocking decisions |
|---|---|---|---|---|---|---|---|
| 00 | Setup and workflow | 0 | Draft ready | [MASTER-PLAN](MASTER-PLAN.md) | — | 2026-10-18 | Owner confirms the working method |
| 01 | Business problem and opportunity | 1 | Awaiting senior partner sign-off | [01](sections/01-business-problem-and-opportunity.md) | [01](meeting-guides/01-business-problem-and-opportunity.md) | 2026-10-18 | Grilling Round 1 |
| 02 | ERP-embedded document apps and module pricing | 2 | Owner agreed; pricing deferred to 18 | [02](sections/02-market-and-competitor-research.md) | [02](meeting-guides/02-erp-document-apps-and-pricing.md) | 2026-10-18 | Grilling Round 5 |
| 03 | Document types and workflows (re-scoped) | 3 | Owner agreed | [03](sections/03-philippine-sme-workflows.md) | [03](meeting-guides/03-document-types.md) | 2026-10-18 | Grilling Round 6 |
| 04 | ISO certification strategy | 4 | **Future goal; senior partner decides; not blocking (D-028)** | [04](sections/04-iso-certification-strategy.md) | [04](meeting-guides/04-iso-certification.md) | 2026-10-19 | Senior partner: goal and timing |
| 05 | Document lifecycle and workflows | 6 | Owner agreed (combined with 08) | [05+08](sections/05-document-lifecycle-and-workflows.md) | — | 2026-10-20 | Grilling Round 8 |
| 06 | Upload and file-format support | 7 | Owner agreed | [06](sections/06-upload-and-file-format-support.md) | — | 2026-10-20 | Grilling Round 9 |
| 07 | OCR and document understanding | 7 | Owner agreed; benchmark pending | [07](sections/07-ocr-and-document-understanding.md) | — | 2026-10-20 | Grilling Round 9 |
| 08 | Versioning and approval | 6 | Owner agreed (combined with 05) | [05+08](sections/05-document-lifecycle-and-workflows.md) | — | 2026-10-20 | Grilling Round 8 |
| 09 | Activity log and integrity | 8 | Owner agreed; lawyer review of retention pending | [09](sections/09-audit-trails-and-integrity.md) | — | 2026-10-20 | Grilling Round 11 |
| 10 | Authenticity and certified true copies | 5 | Owner agreed; lawyer review pending | [10](sections/10-authenticity-and-certified-true-copies.md) | [10–11](meeting-guides/10-11-legal-authenticity-and-privacy.md) | 2026-10-19 | Section 03 |
| 11 | Philippine legal and regulatory research | 5 | Owner agreed; lawyer and tax review pending | [11](sections/11-philippine-legal-and-regulatory-research.md) | [10–11](meeting-guides/10-11-legal-authenticity-and-privacy.md) | 2026-10-19 | Section 03 |
| 12 | Security and antivirus | 7–8 | Owner agreed (file pipeline; access and isolation) | [12](sections/12-security-and-antivirus.md) | — | 2026-10-20 | Grilling Rounds 9, 11 |
| 13 | Database and search architecture | 9 | Owner agreed; code review pending (D-013) | [13](sections/13-database-and-search-architecture.md) | — | 2026-10-20 | Grilling Round 12 |
| 14 | Storage and server requirements | 10 | Owner agreed; Officetul hosting (Q12) pending | [14](sections/14-storage-and-server-requirements.md) | — | 2026-10-21 | Grilling Round 13 |
| 15 | Backup, retention, DR | 10 | Owner agreed; lawyer A23–A30 pending | [15](sections/15-backup-retention-and-disaster-recovery.md) | — | 2026-10-21 | Grilling Round 13 |
| 16 | ERP integration and data model | 9 | Owner agreed; code review pending (D-013) | [16](sections/16-erp-integration-and-data-model.md) | — | 2026-10-20 | Grilling Round 12 |
| 17 | MVP scope and prioritisation | 11 | Not started | — | — | — | Sections 01–16 |
| 18 | Cost estimates and operating model | 10 | Owner agreed; **pricing awaits senior partner** | [18](sections/18-cost-estimates-and-operating-model.md) | [18](meeting-guides/18-costs-and-pricing.md) | 2026-10-21 | Senior partner pricing meeting |
| 19 | Risk register | all | Not started | — | — | — | Living document, started in Phase 1 |
| 20 | Architecture proposal | 12 | Not started | — | — | — | Section 17 |
| 21 | Implementation roadmap | 12–13 | Not started | — | — | — | Section 20 |

## Grilling rounds

| Round | Date | Topic | Questions | Status |
|---|---|---|---|---|
| 1 | 2026-10-18 | Who, what problem, why ISO, constraints | Q1–Q7 | Answered: D-001 to D-006; Q3 and Q4 carried to Round 2 |
| 2 | 2026-10-18 | What "digitise" means, scanning model, paper originals, ISO follow-up, ERP facts | Q1–Q7 | Answered: D-007 to D-009; Q2, Q3 and Q5 carried to Round 3 |
| 3 | 2026-10-18 | Native build, MVP shape, scanner capture, ISO design stance, ERP questionnaire | Q1–Q7 | Answered: D-010 to D-013; Q2, Q3 and Q4 explained and carried to Round 4 |
| 4 | 2026-10-18 | MVP scope (explained), capture channels (explained), hardware, pilot | Q1–Q4 | Answered: D-014 to D-017. **Section 01 frontier empty.** |
| 5 | 2026-10-18 | Pricing, links, permissions, MVP additions, document requests | Q1–Q6 | Answered: D-019 to D-022; pricing deferred to Section 18 (D-018) |
| 6 | 2026-10-19 | OR preset change, preset list and packs, common fields, professional consultation | Q1–Q5 | Answered: D-023 to D-027 |
| 7 | 2026-10-20 | Claims policy, certified copies, evidence features, processor duties, privacy features, professional brief | Q1–Q6 | Answered: D-029 to D-034 |
| 8 | 2026-10-20 | Terms, filing rule, official marking, version states, concurrency, amendments, deletion, duplicates, details history | Q1–Q9 | Answered: D-035 to D-043 |
| 9 | 2026-10-20 | File types, size limit, uploads, virus scanning, encrypted files, OCR engine, OCR benchmark, tool isolation | Q1–Q8 | Answered: D-044 to D-051 |
| 10 | 2026-10-20 | Repository housekeeping: move files to the root, commit plan, public repository | — | Done; no product decisions |
| 11 | 2026-10-20 | Company isolation, folder permissions, downloads, Confidential folders, no hints, Support access, login, encryption, two logs, log protection and fixity, log retention, time and log writer | Q1–Q12 | Answered ("Go ahead please" to all recommendations): D-052 to D-063 |
| 12 | 2026-10-20 | Search engine and triggers, search coverage, identifiers, search privacy, search test, deleted records, shared stored copy, details, expenses report rule, file protection, database version, technical defaults | Q1–Q12 | Answered ("go ahead" to all recommendations): D-064 to D-075 |
| 13 | 2026-10-21 | Hosting, processing server, MySQL upgrade, recovery targets, backup location, backup retention, recovering deletions, exit, purge ledger and restore marker, restore tests, plan limits, ops budget, cost model to senior partner | Q1–Q13 | Answered ("go ahead please" to all recommendations): D-076 to D-088 |

## Change log for sequence

| Date | Change | Reason |
|---|---|---|
| 2026-10-18 | **Phase 2 re-scoped** from "should we build at all?" to "how do ERP-embedded document apps work and what do they cost?" (e.g. Odoo Documents, ERPNext/Frappe), plus how to price the module. | D-002 and D-010 settled build-as-ERP-module. |
| 2026-10-18 | **Phase 3 re-scoped** from customer interviews to desk research plus the owner's client document profile, validated by the pilot. | D-004, D-009, D-012: interviews not feasible. |
| 2026-10-18 | **Phase 9 split:** design against the Officetul questionnaire answers first; review against the real code before any build. | Repository access only after design (D-013). |
