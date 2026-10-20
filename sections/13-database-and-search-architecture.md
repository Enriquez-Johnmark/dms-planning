# 13 — Database and Search Architecture

## 1. Purpose

Decide **which database and search approach** Officetul Documents uses, and how search stays fast, useful and exact about who may see what. The tables themselves and the connection to other Officetul apps are in [Section 16](16-erp-integration-and-data-model.md).

## 2. Status

**Owner agreed (Round 12, D-064 to D-068, D-074, D-075), 2026-10-20.** Research: [R16](../research/R16-search-architecture.md) (search) and [R17](../research/R17-data-model-and-erp-integration.md) §6–§8 (database), both 2026-10-20. Designed without the Officetul code; checked against it later (D-013).

## 3. Executive Summary

- **Stay on MySQL.** No agreed requirement needs PostgreSQL. Every PostgreSQL advantage (row-level security, partial indexes, transactional migrations, richer JSON indexing) has a documented MySQL workaround. *(Recommendation → D-074)*
- **Minimum MySQL 8.4 LTS.** If Officetul runs 8.0 (out of support since April 2026, **Secondary**) or MariaDB, the shared database is upgraded before the pilot. *(D-074)*
- **Search starts on MySQL's built-in full-text search**, with the company and access check inside the same query. Folder moves, Trash and permission changes take effect on the next search. There is no new server to run. *(D-064)*
- **What we give up for now:**
  - no typo tolerance ("Meralko" finds nothing);
  - snippets are built in our own code;
  - weaker ranking on long scanned documents.

  We move to **Meilisearch** (free, MIT licence) when measured triggers fire. *(D-064)*
- **Numbers get special handling:** TINs and invoice and receipt numbers go in their own lookup table, found with or without dashes. *(D-066)*
- **Search never hints and never remembers:**
  - no suggestions from other people's documents;
  - no stored search terms, only daily counts. *(D-067)*

## 4. Business Context

Search is the reason clients digitise: "find the Meralco bill for March", "find everything with this supplier's TIN". Two failures would hurt most:
- **Showing a hidden document.** This is a privacy breach (D-056).
- **Search too slow or too literal to trust.** Users would go back to paper.

The pilot (own company, then one client, D-017) is small, so exactness and simplicity come first, and speed and typo tolerance are measured before we pay for them.

## 5. Verified Research Findings

| Finding | Source |
|---|---|
| MySQL full-text has no typo tolerance, no highlighting and no stemming; its default minimum word length and stopwords drop short words such as "OR" | R16 §3.1 |
| Meilisearch Community Edition is MIT-licensed (v1.55.0); sharding is Enterprise-only, and replication probably is too (from a search excerpt only) | R16 §3.2 |
| Meilisearch stops indexing a field after 65,535 word positions, so long scans must be indexed page by page | R16 §3.2 |
| Typesense keeps the whole index in RAM (about 2–3× the text) and recommends 3 servers | R16 §3.3 |
| Laravel Scout's documentation says third-party engines ignore Eloquent global scopes, so the company filter must be added by hand | R16 §3.5 |
| MySQL 8.4 refuses foreign keys to non-unique keys by default; allows unique indexes on virtual generated columns; supports recursive CTEs | R17 §2, §8 |
| MySQL schema changes commit implicitly and can't be rolled back | R17 §6.3 |
| LTS series get 5 years of premier and 3 years of extended support. The 8.4 dates (to 2029/2032) come from a secondary source | R17 §6.4 |
| Odoo runs only on PostgreSQL; Frappe/ERPNext defaults to MariaDB, so both engine families carry full ERPs | R17 §8 |

## 6. Technical Analysis

### 6.1 Search design *(D-064 to D-067; R16 §4)*

1. **One entry point.** A `DocumentSearch` interface with a MySQL adapter now and a Meilisearch adapter later, sharing one set of contract tests that include the two-company and hidden-folder cases (D-052). The browser never talks to a search engine.
2. **Search table.** One row per searchable Version, holding the title, Document type, flattened details and OCR text up to a cap. Per-page text is kept separately for snippets with page numbers.
3. **Access filter inside the query.** The search query joins folders, grants and link grants live. Results, counts and facets all come from the same filtered query.
4. **What each person searches (D-065).**

   | Access level | Searches |
   |---|---|
   | View | Details and the Official version's text |
   | Contribute / Manage | Also Drafts; Superseded and Withdrawn text with "Include history" |
   | Manage | "Search Trash", kept separate |

5. **Query helper.**
   - All words must match by default, and the last word may be partial.
   - Words such as "OR" or "2307" become Document type filters.
   - Identifiers are detected and looked up in their own table (D-066).
6. **Taglish:** a product-wide list of synonyms (e.g. *resibo* → receipt). Affixed Filipino words (*binayaran*) are a known gap for every engine.

### 6.2 Switch triggers *(D-064)*

| Trigger | Measure |
|---|---|
| Speed | 95% of searches slower than 1.5 s on production data |
| Typos | More than 10% of searches find nothing, and the test shows Meilisearch would find most of them |
| Scale | More than about 20 GB of searchable text |
| Product | A paying client requires typo-tolerant search |

