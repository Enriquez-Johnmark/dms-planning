# MVP Screen Sketches

Low-detail sketches of the six main Officetul Documents screens, made on 2026-10-21 for the senior partner meeting. They show **what the agreed decisions look like**; they add no new scope. Sample company, people and amounts are fictional.

- **Live canvas (private until shared):** https://claude.ai/artifact/5wmzY5kg24w1774v4k8L1Z
- **Source files:** [`screens/`](screens/) (one `.dc.html` per screen plus `canvas.json`; they render only in the canvas)

| # | Screen | File | Shows | Decisions |
|---|---|---|---|---|
| 1 | Inbox: filing a captured file | `Main.dc.html` | Captured files with their state (ready, security check pending, password-protected, possible duplicate); filing form with Document type, Folder, details, "File as Official record / Draft", link to an Officetul record, Paper original | D-014, D-026, D-036, D-042, D-047, D-048, D-092 |
| 2 | Document page | `Document.dc.html` | Versions with Draft / Official / Superseded, Revision labels, expiry badge, details, Paper original, Links, preview, integrity check, "Why can I see this?", history | D-026, D-035–D-038, D-043, D-053, D-061 |
| 3 | Folder list | `Folder.dc.html` | Folder tree with Restricted and Confidential folders, access summary, saved filters (incl. "No Official version"), document list | D-053, D-055, D-110d |
| 4 | Search results | `Search.dc.html` | Full-text results with page-numbered snippets, filters with counts from permitted documents only, identifier tip | D-056, D-064–D-067 |
| 5 | Documents panel on Expense #881 | `Expense.dc.html` | Linked invoice (Official), a just-attached receipt being checked, attach area | D-019, D-075, D-098h |
| 6 | Expenses without an invoice or receipt | `Report.dc.html` | Expenses missing support, with the reason (no document, Draft, Withdrawn) | D-023, D-056, D-072 |

**Not shown (by decision):** link-granted access (D-093), scan-to-email (D-094), staff support access (D-095), "Include history" and "Search Trash" (D-090).
