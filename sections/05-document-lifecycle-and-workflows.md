# 05 + 08 — Document Lifecycle, Versions and Official Marking

Covers Section 05 (lifecycle and workflows) and Section 08 (versioning and approval). They are combined because they describe the same model.

## 1. Purpose

Define **what a document is, how it changes, and which version counts**, so that staff never act on the wrong version and every change can be traced. This is a business-level model. Tables and code come in Phase 9 and later.

## 2. Status

**Owner agreed (Round 8, D-035 to D-043), 2026-10-20.** No external research was needed beyond earlier notes: SharePoint and Odoo version behaviour in [R02](../research/R02-dms-alternatives-landscape.md) §3 and [R05](../research/R05-erp-embedded-document-apps.md) §6, evidence rules in [R09](../research/R09-electronic-records-evidence-and-certified-copies.md) §6, and drawing revision practice in [R08](../research/R08-doc-types-law-and-construction.md) §3. Terms are recorded in [GLOSSARY.md](../GLOSSARY.md).

**Decisions already in force:**
- D-014: an explicit official version; approval chains in the next release.
- D-019: many-to-many links.
- D-020: folder permissions.
- D-026: original plus hash; revision label and status kept separate from the official marker.
- D-031: capture record; derived files.

## 3. Executive Summary

- **Separate the business "document" from its "versions" and the stored "files".** A lease contract is one document. Its drafts and signed copy are versions. Each version keeps its original file unchanged.
- **"Latest" is not "official".** Each document has **at most one official version**, chosen explicitly by a person with permission. Uploading a new version never replaces the official one automatically.
- **Filing is an explicit choice.** When a scan is filed from the Inbox, the user chooses **Official record** or **Draft**, with a default per document type. A scanned receipt defaults to Official; a contract being negotiated defaults to Draft.
- **Four version states:** Draft → Official → Superseded, plus **Withdrawn** for an official version marked in error. Expiry is a badge, not a state.
- **No locking in the first version.** Two people can each upload a draft. The system warns when a draft was based on an older version.
- **An amendment is a separate document** linked to the original ("amends"), not a new version, because both stay in force.
- **Nothing is ever purged automatically.** Deleted items go to Trash, and only a company admin can delete permanently, with a reason that is logged.

## 4. Business Context

The worst outcomes this model prevents:
- a site team building from a superseded drawing;
- a bookkeeper claiming VAT on the wrong invoice copy;
- a lawyer filing an unsigned draft as the executed contract;
- nobody able to say who changed what.

The master plan requires that "a newly uploaded file must not automatically become the authoritative version".

## 5. Findings from Earlier Research

| Finding | Source |
|---|---|
| SharePoint separates minor (draft) and major (published) versions, with optional check-out and content approval | R02 §3.2 |
| Odoo keeps versions in upload order, with a lock, but has **no official marker** | R05 §1, §6 |
| Construction practice: revision label (A, B, 0, 1…) plus issue status (For Information, For Approval, For Construction, As-Built, Superseded) | R08 §3.3 |
| A court weighs which counterpart was "intended to have the same effect" (Rule 130 s.4(a)); log who marked the official version, when and why | R09 §6 |
| Zoho permanently deletes cleared inbox items, a pitfall to avoid | R05 §9 |

## 6. Technical Analysis (business-level model)

### 6.1 Proposed terms

| Term | Meaning | Example |
|---|---|---|
| **Inbox item** | Something captured (upload, email, phone) and not yet filed. It keeps its capture record. | A scan emailed from the office printer at 9:14 |
| **Document** | The business identity of a record: type, folder, details, links. It holds one or more versions. | "Lease contract with ABC Realty, Unit 3B" |
| **Version** | One numbered state of a document's content (v1, v2, …), with its own original file. It is never edited after creation. | v1 draft from the lawyer; v2 signed scan |
| **Original file** | The exact bytes received, with a fingerprint (hash). Never modified. | `lease-signed.pdf`, SHA-256 … |
| **Derived file** | Something the system makes from an original: OCR text, preview, thumbnail. | Searchable text layer |
| **Official version** | The one version of a document that the company has marked as the one to rely on. At most one per document. | v2, marked official by the office manager on 3 Nov |
| **Revision label** | The company's own label for a version, free text. | "Rev C", "Final", "Signed" |
| **Issue status** | The company's own status pick-list for a document type. Not the same as the version state. | "For Construction" |
| **Link** | A labelled connection between a document and an Officetul record (D-019) | "receipt for" Expense #881 |
| **Related document** | A labelled connection between two documents | Amendment No. 1 "amends" the lease |