Each trigger must hold for two consecutive weeks. The counts behind them are kept without storing search terms (D-067).

### 6.3 Sizing *(Estimate; R16 §6, R17 §6.1)*

| Scale | Searchable text | MySQL full-text disk | Core Documents tables |
|---|---|---|---|
| Year 1: 20 companies × 5,000 Documents | ≈ 1.7 GB | ≈ 3–4 GB | under 1 GB |
| Long run: 100 companies × 50,000 Documents | ≈ 85 GB | ≈ 130–215 GB | ≈ 35 GB, plus 18–150 GB of Activity log a year |

At year-1 scale every option is cheap, so the choice is about effort and exactness. Run a 1-million-document load test before passing about 1 million Documents.

### 6.4 Database practice *(D-074, D-075; R17 §6)*

- MySQL 8.4 LTS minimum; no Innovation releases.
- Migrations are small, one change each and re-runnable, and run as their own database user (D-061).
- The hot tables (about 15–20 GB at full scale) should fit in the database's memory cache.
- The Activity log has no foreign keys, so it can be split by date if it passes about 500 million rows.

## 7. Alternatives and Trade-offs

| Question | Options | Choice |
|---|---|---|
| Database | **MySQL** / PostgreSQL / both | MySQL: Officetul already uses it; one engine for a small team |
| Search engine | **MySQL full-text first** / Meilisearch first / Typesense / OpenSearch | MySQL first: exact and immediate access, nothing new to run; Meilisearch when triggers fire |
| Sync to a search engine | Scout automatic sync / **own outbox (later)** | Own outbox when Meilisearch arrives: Scout's engines skip the company filter |
| Suggestions | Company-wide / **none in the MVP** / per-user later | None: a suggestion can reveal a hidden document |

## 8. Recommendation

Adopt §6. Build the engine-independent parts now, so a later switch means a new adapter and a re-index rather than a redesign:
- the search interface;
- the search table;
- the identifier table;
- the query helper;
- the per-Access-level rule;
- the contract tests.

## 9. Cost and Operational Implications

- **MVP:** no new service. Full-text indexes add disk to the shared database (§6.3), and a nightly `OPTIMIZE` keeps the index tidy after deletes.
- **Later switch:** 2–4 developer-weeks, plus one more service to secure, back up and monitor (**Estimate**).
- **Upgrade:** if Officetul is on MySQL 8.0 or MariaDB, upgrading affects every Officetul app; the maintainer must estimate it (OQ-028).

## 10. Risks and Limitations

| ID | Risk | Mitigation |
|---|---|---|
| RK-47 | **MySQL full-text is slow** on common words at scale | Triggers; load test before 1M Documents; Meilisearch path ready |
| RK-48 | **Users expect typo tolerance** and trust search less | Measure the zero-result rate; Taglish synonyms; switch trigger |
| RK-49 | **Changing a server-wide full-text setting** (minimum word length, stopwords) rebuilds other Officetul modules' indexes | Check existing full-text indexes in the code review; prefer the identifier table over global settings |
| RK-50 | **The MySQL upgrade** is larger than expected | Maintainer estimate before the pilot (OQ-028) |

## 11–12. Questions and Decisions

Grilling Round 12, Q1–Q5, Q11 and Q12.

## 13. Confirmed Decisions

| ID | Decision |
|---|---|
| D-064 | MySQL full-text first, behind a swappable interface; Meilisearch when triggers fire |
| D-065 | What each Access level searches; Trash and history separate |
| D-066 | Identifier table for TINs and invoice and receipt numbers |
| D-067 | No cross-document suggestions; no stored search terms |
| D-068 | Search and leak test on the benchmark set before the first external client |
| D-074 | Stay on MySQL; 8.4 LTS minimum |
| D-075 | Technical defaults (shared with Section 16) |

## 14. Open Questions

- How fast is MySQL full-text with millions of OCR-like rows on our server class? (Load test.)
- How does MySQL split `123-456-789-000`, `2550Q` and `OR#4521`? (Test on the target version.)
- The exact BIR TIN format for the identifier pattern (tax adviser, D-027).
- The MySQL version and settings, existing full-text indexes, Scout and global search in Officetul ([ERP questionnaire](../questionnaires/erp-technical-questionnaire.md) Q4, Q15, Q25–Q26).
- The MySQL upgrade effort, if needed (OQ-028).

## 15. Dependencies

Depends on D-026, D-049, D-050, D-052, D-053, D-056 and D-060. Feeds Section 14 (database server size), Section 17 (MVP scope) and Section 18 (cost).

## 16. Sources

R16 and R17 (all accessed 2026-10-20), especially the MySQL 8.4 reference manual, the Meilisearch and Typesense documentation, and the Laravel Scout 13.x documentation.

## 17. Next Steps

- [x] Search approach and switch triggers agreed (D-064)
- [x] Search coverage, identifiers and privacy agreed (D-065 to D-067)
- [x] Search test agreed (D-068)
- [x] Database engine agreed (D-074)
- [ ] Questionnaire answers on MySQL version and settings
- [ ] Load test before about 1M Documents
