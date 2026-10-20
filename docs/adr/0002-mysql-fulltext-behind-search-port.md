# 0002 — MySQL FULLTEXT search first, behind a search port

**Status:** Accepted, 2026-10-21. Decisions D-064 (also D-056, D-065, D-066, D-067, D-074).

## Context

Search is why clients digitise. The two failures that hurt most:
- showing a hidden Document (a privacy breach, D-056);
- search too slow or too literal to trust.

Dedicated engines (Meilisearch, Typesense, OpenSearch) offer typo tolerance and better ranking. But they need a synced copy of every access rule, and one more service to secure, back up and monitor. Laravel Scout's third-party engines also ignore Eloquent global scopes (R16 §3.5). Pilot scale is small: about 1.7 GB of searchable text in year 1 (Estimate).

## Decision

- MVP search uses **MySQL InnoDB FULLTEXT** through one `DocumentSearch` port with a MySQL adapter.
- The company and access filter is joined live **inside the same query**. Counts and facets come from that query.
- Identifiers (TINs, invoice and receipt numbers) go in their own normalised table (D-066).
- No suggestions are built from documents, and no search terms are stored (D-067).
- Contract tests, including the two-company and hidden-folder cases, run against any adapter.
- Move to self-hosted Meilisearch CE when a trigger holds for two weeks:
  - p95 search time over 1.5 s;
  - zero-result searches over 10% that Meilisearch would fix;
  - more than about 20 GB of searchable text;
  - a paying client requires typo tolerance.

## Consequences

**Good:**
- Access changes take effect on the next search.
- No index drift, and no new server.
- Exact counts with no hints about hidden Documents.

**Bad:**
- No typo tolerance ("Meralko" finds nothing).
- Snippets are built in our own code.
- Weaker ranking on long scans.
- Server-wide full-text settings can't be changed without affecting other apps (RK-49).
- A load test is needed before about 1 million Documents.
- The later switch costs 2–4 developer-weeks (Estimate), plus an outbox and one more service.

## Alternatives considered

- **Meilisearch first:** better search, but access lists in the index and one more service from day one.
- **Typesense:** whole index in RAM, and three servers recommended.
- **Filtering after search:** leaks through counts and paging (R15 §6). Rejected.