**Version** is the system concept; **revision label** is the company's own text, to avoid clashing with construction "revisions".

### 6.2 Version states

```text
           mark official                 another version marked official
  Draft ──────────────────▶ Official ─────────────────────────────────▶ Superseded
    │                          │                                            │
    │ discard (to Trash)       │ withdraw (reason required)                 │ restore as official
    ▼                          ▼                                            ▼
  Trash                    Withdrawn                                     Official
```

- **At most one Official** per document. A document may have none, e.g. a contract still in negotiation.
- **Withdrawn** means "this was marked official by mistake or is no longer valid". A reason is required. The document then has no official version until someone marks one.
- **Restore:** a superseded version can be made official again. This is logged and no file is copied.
- **Expired** (past the expiry date) is a badge shown on the document. It is not a version state.

### 6.3 Key scenarios

| # | Scenario | What happens |
|---|---|---|
| 1 | **File a new scan** | From the Inbox, the user picks type, folder and details, and **chooses Official record or Draft** (default per type). The document is created with v1. The Inbox item is marked processed, not deleted. |
| 2 | **Attach from Officetul** (e.g. on an expense) | The document is created in the type's default folder, linked "receipt for" the expense, as an Official record. |
| 3 | **Upload a corrected copy** | It becomes v2, **Draft**. v1 stays official until someone with permission marks v2 official; v1 then becomes Superseded. |
| 4 | **Two people upload drafts** | Both exist, each showing "based on v1". If v2 is made official, v3 shows the warning "based on v1, but v2 is now official". |
| 5 | **Draft rejected** | Discard it (to Trash) or leave it as Draft. Formal rejection comes with approval chains (next release). |
| 6 | **Change after it's official** | Files are never edited. Any change is a new version. Details such as the amount or a date are edited on the document, with the old and new values logged. |
| 7 | **Marked official by mistake** | Withdraw it with a reason, then restore the correct version as official. |
| 8 | **Contract amendment** | A new document of type Contract, related to the original as "amends". Both stay in force. |
| 9 | **Construction drawing Rev C replaces Rev B** | New version, revision label "Rev C", issue status "For Construction". When marked official, Rev B becomes Superseded and shows a **SUPERSEDED** banner. |
| 10 | **Permit expires** | An "Expired" badge appears. Reminders come in the next release. |
| 11 | **Accidental delete** | The item goes to Trash and can be restored by anyone with folder Manage rights. Permanent deletion is company-admin only, with a reason, and logged. Nothing is purged automatically. |
| 12 | **Same file arrives twice** | Matching fingerprint: warning "this file already exists as [Document]", with options to **link to existing** or file anyway. |
| 13 | **Emergency publication** | Not needed in the MVP: anyone with Manage rights can mark official immediately. With approval chains (next release), an "urgent" bypass needs a reason. |
| 14 | **Multiple approvers** | Next release: approval chains on "mark official". |

### 6.4 Who can do what (MVP)

Access levels confirmed by D-053 (Phase 8, [Section 12 Part B](12-security-and-antivirus.md)). Company admins don't see the contents of Confidential folders unless they add themselves (D-055). Downloading originals is a separate role ability (D-054).

