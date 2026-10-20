# 0006 — Folders alone grant access; no per-Document permissions

**Status:** Accepted, 2026-10-21. Decisions D-053, D-093 (also D-020, D-054, D-055, D-056, D-092, D-098a).

## Context

SMEs need "only HR sees HR". But per-file rules multiply fast, are hard to explain, and make every search query costlier. SharePoint advises minimising unique permission scopes, and Google restricts only at folder level (R15 §5.1). Link-based visibility (anyone who may open Expense 881 sees its receipt) depends on how Officetul decides record access (OQ-025). That is unknown until the code review, and it carries leak risk (RK-13, RK-46).

## Decision

- **Access levels:** grants on **Folders** at three levels.
  - **View:** Official version, details, preview.
  - **Contribute:** adds Drafts and history, filing, new Versions as Draft, and moving own Drafts to Trash.
  - **Manage:** adds marking Official, withdrawing, restoring, Trash and changing grants.
- **Folder rules:** subfolders inherit unless **Restricted**. **Confidential folders** hide their contents from company admins; an admin may add themselves, which is logged and announced.
- **Role abilities** sit on top: use, download, export, admin, Inbox reviewer.
- **What never grants access:** per-Document permissions (none in the MVP); Sensitivity (a label only); Links, until the "Grants view" switch arrives as a Should item after OQ-025.
- **Explaining access:** "Why can I see this?" and "Who can see this?" panels.

## Consequences

- **Good:**
  - One SQL check through `acl_folder_id`, so search filtering is fast and exact.
  - Easy to explain and test.
  - No dependency on unseen Officetul code.
- **Bad:**
  - An exception for one file means moving it to a Restricted folder.
  - Expense approvers can't see linked receipts unless they hold Folder access.
  - Adding per-Document rules or Link visibility later touches the search query, panels, the "Why" panel and the leak tests.

## Alternatives considered

- **Roles only:** can't express "only HR sees HR".
- **Per-Document permissions:** more scopes, slower search, and the pattern Microsoft and Google steer away from.
- **A separate permission service (OpenFGA, SpiceDB):** no maintained PHP client, and syncing it is our job.
- **Link visibility in the MVP:** blocked on OQ-025. Moved to Should by D-093.
