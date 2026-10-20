# 0003 — Fail-closed company guard plus composite company keys

**Status:** Accepted, 2026-10-21. Decisions D-052, D-075 (also D-091). Subject to the D-013 code review.

## Context

Many companies share one database (ADR-0001). Laravel tenancy packages leave queries **unfiltered** when no company is set. Global scopes cover only Eloquent models, not raw queries, and MySQL has no row-level security (R15 §1). Real leaks came from cached logged-in pages, state left over between requests, guessable IDs and over-broad storage links (R15 §3). One leak of a client's confidential files would end trust and could be a reportable breach (D-032).

## Decision

- **Guard:** every Documents model, job and command runs under a guard that **throws** when no company is known. Cross-company code uses one logged `runAsSystem` escape.
- **Database backstop:**
  - ULID keys;
  - `UNIQUE (tenant_id, id)` on parents and `(tenant_id, parent_id)` foreign keys on children, so no row can point at another company's row;
  - separate database users.
- **Requests:** another company's ID returns 404. Raw queries and `withoutGlobalScopes` are banned in the Documents namespace.
- **Release gate:** a two-company test suite (route sweep, "list equals detail", job, search, preview and storage tests) **blocks release**. A manual two-company check runs before each pilot, including our own pilot on production (D-091).

## Consequences

- **Good:**
  - A forgotten filter becomes an error, not a leak.
  - The database refuses cross-company links even if code is wrong.
  - Our own pilot can run safely on production.
- **Bad:**
  - Every table carries `tenant_id` and wider keys.
  - Ops scripts need the explicit escape.
  - If Officetul's own tenancy is unscoped with no company set, other apps stay exposed. That fix belongs to the maintainer (RK-43, D-096).
  - The test suite must stay complete as routes are added.

## Alternatives considered

- **Officetul's tenancy as it is:** fails open in common packages.
- **Database per company:** strongest, but high ops cost; kept only if Officetul already works that way.
- **MySQL views per company:** a view can't follow the session's company (R15 §1.4).
- **PostgreSQL for row-level security:** a second engine for one module. Rejected (D-074).