| Action | View | Contribute | Manage | Company admin |
|---|---|---|---|---|
| See document and official version | ✓ | ✓ | ✓ | ✓ |
| See drafts and history | — (configurable) | ✓ | ✓ | ✓ |
| File from Inbox, upload new versions (as Draft) | — | ✓ | ✓ | ✓ |
| **Mark official, withdraw, restore** | — | — | ✓ | ✓ |
| Delete to Trash / restore from Trash | — | Own drafts only | ✓ | ✓ |
| **Delete permanently** | — | — | — | ✓ (reason required) |

### 6.5 Activity log events (MVP)

Captured, filed, version added, marked official, withdrawn, restored, details changed (old → new), linked or unlinked, moved, permissions changed, viewed, downloaded, exported, deleted, restored from Trash, deleted permanently, fixity check result. Protection, visibility and retention are set in [Section 09](09-audit-trails-and-integrity.md) (D-060 to D-063).

## 7. Alternatives and Trade-offs

| Question | Options | Recommendation and why |
|---|---|---|
| Is the first filing official? | Always Draft / always Official / **user chooses, default per type** | Choosing keeps receipts quick and contracts safe, and the choice is an explicit, logged act |
| Locking | Check-out locks / **no lock plus a "based on" warning** | Locks get forgotten and block SMEs; the warning catches the real risk |
| Amendments | New version / **separate related document** | Both are in force; a version would wrongly supersede the original |
| Details history | Version the details / **log old → new values** | Simpler; the log is enough for audits |
| Permanent delete | Never / auto after N days / **admin only, with reason** | Respects D-008 and privacy disposal duties; the full retention policy comes in the next release |

## 8. Recommendation

Adopt §6 as the lifecycle model for the MVP. Approval chains, urgent bypass, retention and legal hold are designed in the next release on top of the same states. *(Recommendation; Grilling Round 8)*

## 9. Cost and Operational Implications

- Low extra development cost: states and checks on one "mark official" action.
- Deduplication by fingerprint saves storage.
- Trash with no automatic purge means storage grows; the storage allowance covers it (D-018 / Section 18).

## 10. Risks and Limitations

| ID | Risk | Mitigation |
|---|---|---|
| RK-27 | **Staff skip marking official**, so documents sit as Draft | Default-to-Official for record types; "documents with no official version" list |
| RK-28 | **Wrong person marks official** | Manage-only permission; logged; withdraw and restore; approval chains next release |
| RK-29 | **Trash grows forever** | Admin bulk purge with reason; retention rules next release |

## 11. Questions for the Senior Partner

None needed: this is an operational design for the owner, so there is no separate meeting guide. §3 can be shown to the senior partner for awareness.

## 12. Decisions Required

Grilling Round 8.

## 13. Confirmed Decisions

| ID | Decision |
|---|---|
| D-035 | Vocabulary (see [GLOSSARY.md](../GLOSSARY.md)) |
| D-036 | Filing rule: choose Official record or Draft (default per type); later uploads start as Draft |
| D-037 | Manage rights to mark official, withdraw, restore |
| D-038 | Version states: Draft, Official, Superseded, Withdrawn |
| D-039 | No locking; "based on" warning |
| D-040 | Amendments are separate related documents |
| D-041 | Trash; no automatic purge; admin-only permanent delete with reason |
| D-042 | Duplicate warning by fingerprint |
| D-043 | Details changes logged old → new, not versioned |

## 14. Open Questions

- Officetul's existing roles: do "View / Contribute / Manage" map onto them? (Questionnaire Q8)

## 15. Dependencies

- **Feeds:** Section 09 (activity log protection), Section 12 (permissions), Section 13 (data model), Section 17 (MVP).
- **Depends on:** D-014, D-019, D-020, D-026, D-031.

## 16. Sources

R02, R05, R08, R09 (as cited in §5).

## 17. Next Steps

**Acceptance criteria:**
- [x] Terms confirmed; GLOSSARY.md created (D-035)
- [x] Version states and filing rule agreed (D-036, D-038)
- [x] Permissions for official marking and deletion agreed (D-037, D-041)
- [x] Amendments, duplicates and details history agreed (D-040, D-042, D-043)
