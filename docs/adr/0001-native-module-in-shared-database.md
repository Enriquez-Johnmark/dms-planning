# 0001 — Documents is a native Officetul module in the shared database

**Status:** Accepted, 2026-10-21. Decisions D-002, D-007, D-010, D-075 (also D-074, D-096). Subject to the D-013 code review.

## Context

Officetul is sold as a one-stop ERP for Philippine SMEs (D-002). The team is two developers (D-089). We could:
- embed an open-source DMS (Paperless-ngx, Mayan, Alfresco);
- run Documents as a separate service with its own database;
- build it inside the Officetul codebase.

Links between Documents and records (an Expense, a Project) are the selling point (D-019). Search must filter by access inside the query (D-056), and that filter needs Folders, grants and linked records in one database.

## Decision

- Documents is built natively in the Officetul Laravel + Inertia + React codebase.
- It shares Officetul's MySQL 8.4 database, login, roles and company model.
- Open-source tools are used only as components (ClamAV, Tesseract, Gotenberg, Tika).
- Other Officetul apps may use only `Documents\Contracts\*`, `Documents\Events\*` and the React Documents panel.
- Pest architecture tests enforce that boundary.
- Apps register a `LinkableRecordProvider`. Links use a `linked_records` registry with an enforced morph map.

## Consequences

**Good:**
- One login, one permission model, one company model and one deploy for a small team.
- Link and access checks run as SQL joins in the same database. There is no sync and no stale copy.
- The expenses report (D-072) is one query.

**Bad:**
- Documents' schema rules (composite keys, ULIDs, morph map, separate database users) touch Officetul-wide settings. That work is owned by the Officetul maintainer (D-096).
- Documents inherits Officetul's MySQL version, hosting and release cycle.
- Selling Documents standalone later would mean extracting auth, tenancy and links. That is a large job.

## Alternatives considered

- **Embed an open-source DMS:** a second login and permission model, and AGPL licence risk. Rejected by D-010.
- **Separate service and database:** cleaner boundaries, but access filtering in search would need synced copies of grants and record visibility. That is the leak path D-056 forbids, and it means two systems to run.
- **Module with its own schema on the same server:** no gain over shared tables while foreign keys to Officetul records are wanted.
