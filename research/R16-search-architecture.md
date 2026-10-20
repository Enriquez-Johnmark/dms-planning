# R16 — Search engine and search architecture

- **Status:** First pass, design-level. Research only, no code. **Written without access to the Officetul code**, so the design sits behind an interface that adapts to whatever Officetul already uses (D-013). §10 lists what to check when the code is available.
- **Researched:** 2026-10-20. **All sources accessed 2026-10-20.**
- **Question:** Which search engine and design should Officetul Documents use for full-text search over document details and OCR text, given that MySQL is preferred, the team is 1–2 developers (D-005), and access filtering must be exact (D-056)?
- **Labels:** **Verified fact** (read in the primary source), **Vendor claim**, **Secondary** (news, a third party or a search excerpt only), **Estimate**, **Open question**, **Recommendation** (proposed, not approved). Source IDs `[S#]` point to §14.

**Access caveats.**
- Most vendor pages were read through a fetch tool that summarises pages. Short quotes should be re-checked before they go into any client-facing text.
- Version numbers and licences come from the GitHub REST API and the raw `LICENSE` files, read on 2026-10-20.
- The Scout documentation is titled "Laravel 13.x", but the package itself is **laravel/scout 11.9.0**, which supports Laravel 9–13 [S41]. "Scout 13.x" in this note means "the Scout docs for Laravel 13".
- The MySQL 8.4 manual does not describe how the built-in parser handles a hyphen inside `123-456-789-000`. The behaviour below is inferred from the fine-tuning page. It is marked **Estimate** and must be tested.
- The OpenSearch install page and the Meilisearch multitenancy guide did not render. The Meilisearch delete-by-filter endpoint was seen only in search excerpts.
- **No published benchmark** was found of MySQL FULLTEXT, Meilisearch or Typesense on multi-tenant OCR text at a scale of millions of documents. Every size and speed figure for our case is an **Estimate** and must be measured (§7).

**Terms** follow [GLOSSARY.md](../GLOSSARY.md): Document, Document type, Folder, Version, Official version, Draft, Superseded, Withdrawn, Derived file, Link, Trash, Access level, Restricted folder, Confidential folder, Activity log. Other terms used here:
- **Company:** the tenant (as in R15).
- **Search row:** one entry in the search index. In the design below, a Search row holds one Version's searchable text, plus the Document's details.
- **Search port:** the single PHP interface that the rest of Documents calls to search. Engines plug in behind it.
- **Stale set:** a short list of Document IDs whose index entry is known to be out of date. Search excludes them until they are re-indexed (§5).

---

## 1. Short answer

1. **MVP: MySQL 8.4 InnoDB FULLTEXT behind a Search port, with the company and access filter as a SQL join in the same query.** This adds no new service. Access is always current, because the folder and grant tables are read live. Counts and facets come from the same `WHERE`, which makes D-056 straightforward. **Recommendation**
2. **What MySQL lacks:** typo tolerance, highlighting and stemming. Default relevance ignores document length (§3.1). We build snippets in PHP and accept "no typo tolerance" for the pilot. **Verified fact** for the gaps [S1–S9]; workaround **Recommendation**
3. **Later: Meilisearch Community Edition (MIT), self-hosted, behind the same Search port, fed by an outbox.** We move when measured triggers fire (§8): slow searches, too many zero-result searches from typos, or data growth. **Recommendation**
4. **Typesense** keeps the whole index in RAM (2–3× the indexed text) and recommends a 3-node cluster, which is too costly for OCR-heavy data. **OpenSearch/Elasticsearch** costs too much to operate for 1–2 developers. Neither is recommended. **Estimate** [S30][S39]
5. **Engine-independent parts** must be built in the MVP anyway: a search projection table; an identifier table for TINs and invoice numbers; a query parser for document-type aliases such as "OR"; a rule for which Versions are searchable at which Access level; a stale set; and a keyword-hit test suite. **Recommendation**

---

## 2. Requirements derived from the decisions

| # | Requirement | Source decision | Label |
|---|---|---|---|
| R1 | Search title, Document type, details fields and OCR text in one box | D-014, D-024, D-049 | Verified fact (decision log) |
| R2 | OCR text is `eng+fil`, with optional `ceb` per company, and is noisy on receipts and phone photos | D-049, R12 §1 | Verified fact |
| R3 | Filters: Document type, Folder (with subfolders), date range (document, event, expiry dates), linked Officetul record, version status, expiry badge | D-019, D-026 item 4, D-038 | Recommendation |
| R4 | Snippets with the matched words highlighted, and the page number when the hit is in OCR text | D-014 ("OCR full-text search") | Recommendation |
| R5 | Some typo tolerance ("Meralko" finds "Meralco") | — | Recommendation (not decided; Decision 5) |
| R6 | Filipino and Taglish words (e.g. *kasunduan*, *bayad*, *resibo*). Affixed forms such as *binayaran* and *pagbabayad* are a known gap for every engine | R12 §6.1 | Estimate |
| R7 | Number and ID search: TIN `123-456-789-000` typed with or without hyphens; invoice, OR and CR numbers; BIR form numbers (2307, 2550Q); case numbers | D-021, D-023 | Recommendation |
| R8 | "OR" is a search alias for Payment Receipt (OR/CR/AR) | D-023 | Verified fact (decision log) |
| R9 | Company and access filters run **inside** the query; Laravel proxies every search; the browser never talks to the engine | D-052, D-056, R15 §6 | Verified fact (decision log) |
| R10 | Results, snippets, counts, facets and suggestions come **only** from the viewer's permitted set; empty states are identical | D-026 item 6, D-056 | Verified fact (decision log) |
| R11 | View sees the Official version and details. Contribute also sees Drafts and history. Record links grant View only | D-053 | Verified fact (decision log) |
| R12 | Trash is restorable. Permanent delete is by company admin only. Withdrawn is a state, not a deletion | D-038, D-041 | Verified fact (decision log) |
| R13 | Password-protected files are not searchable | D-048 | Verified fact (decision log) |
| R14 | No search terms in the Activity log or Security log | D-060 | Verified fact (decision log) |
| R15 | A file can be up to 1 GB, so a single Version may produce very long OCR text | D-045 | Verified fact (decision log) |
| R16 | Two-company isolation tests block a release, and search is in scope | D-052, R15 §4.2 | Verified fact (decision log) |
| R17 | Built and run by 1–2 developers. Pilot 4–6 months after scope approval | D-005 | Verified fact (decision log) |

---

## 3. Options

### 3.1 MySQL 8.4 InnoDB FULLTEXT

| Point | Detail | Label |
|---|---|---|
| Index type | Inverted index storing the word, the documents it occurs in, and byte positions. It is split into 6 auxiliary tables. Index creation uses 2 threads by default (`innodb_ft_sort_pll_degree`). | **Verified fact** [S8] |
| Column types | `CHAR`, `VARCHAR`, `TEXT` only, on InnoDB or MyISAM. All columns of one FULLTEXT index must share a character set and collation. | **Verified fact** [S1][S5] |
| **No partitioned tables** | "Full-text searches are not supported for partitioned tables." Tenants cannot be split into partitions of one table. | **Verified fact** [S5] |
| `MATCH()` columns | The column list must exactly match one FULLTEXT index, and `AGAINST()` must be a constant | **Verified fact** [S5] |
| Commit-time indexing | Inserts, updates and deletes are tokenised at commit, so a search sees only committed data. An index cache holds recent inserts (`innodb_ft_cache_size` 8,000,000 bytes per table; `innodb_ft_total_cache_size` 640,000,000 bytes). | **Verified fact** [S8][S9] |
| **Deletes do not shrink the index** | A deleted row's DOC_ID goes into an `FTS_*_DELETED` table, and the "indexed record remains in the full-text index". Space comes back only after `OPTIMIZE TABLE` with `innodb_optimize_fulltext_only=ON`. | **Verified fact** [S8] |
| `FTS_DOC_ID` | Defining it at `CREATE TABLE` avoids a table rebuild. Values must keep increasing; the gap between the largest used and the next may be at most 65,535. | **Verified fact** [S8] |
| Word characters | "letters, digits, and underscores" plus single apostrophes. Delimiters include space, comma and period. The manual shows how to make `-` a word character, which means a hyphen is **not** one by default. | **Verified fact** [S2][S6] |
| TIN `123-456-789-000` | Likely indexed as the four tokens `123`, `456`, `789`, `000`. A plain natural-language search for it would also return every document that contains `000`. A quoted phrase `"123 456 789 000"` in boolean mode should match in order. | **Estimate** (test it) |
| Minimum word length | `innodb_ft_min_token_size` defaults to **3** (range 0–16); `innodb_ft_max_token_size` defaults to 84. Both are global, need a **restart** and need a rebuild of every FULLTEXT index on the server. | **Verified fact** [S4][S9] |
| Default stopwords | 36 entries, including `a`, `at`, `or`, `on`, `in`, `is`, `to`, `the`. Can be replaced server-wide or per table with a stopword table set **before** the index is created. | **Verified fact** [S4] |
| Consequence for "OR", "CR", "AR" | "OR" is both a stopword and below the 3-character minimum, so a full-text search for "OR 1234" ignores "OR". **The alias must be handled by our query parser** as a Document type filter (D-023), not by the full-text index. | **Estimate** from the facts above |
| Filipino particles | *ng*, *sa*, *na*, *si* fall below the 3-character minimum. *ang*, *mga*, *ay*, *kay* are indexed unless added to a custom stopword list. | **Estimate** |
| Natural language mode | No operators except `"`. Rows come back sorted by relevance only when there is no `ORDER BY`, the full-text index is used, and it is the leftmost non-constant table in a join. | **Verified fact** [S2] |
| Boolean mode | `+` (must), `-` (must not), `*` (prefix), `"phrase"`, `@distance` (InnoDB only). InnoDB supports only *leading* `+`/`-`. "A boolean-mode search that contains only terms preceded by `-` returns an empty result." | **Verified fact** [S3] |
| Ranking | TF-IDF variant: `rank = TF × IDF × IDF`, summed over words ("based on BM25 and TF-IDF"). The formula on the page has **no document-length factor**. | **Verified fact** [S3] |
| Ranking consequence | Long OCR documents (a 100-page contract that mentions "Meralco" 40 times) will tend to outrank a one-page Meralco bill. Rank by title and type match first, then score, then recency. | **Estimate** |
| ngram parser | Built for CJK. Default token size 2 (range 1–10). It ignores `innodb_ft_min_token_size`. A phrase `"abc def"` does **not** match `abcdef`. | **Verified fact** [S7] |
| ngram for our case | It would give substring matching (e.g. inside run-together OCR words) but a much larger index and noisy results for English and Filipino. **Not recommended.** | **Estimate** |
| Stemming and typos | The manual documents neither stemming nor typo tolerance. The only fuzzy aid is "query expansion" (blind feedback). | **Verified fact** (absence in [S1]–[S6]) |
| Highlighting | None. `MATCH()` returns only a score. | **Verified fact** [S1] |
| Result-cache limit | `innodb_ft_result_cache_limit` defaults to 2,000,000,000 bytes per query (dynamic). A very common word over millions of rows may hit it. | **Verified fact** (variable) [S9]; behaviour at our scale **Estimate** |
| Tenant filter performance | The full-text index is **global** across companies. `MATCH()` finds candidates in every company before `tenant_id = ?` removes them, so the cost of a common word grows with the **whole** table, not one company. | **Estimate** (no primary source found; must benchmark) |
| MySQL 9.x | 9.7.4 was released 2026-10-20. The "What is new in 9.7" page lists no full-text changes. Whether 9.7 is the next LTS was not confirmed on the pages read. | **Verified fact** [S10]; LTS status **Open question** |
| HeatWave | Oracle's managed service offers vector and "hybrid" semantic plus keyword search. It is not MySQL Community and ties hosting to Oracle Cloud. Out of scope. | **Secondary** [S11] |

**Laravel side:** `whereFullText($cols, $q, ['mode' => 'boolean'])` compiles to `match (...) against (? in boolean mode)`, so the search string is always a bound parameter [S42]. **Verified fact**

### 3.2 Meilisearch

| Point | Detail | Label |
|---|---|---|
| Version | **v1.55.0** (2026-10-12). Releases come roughly every 2–3 weeks; patches cover older minors. | **Verified fact** [S13] |
| Licence | `SPDX: MIT AND BUSL-1.1`. Files marked "Enterprise Edition" are under BUSL 1.1. Production use of those needs a commercial licence. Each version converts to MIT four years after publication. Everything else is MIT. | **Verified fact** [S14] |
| What is Enterprise-only | The docs say "The only feature exclusive to the Enterprise Edition is" **sharding**. Replication also needs EE ≥ 1.37. | **Verified fact** [S15]; replication **Secondary** (search excerpt) |
| Consequence | A single self-hosted node of Community Edition is MIT and fine for production. **High availability via replication is not free.** Fall back to fast rebuilds from MySQL (§5). | **Estimate** |
| Multi-tenancy | One shared index with a `tenant_id` filter. Tenant tokens embed a filter that "automatically" applies to every search. Tokens restrict only the search endpoint. | **Verified fact** [S24] |
| Our use | Laravel proxies every search with the master or search key server-side, and adds `tenant_id = X AND (acl_folder_id IN [...] OR document_id IN [...])`. **No tenant tokens in the MVP** (R15 §6). | **Recommendation** |
| Filters | `IN [a, b]`, `NOT IN`, `=`, `!=`, `>`, `TO`, `EXISTS`, `IS NULL`, `IS EMPTY`. Attributes must be in `filterableAttributes`. Comparison operators must be enabled per attribute (`filter.comparison` defaults to false). | **Verified fact** [S21][S22] |
| Filter limits | Nesting depth 200. Each filterable value ≤ 468 bytes. **No documented limit on the number of values in `IN`.** | **Verified fact** [S16][S21]; large-list performance **Open question** |
| Query limits | **10 words per query** (later words ignored). 1,000 hits by default (`maxTotalHits`). 100 values per facet by default. | **Verified fact** [S16][S22] |
| **Positions per attribute: 65,535** | Words beyond that are "silently ignored". A hard separator (comma, period) uses 8 positions. A long OCR text in **one** attribute could be cut after roughly 10–30 pages. | **Verified fact** [S16]; page estimate **Estimate** |
| Consequence | Index **one Search row per page, or per small group of pages**, and group with `distinctAttribute = document_id`. | **Recommendation** |
| Counts | `offset/limit` returns `estimatedTotalHits`. `page/hitsPerPage` returns exhaustive `totalHits`. Use page mode so counts are exact and come from the filtered set. | **Verified fact** [S20] |
| Highlighting | `attributesToHighlight`, `attributesToCrop` (`cropLength` 10 words by default), custom pre/post tags, `showMatchesPosition`. | **Verified fact** [S20] |
| XSS | OCR text is untrusted. Use sentinel tags and escape in React, never `dangerouslySetInnerHTML` (same warning as PostgreSQL's `ts_headline` [S12]). | **Recommendation** |
| Typo tolerance | On by default. 1 typo from 5 characters, 2 from 9. A typo on the first letter counts double. Numbers are typo-tolerant by default (`disableOnNumbers: false`), so `2024` can match `2025`. | **Verified fact** [S19] |
| Consequence | Set `disableOnNumbers: true`, and keep identifiers in an attribute listed in `disableOnAttributes`. | **Recommendation** |
| Language | Charabia gives Latin-script text decomposition, lowercasing and removal of nonspacing marks (*ñ* matches *n*). **Tagalog and Cebuano are not in the optimised list.** `localizedAttributes` accepts `tgl`. | **Verified fact** [S18][S22] |
| Language consequence | No Tagalog stemmer; `tgl` probably only changes detection. Use **synonyms** for a curated Taglish list (e.g. *resibo* ↔ receipt). | **Estimate** |
| Separators | Hyphen is a soft separator (distance 1); comma and period are hard (distance 8). Configurable `separatorTokens`, `nonSeparatorTokens`, `dictionary`, `synonyms`, `stopWords` (empty by default). | **Verified fact** [S22][S23] |
| Storage and RAM | "works optimally when the full dataset fits in RAM". A RAM-to-disk ratio of about 1/3 "does not materially impact performance"; about 1/10 "works well" for many workloads. "There is no reliable way to predict the final size of a database." LMDB does not return freed space to the OS. Use NVMe, not network disks. | **Verified fact** [S17] |
| Measured example | movies.json, 9.1 MB raw, gave 224 MB on disk and about 305 MB RAM. That is small documents with many fields, so not representative of OCR text. | **Vendor claim** [S17] |
| Other limits | Task database 20 GiB maximum; above 1M tasks, finished tasks are auto-deleted. Default payload 100 MB (changeable). | **Verified fact** [S16] |
| Deletes | Delete by filter: `POST /indexes/{uid}/documents/delete` with `{"filter": ...}`, available since v1.2. It is asynchronous (a task). | **Secondary** [S26] |
| Telemetry | Anonymous analytics; disable with `--no-analytics` / `MEILI_NO_ANALYTICS=true`. The vendor says it never stores document data. | **Verified fact** [S25] |
| Cloud | "Starting at $20/month"; an estimator shows $30/month for 100K documents and 50K searches. No Asia region was named. It would be a sub-processor. Self-host instead (as with D-049 for OCR). | **Vendor claim** [S27]; **Recommendation** |
| PHP client | meilisearch-php v1.17.0 (stable), with a 2.0 beta series in progress | **Verified fact** [S13] |

### 3.3 Typesense

| Point | Detail | Label |
|---|---|---|
| Version and licence | **v30.2** (2026-04-19), **GPL-3.0**. Client libraries are Apache-2.0. | **Verified fact** [S29] |
| GPL meaning | The README says running it "as a separate daemon" does not make our code a derived work; it chose GPL **not** AGPL for that reason. Duties start only when **distributing** (modified) Typesense. | **Vendor claim** [S29] |
| GPL for Officetul | As a separate SaaS service, no obligations arise from network use. If Officetul is ever shipped on-premises with Typesense bundled, we must offer Typesense's source. D-051 bans only AGPL. | **Estimate** (lawyer may confirm, D-027) |
| RAM | "you'd typically need 2X-3X MB RAM" where X is the size of the searched fields. Unindexed fields stay on disk. | **Vendor claim** [S30] |
| High availability | 3-node or 5-node cluster "strongly" recommended. At least 2 vCPU per node. | **Vendor claim** [S30] |
| Access | Scoped keys embed `filter_by` that "users will not be able to override" | **Verified fact** [S31] |
| IDs | By default special characters are removed. `token_separators` and `symbols_to_index` can be set per field. | **Verified fact** [S34][S35] |
| Typos and pages | 2 typos by default (`min_len_1typo` 4, `min_len_2typo` 7). Maximum 250 hits per page. | **Verified fact** [S33] |
| Language | Special handling for CJK, Thai, Vietnamese and right-to-left scripts. Default locale `en` "broadly supports most European languages". **No Tagalog.** | **Verified fact** [S32][S34] |
| Fit | Good engine, but holding all OCR text in RAM across 3 nodes is the most expensive option for our data shape (§6). | **Estimate** |

### 3.4 OpenSearch / Elasticsearch

| Point | Detail | Label |
|---|---|---|
| OpenSearch | Apache-2.0, latest 3.9.0 (2026-09-29). Document-level security uses role queries with `${user.name}` and `${attr...}` substitution. "Keep queries simple" for performance. | **Verified fact** [S36][S37] |
| Elasticsearch | Default code is triple-licensed AGPL-3.0 / SSPL / Elastic License 2.0; x-pack is ELv2 only. Latest 9.5.5 (2026-10-06). | **Verified fact** [S38] |
| Ops | JVM heap at most 50% of node memory, under about 26–30 GB. Mappings, analyzers, shard sizing, snapshots and upgrades all need care. | **Verified fact** [S39]; burden **Estimate** |
| Fit | Best analyzers and highlighting, but too much to operate for 1–2 developers. Elasticsearch's AGPL option conflicts with the spirit of D-051. **Not recommended.** | **Estimate** |

### 3.5 Laravel Scout (docs for Laravel 13.x; package 11.9.0)

| Point | Detail | Label |
|---|---|---|
| Drivers | `database` (MySQL/PostgreSQL), `collection`, `algolia`, `meilisearch`, `typesense`, `turbopuffer` | **Verified fact** [S40] |
| Database engine | Applies `LIKE '%q%'` to every searchable column by default. `#[SearchUsingFullText]` uses `whereFullText`; `#[SearchUsingPrefix]` uses `q%`. Conditions are joined with **OR**. "MySQL orders by relevance by default", so Scout adds no relevance ordering on MySQL. | **Verified fact** [S41] |
| Database engine consequence | A `LIKE '%q%'` on a big OCR column is a full scan. OR-ing it with `MATCH` likely stops MySQL from using the full-text index well. Do **not** use Scout's database engine as-is. | **Estimate** |
| Global scopes, database engine | The query starts from `newQuery()`, so global scopes such as a company scope apply | **Verified fact** [S41] |
| Global scopes, external engines | "search engines are not aware of your Eloquent model's global scope definitions … you should recreate the global scope's constraints". **The company filter is not automatic** with Meilisearch, Typesense or Algolia. | **Verified fact** [S40] |
| Collection engine | For prototypes and tests only; it loads all records into PHP | **Verified fact** [S40] |
| Filters | `where`, `whereIn`, `whereNotIn`. For Meilisearch they need `filterableAttributes`; engine options go through `options()`. `query()` filters only on the database engine. | **Verified fact** [S40] |
| Sync | Model observers sync on save. `queue` and `after_commit` both default to **false**. Meilisearch and Algolia always index asynchronously. | **Verified fact** [S40][S41] |
| `identify` | Sends the searching user to Algolia only; leave it off | **Verified fact** [S41] |
| Observer gap | Model-event syncing misses bulk `update()`/`delete()` queries; D-063 already rejected model-event packages for the log | **Estimate** (R14 §3) |
| Recommendation | Do **not** build Documents search on Scout's observers or its database engine. A small `DocumentSearch` port with our own MySQL adapter now and a Meilisearch adapter later (meilisearch-php directly, or a Scout engine if Officetul already uses Scout). | **Recommendation** |

### 3.6 PostgreSQL and MariaDB (brief)

- **PostgreSQL 18:**
  - `tsvector` with per-language configurations, `ts_rank` / `ts_rank_cd` (cover density, with length normalisation options) and `ts_headline` for highlighting.
  - The manual warns `ts_headline` "uses the original document … so it can be slow", and its output is not safe HTML.
  - No Tagalog configuration was checked.
  - Better than MySQL FULLTEXT on paper, but moving off MySQL is not on the table. **Verified fact** [S12]; fit **Estimate**
- **MariaDB:** shares the InnoDB full-text lineage. Not researched further, since Officetul runs MySQL. **Estimate**

### 3.7 Comparison for our case

| Need | MySQL FULLTEXT | Meilisearch CE | Typesense | OpenSearch |
|---|---|---|---|---|
| New service to run | None | 1 process + disk | 3-node cluster advised | Cluster + JVM |
| Exact, current access filter | **Live SQL join** | Filter on indexed keys; lag on changes | Same as Meilisearch | DLS or filter; lag |
| Counts and facets from permitted set | `GROUP BY` on same `WHERE` | `totalHits` + `facetDistribution` after filter | After filter | After filter |
| Typo tolerance | No | Yes | Yes | Yes (fuzzy) |
| Highlighting | Build in PHP | Built in | Built in | Built in |
| Filipino stemming | No | No | No | No (no Tagalog analyzer seen) |
| Long OCR text | `MEDIUMTEXT`, fine | 65,535-position cap → chunk per page | RAM cost | Fine |
| Licence | GPL server (already in use) | MIT (CE) | GPL-3.0 | Apache-2.0 |
| Effort for 1–2 devs | Lowest | Medium | Medium–high | High |

All rows are drawn from §3.1–3.6; the effort row is an **Estimate**.

---

## 4. Engine-independent design (build in the MVP)

All **Recommendations**.

### 4.1 Search port

- One interface, for example `DocumentSearch::search(SearchQuery $q, Viewer $v): SearchResult`, returning hits, an exact total, facets and snippets.
- The **only** caller is a Laravel controller. The browser never talks to an engine (D-056).
- Two adapters:
  - `MySqlFullTextSearch` (MVP);
  - `MeilisearchSearch` (later).
- One shared contract test suite runs against both adapters: the same fixtures and the same expected results, including the two-company and hidden-folder cases (D-052).

### 4.2 Search projection table `document_search_rows`

- **One row per searchable Version**, plus one "details only" row for a Document with no Version a View user may see. Columns:
  - `tenant_id`, `document_id`, `version_id`, `folder_id`;
  - `searchable_from` (`official` / `draft` / `history`);
  - `document_type_id`, the date columns, `issue_status`, `expires_at`, `in_trash`;
  - `title`, `type_name`, `details_text` (all detail values flattened);
  - `ocr_text` (`MEDIUMTEXT`, capped; see below).
- An explicit `FTS_DOC_ID` defined at creation [S8]. A FULLTEXT index on `(title, type_name, details_text, ocr_text)`.
- **`acl_folder_id` is not stored in this table for MySQL.** It is joined live through `folders`, so a folder move or a Restricted switch takes effect on the next query with no re-index.
- The same projection is what an external engine would receive. Switching engines is then a new adapter plus a bulk export, not a redesign.
- **OCR cap per Version.** Index at most a fixed amount of OCR text (for example the first 1 MB, about 150k words; **Estimate**). Show a "partly searchable" badge beyond it.
  - Reason: a 1 GB drawing set (D-045) can produce huge, noisy OCR text that bloats the index and dominates ranking.
  - The full OCR text stays in the Derived file.
- **Per-page text** lives in `version_page_texts` (`version_id`, `page_no`, `text`). It is used for snippets with page numbers, and later as Meilisearch's per-page Search rows (§3.2).

### 4.3 Which Versions are searchable (D-053)

| Viewer's Access level on the Folder | Searches | Label |
|---|---|---|
| View (folder grant or record-link grant) | Details + **Official version** text. If there is no Official version, details only | Recommendation (Decision 2) |
| Contribute / Manage | The above + **Draft** text. Superseded and Withdrawn text only with an "Include history" switch | Recommendation (Decision 2) |
| Anyone | Documents in **Trash** never appear in normal search. Manage holders get a separate "Search Trash" | Recommendation (Decision 3) |
| Anyone | Encrypted files (D-048): details only, never text | Verified fact (D-048) |

The rule lives in one SQL fragment shared by search, counts and facets, so a Draft's words can never surface for a View user through a count or facet.

### 4.4 Access filter inside the query (MySQL)

```sql
-- Sketch only (Estimate); real names follow Officetul's schema
SELECT r.document_id, MATCH(r.title, r.type_name, r.details_text, r.ocr_text)
       AGAINST (:q IN BOOLEAN MODE) AS score
FROM document_search_rows r
JOIN folders f ON f.tenant_id = r.tenant_id AND f.id = r.folder_id
WHERE r.tenant_id = :tenant
  AND MATCH(r.title, r.type_name, r.details_text, r.ocr_text) AGAINST (:q IN BOOLEAN MODE)
  AND r.in_trash = 0
  AND (
        (f.acl_folder_id IN (:view_acl_ids)       AND r.searchable_from = 'official')
     OR (f.acl_folder_id IN (:contribute_acl_ids) AND r.searchable_from IN ('official','draft'))
     OR (r.document_id IN (SELECT ... link-granted documents ...) AND r.searchable_from = 'official')
  )
  -- plus user filters: type, folder subtree, dates, linked record, status
ORDER BY score DESC, r.document_date DESC
LIMIT 20 OFFSET :o;
```

- **Counts and facets** run the same `WHERE` with `COUNT(DISTINCT document_id)` and `GROUP BY document_type_id` (and so on). Never count from an unfiltered query.
- The `:view_acl_ids` and `:contribute_acl_ids` lists are computed per request from the folder grants (R15 §5.2).
- **Link-granted documents.** Visibility via record links is a subquery if Officetul's record access can be expressed in SQL. Otherwise it is a precomputed list. **Open question**: it depends on how Officetul decides who may open an expense (R15 §12.2).
- Mitigations for the global-index cost (§3.1), in order, to try only if the benchmark needs them:
  1. Add a company token to the indexed text (for example `zt000042zt`) and require it with `+` in boolean mode, while still keeping `tenant_id = ?` as the real guard. **Estimate**: it narrows candidates inside the full-text engine.
  2. Split companies across a fixed number of projection tables (manual sharding; partitioning is not allowed with FULLTEXT [S5]).
  3. Move to Meilisearch.

### 4.5 Query parser

- **Plain words** become required terms in boolean mode (`+meralco +march*`).
  - The last word gets a `*` prefix for type-as-you-go.
  - Strip or escape characters that are boolean operators.
  - If nothing matches, retry once with the words optional (OR) and say "showing partial matches". Both queries are filtered, so there is no leak.
- **Document-type aliases** (D-023): "OR", "CR", "AR", "2307", "GIS" map to Document type filters, shown as removable chips. This also covers the fact that MySQL drops "OR" (§3.1).
- **Identifier-like input** (digits with `-`, `/`, spaces, at least 6 digits) goes to the identifier table (§4.6) **and** to full text as a quoted phrase.
- **Month and year words** ("March", "Marso", "2026") stay as text terms in the MVP. A "Date" filter chip is offered instead of guessing.

### 4.6 Identifier table

- `document_identifiers` (`tenant_id`, `document_id`, `version_id`, `kind`, `raw`, `normalised`, `source`).
  - `normalised` is digits and letters only, upper case. `123-456-789-000` becomes `123456789000`.
  - `kind` is TIN, invoice number, OR number, case number or form number.
  - `source` is `details` or `ocr`.
- **Sources:**
  - detail fields typed as identifiers (Document type configuration);
  - regex extraction from OCR text, for example a TIN pattern of 3-3-3 digits with an optional branch code. The exact BIR format is to be confirmed (**Open question**).
- Lookups go through a B-tree index on `(tenant_id, normalised)` with exact or prefix match. They pass through the **same access filter** as §4.4.
- **OCR confusions** (`O`↔`0`, `l`/`I`↔`1`, `S`↔`5`) are mapped only inside digit runs when building `normalised`. **Estimate**: this improves recall on receipts; it must be measured in §7.
- This table works the same under any engine.

### 4.7 Snippets and highlighting (MySQL)

- **After** the filtered query returns a page of 20 Document IDs, load their `version_page_texts` for the searchable Version. Find the query terms (case-insensitive, accent-folded) and cut a window of about 25 words around the best hit. Return the page number.
- **Escape** all text, and mark hits with offsets rather than HTML. React renders the `<mark>` tags itself. OCR text is untrusted input (XSS).
- The cost is reading about 20 × 15 KB of text per page of results. **Estimate**: negligible.
- Never build snippets, "did you mean" or suggestions from rows outside the filtered set (D-056).

### 4.8 Language handling

- **Collation.** Use `utf8mb4` with an accent-insensitive collation such as `utf8mb4_0900_ai_ci`, so *Parañaque* is found by *Paranaque*. **Estimate**: confirm that the full-text comparison follows the collation, in the §7 test.
- **Stopwords and short words.**
  - Start with the MySQL defaults (minimum 3) for the pilot.
  - Test minimum **2** with a custom stopword table (default English list **minus** `or`, **plus** Filipino particles *ang*, *ng*, *mga*, *sa*, *na*, *ay*, *si*, *ni*, *kay*) in the §7 benchmark.
  - The setting is server-wide and needs a restart and a rebuild of every FULLTEXT index (**check Officetul's other FULLTEXT indexes, §10**).
- **Taglish synonyms.** A small curated table expands query words in Laravel: *resibo* → receipt; *kasunduan* → contract/agreement; *bayad* → payment; *upa* → rent/lease.
  - It is per product, not per company, so no company data leaks.
  - Meilisearch can later take the same list as `synonyms`.
- **Affixed Filipino forms** (*binayaran* from *bayad*) are not handled by any engine (§3). Record them as a known gap and measure them in §7.

---

## 5. Keeping the index in sync

### 5.1 MVP (MySQL)

The projection table is in the **same database**, so most changes update it **in the same transaction** as the change itself. That gives the strongest consistency possible.

| Event | Projection change | Label |
|---|---|---|
| Details edited (D-043) | Update `title` / `details_text` of the Document's rows, same transaction | Recommendation |
| OCR finished (job) | Insert or update the Version's row and page texts, and extract identifiers. Then run a **re-check** that the Version is still in a searchable state | Recommendation |
| Marked Official / Superseded / Withdrawn / restored (D-038) | Update `searchable_from` for the affected Versions, same transaction | Recommendation |
| Moved to another Folder | Update `folder_id`; the access join is live | Recommendation |
| Folder moved, Restricted switched, grants changed | **Nothing in the projection.** `acl_folder_id` and grants are joined live, so the effect is immediate (ASVS 8.3.2, R15 §6) | Recommendation |
| To Trash | `in_trash = 1`, same transaction | Recommendation |
| Permanent delete (D-041) | Delete rows, page texts and identifiers. Queue a nightly `OPTIMIZE TABLE` with `innodb_optimize_fulltext_only=ON`, because deleted words stay in the FULLTEXT index until then [S8] | Verified fact (index behaviour); Recommendation |
| Company offboarding | Delete by `tenant_id`, then `OPTIMIZE` as above. Backups follow the backup retention policy (D-033 export first) | Recommendation |

- **No Scout observers** (D-063 logic). A `DocumentSearchProjector` service is called explicitly by the Documents domain actions.
- A **nightly reconciliation job** per company compares a hash of the projection with the source tables and repairs drift. It runs per company, using the fail-closed guard (D-052).

### 5.2 Later (external engine): outbox and stale set

- **Outbox.** Every change in §5.1 also writes a `search_outbox` row (`tenant_id`, `document_id`, `reason`, `created_at`) **in the same transaction**. A worker drains it and re-reads the **current** state from MySQL, so the order of events does not matter. Upserts are idempotent.
  - This is the transactional outbox pattern: "store the message in the database as part of the transaction"; a relay may publish more than once, so consumers must be idempotent. **Verified fact** [S44]
  - Laravel's `after_commit` / `afterCommit()` avoids jobs running before the commit [S43], but a job can still be lost if Redis is down after the commit. The outbox table cannot be lost that way. **Verified fact** [S43]; comparison **Estimate**
- **What the external index stores:** `tenant_id`, `document_id`, `version_id`, `folder_id`, `acl_folder_id`, `searchable_from`, `document_type_id`, dates, `in_trash`, and the text fields. **Never user or role lists** (R15 §6).
- **Stale set for exact access (D-056).** When an action changes who may see a Document in a way the external index has not caught up with, the same transaction adds the affected Document IDs to `search_stale` (with the company). Such actions are:
  - Trash or Withdraw;
  - a move into a Restricted folder;
  - a folder move that changes `acl_folder_id`;
  - permanent delete.

  Every external search adds `AND document_id NOT IN [stale IDs for this company]`. The worker clears entries once the engine confirms the task.
  - Effect: during the lag a Document may be **missing** from someone's results (an under-count). It is never **shown or counted** to someone who lost access.
  - The list stays small because entries live only for seconds to minutes. **Recommendation**
  - A large subtree move (thousands of Documents) puts a single `acl_folder_id` into a "stale folder" list instead (`acl_folder_id NOT IN [...]`). **Recommendation**
- **Belt and braces.** Each displayed hit is re-checked with the MySQL policy before rendering. A mismatch is logged as a **security event** (Security log, D-060) and triggers a re-index. It does not silently pass.
- **Rebuild.** A full re-index from MySQL per company must be scriptable and timed. This is the substitute for paid replication (§3.2).
- **LMDB keeps freed pages** [S17]. After offboarding, deleted text may persist in the index file until the index is rebuilt. Plan a periodic "dump and rebuild" before promising certified deletion (next release, D-033). **Estimate**

---

## 6. Sizing estimates

**Assumptions (all Estimate):**
- 100 companies × 50,000 Documents = **5,000,000** Documents, with one searchable Version each (drafts add more).
- 5 pages × 500 words = **2,500 words** per Document. Receipts have far fewer and contracts more.
- About 6 bytes per word (5 letters + separator) plus 10% OCR noise, so 2,500 × 6 × 1.1 ≈ **16.5 KB** of OCR text. Add about 1 KB of title and details: **≈ 17 KB per Document**.
- Raw searchable text: 5,000,000 × 17 KB ≈ **85 GB**.
- Second scenario, a realistic first year: 20 companies × 5,000 Documents = 100,000 Documents → **≈ 1.7 GB**.

| Engine | Disk (5M Docs) | RAM (5M Docs) | Year 1 (100k Docs) | Arithmetic and basis | Label |
|---|---|---|---|---|---|
| **MySQL FULLTEXT** | Text 85 GB + FULLTEXT aux tables, assumed 0.5–1.5× text (43–128 GB). **≈ 130–215 GB** | Buffer pool for the hot working set, not the whole index. Guess **16–32 GB** for the DB server, shared with Officetul | ≈ 3–4 GB disk; fits any existing DB server | Aux-table ratio is a guess (positions stored per word [S8]); no primary figure found | **Estimate** |
| **Meilisearch** | Ratio unknown for long text ("no reliable way to predict" [S17]). Assume 2–5× text: **170–425 GB** on NVMe | 1/3–1/10 of disk [S17]: **17–140 GB** | Disk 3.4–8.5 GB; RAM about 1–3 GB | 85 GB × 2–5; then ÷3 to ÷10 | **Estimate** (wide; must measure) |
| **Typesense** | ≥ raw size (85 GB) [S30] | 2–3 × 85 GB = **170–255 GB per node**, × 3 nodes = 510–765 GB | RAM 3.4–5.1 GB per node | Vendor rule [S30] applied to OCR text | **Estimate** |
| **OpenSearch** | Comparable to Meilisearch, plus replicas | Heap ≤ 50% of RAM, plus file cache [S39] | Smallest cluster still 3 nodes for HA | — | **Estimate** |

**Reading the table:**
- At year-1 scale, every engine is cheap, so the choice is about **effort and exactness**, not hardware.
- At 5M Documents, MySQL is cheapest on paper. Its real risk is **query speed** on common words across one global index (§3.1), not size.
- Meilisearch is moderate if the 1/10 RAM ratio holds. Typesense is expensive because OCR text must sit in RAM.
- **Measure before choosing for scale.** Load 1M synthetic Documents (generated OCR-like text plus real-looking Filipino words) into each candidate on the production VM class, then record disk, RAM, p50/p95 latency for common and rare terms, and indexing throughput. **Recommendation**

---

## 7. Search quality: how to test

All **Recommendations**, building on R12 §6 and D-050.

1. **Reuse the OCR benchmark set** (240 pages, D-050) and its **keyword-hit rate**. Run the query list against the **real** search stack (projection + parser + MySQL), not against raw OCR text.
2. **Query list per stratum.** For each, record the expected Documents:
   - **exact words** ("Meralco", "kasunduan");
   - **typo'd words** ("Meralko", "kasundaun");
   - **Taglish** ("resibo Meralco", "bayad upa");
   - **affixed Filipino** ("binayaran");
   - **identifiers in three forms** (`123-456-789-000`, `123456789000`, `123 456 789`);
   - **aliases** ("OR 4521", "2307 Q1");
   - **phrases** ("Deed of Absolute Sale").
3. **Metrics:**
   - recall@10 (expected Document in the top 10);
   - **zero-result rate**;
   - mean reciprocal rank;
   - p50/p95 latency.

   Report them per stratum, as in R12 §6.4, because receipts and phone photos get their own bars.
4. **Configurations to compare:**
   - MySQL with minimum token size 3 and default stopwords;
   - MySQL with minimum 2 and the custom stopword list;
   - MySQL with the identifier table on and off;
   - Meilisearch defaults;
   - Meilisearch with `disableOnNumbers` and synonyms.

   The difference in typo'd-query recall between MySQL and Meilisearch is the evidence for the switch trigger (§8).
5. **Known OCR-noise effects to look for:**
   - split words ("Meral co");
   - merged words ("MeralcoInc");
   - `0/O`, `1/l/I`, `5/S` swaps inside numbers;
   - line-end hyphenation;
   - garbage tokens on photos (R12 [S42]: Tesseract emits extra text on noise).

   Garbage inflates the index and IDF. Consider dropping tokens on pages whose mean OCR confidence is below a threshold (R12 §5). **Estimate**
6. **Leak tests.** These are part of the D-052 release gate, not quality scoring:
   - A word that exists **only** in a hidden Document (other company, Restricted folder, Confidential folder, Draft for a View user, Trash) must give the **same** results, total, facets, suggestions and empty state as a nonsense word.
   - Repeat after a folder move and after a Restricted switch, with no wait. This tests the live join or the stale set.
7. **Calibration.** The owner marks "good enough to find this document" on 10 live searches, as in R12 §6.4 step 2. No pass numbers are proposed here.

---

## 8. Recommended path and trade-off

**Recommendation: MVP on MySQL FULLTEXT behind the Search port. Prepare (but do not run) Meilisearch Community Edition as the next adapter.**

**Why MySQL first:**
- **Exactness for free.** The access filter is a live SQL join and the projection updates in the same transaction. Permission changes, Trash and Withdraw take effect on the next query, with no stale set and no lag (D-056).
- **No new stateful service** to secure, back up, monitor, upgrade and include in isolation tests (D-005, D-052).
- **Pilot scale is small** (D-017: own company, then one client), so speed risk is low there.

**What we give up:**
- No typo tolerance.
- Snippets are built in PHP.
- Weaker relevance on long OCR documents.
- A common-word speed risk at large scale.

**Why not Meilisearch first:**
- It is faster to a polished feel (typos, highlighting).
- But it brings sync and the stale set, a 65,535-position cap (needing per-page rows), no free replication, and one more service to run, all before a pilot proves search is the pain point.

**Switch triggers.** Move search to Meilisearch when **any** of these holds over two consecutive weeks of real use. The numbers are placeholders for the owner's calibration (Decision 7):

| Trigger | Measure | Label |
|---|---|---|
| Speed | p95 search time above **1.5 s** on production data, after the §4.4 mitigations | Recommendation (placeholder) |
| Typo pain | Zero-result rate above **10%** of searches, **and** the §7 test shows Meilisearch recovers most of them | Recommendation (placeholder) |
| Scale | Total searchable text above about **20 GB**, or `OPTIMIZE` windows disturbing other Officetul modules | Estimate |
| Product | A paying client requires search-as-you-type with typo tolerance | Recommendation |

The zero-result rate is counted as a number per company and day, **without storing search terms** (D-060).

**Switch cost (Estimate):** 2–4 developer-weeks. That covers the adapter, outbox worker, stale set, per-page Search rows, the contract tests passing on both adapters, a full rebuild script, and ops (backup, monitoring, `--no-analytics`). The cost stays low only if §4 is built as described.

---

## 9. Recommendations summary

| MVP | Next release / later |
|---|---|
| `DocumentSearch` port; MySQL FULLTEXT adapter; contract tests shared by future adapters | Meilisearch CE adapter (self-hosted, `--no-analytics`), when §8 triggers fire |
| Projection table with explicit `FTS_DOC_ID`; one FULLTEXT index; OCR capped per Version; per-page text table | Per-page Search rows + `distinctAttribute` for Meilisearch |
| Live access join (`acl_folder_id`, grants, link grants) in the same `WHERE` as `MATCH`; counts and facets from the same `WHERE` | Outbox + stale set + per-hit MySQL re-check |
| Searchable Versions by Access level (Decision 2); Trash excluded (Decision 3) | "Include history" search for Contribute |
| Query parser: AND by default, prefix on last word, type aliases as chips, identifier detection | Date phrases ("March 2026") understood as a date filter |
| Identifier table with normalised TINs and invoice/OR numbers from details and OCR (Decision 4) | Per-company custom identifier patterns |
| Snippets built in PHP from permitted results, escaped, with page number | Highlight boxes on the page image (hOCR/ALTO, R12 §5) |
| Curated Taglish synonym list (product-wide) | Meilisearch `synonyms`; Filipino affix handling if the benchmark shows need |
| Nightly `OPTIMIZE` after deletes; nightly reconciliation per company | Index "dump and rebuild" before certified deletion (D-033) |
| No search terms stored; only counts of zero-result searches (D-060) | — |
| §7 search test on the D-050 benchmark set, including leak tests, before the first external client | 1M-Document load test before passing about 1M Documents |

---

## 10. Check in the Officetul code (when access is granted, D-013)

1. **MySQL:**
   - Version (8.0, 8.4 or 9.x) and hosting (RDS parameter groups, or self-managed `my.cnf`).
   - Current `innodb_ft_min_token_size`, stopword settings and `innodb_ft_*` cache values.
   - Default charset and collation.
2. **Existing FULLTEXT indexes** in other modules. A server-wide token-size or stopword change rebuilds and changes **them** too.
3. **Scout:** installed? Which version and driver? Are `queue` and `after_commit` on? Do any models use `Searchable`?
4. **Existing search UI** (global search bar). Can Documents plug into it, and how does it filter by company today?
5. **Tenancy** (R15 §13.1): how `tenant_id` is applied, and whether global scopes exist. These would **not** apply to an external engine [S40].
6. **Record access** for link-granted visibility: can "who may open Expense #881" be expressed as SQL, or only in PHP policies?
7. **Queues:** driver (Redis or database), `after_commit` setting, worker hosting. These are needed for the OCR job and later the outbox.
8. **Server size** of the production DB (RAM, buffer pool, disk type), and whether a separate replica exists that search could read from.
9. **Logging:** do request logs or error trackers capture query strings (`?q=`)? D-060 forbids storing search terms.
10. **Front end:** an existing table or list component with facets and pagination in Inertia/React.

---

## 11. Open questions

1. How fast is MySQL FULLTEXT with `tenant_id` filtering on 1M–5M rows of OCR-like text on our VM class? No published figure was found. The §6 load test is needed.
2. How does the built-in parser tokenise `123-456-789-000`, `2550Q`, `OR#4521` and `P1,250.00`? Test on the target MySQL version.
3. Does MySQL's full-text matching follow an accent-insensitive collation (*ñ* vs *n*) in practice?
4. Minimum token size 2 plus Filipino stopwords: what is the index size cost and the recall gain? (§7 configuration test)
5. Meilisearch: how fast is a filter with a long `IN [...]` list (hundreds of `acl_folder_id` values, thousands of link-granted Document IDs)? No limit or guidance is documented.
6. Is MySQL 9.7 an LTS release, and does Officetul plan to move from 8.4? Not confirmed on the pages read.
7. The exact BIR TIN format (9 digits plus 3- or 5-digit branch code) for the identifier pattern. To confirm with the tax adviser (D-027) or BIR material.
8. Link-granted visibility (D-053): if Officetul's record access is per record rather than per role, the link-grant list could be large for both engines. This depends on R15 Open question 2.
9. Meilisearch replication is Enterprise-only from v1.37 (search excerpt). What does an EE licence cost for a single small cluster? No price is published ("contact sales").
10. Whether typo tolerance matters to pilot users at all. Measure the zero-result rate in the pilot before paying for it.

---

## 12. Decisions for the owner

Each is worded so that "yes" accepts the recommendation.

1. **Start with MySQL's built-in full-text search, behind a switchable search layer, and move to Meilisearch only when measured triggers fire?**
   - *Recommended:* yes. No new server to run, and access checks are always exact and immediate.
   - *Example:* Maria types "Meralco March". MySQL finds every Document in folders Maria may see that contains both "Meralco" and words starting "March", newest first, with a short excerpt showing "page 2: …Meralco billing period **March** 2026…". If Maria types "Meralko" (a typo), the MVP finds nothing and says so. If that happens a lot, it is our signal to switch.

2. **Viewers search only Official versions (and details); Contribute and Manage holders also search Drafts; old and Withdrawn versions are searched only with an "Include history" switch?**
   - *Recommended:* yes. This matches D-053.
   - *Example:* a draft lease in *Admin › Leases* says "rent ₱85,000" but the Official version says "₱80,000". Jun has View only, so searching "85,000" finds nothing. Ana in Admin has Contribute and finds the draft, marked **Draft**.

3. **Documents in Trash never appear in normal search; Manage holders get a separate "Search Trash"?**
   - *Recommended:* yes.
   - *Example:* Carla trashes a duplicate BIR 2307. Searching "2307 ABC Trading" no longer shows it to anyone. The folder's manager can still find it under *Trash › Search* and restore it.

4. **Treat TINs, invoice numbers and OR numbers as identifiers, found whether typed with or without dashes or spaces?**
   - *Recommended:* yes, via a small identifier table filled from details and OCR text.
   - *Example:* Ben types `123456789000`. The supplier invoice whose OCR says "TIN: 123-456-789-000" is found, and so is the one where the TIN was typed into the details as "123 456 789 000". Typing "OR 4521" filters to Payment Receipts and looks for receipt number 4521.

5. **No "Did you mean…?" suggestions in the MVP, and none ever built from words in documents the user cannot see?**
   - *Recommended:* yes. A suggestion built from company-wide words could reveal a hidden document (D-056).
   - *Example:* HR's confidential folder holds a letter about "retrenchment". If Maria mistypes "retrenchmnt", a company-wide suggestion "Did you mean retrenchment?" would hint that such a document exists. With this rule, Maria just sees "No results".

6. **Search terms are never stored; we keep only daily counts of searches and of searches that found nothing, per company?**
   - *Recommended:* yes (extends D-060). The counts drive the switch trigger without logging what anyone looked for.
   - *Example:* an HR officer searches an employee's name. Nothing records that name. Ops only see "Company X: 140 searches today, 12 with no results".

7. **Accept the switch triggers in §8 as starting values, to be calibrated with real pilot data?**
   - *Recommended:* yes. The triggers are p95 above 1.5 s, more than 10% zero-result searches that Meilisearch would fix, more than about 20 GB of text, or a paying client's demand.
   - *Example:* after three months, the pilot's zero-result rate is 14%, and the §7 test shows most failures are typos such as "Meralko". That meets the trigger, so we schedule the 2–4 week Meilisearch switch.

8. **Run the search test (typos, Taglish, IDs in different formats, hidden-document leak checks) on the OCR benchmark set before the first external client?**
   - *Recommended:* yes. It extends D-050 and reuses its 240 pages.
   - *Example:* the test types "resibo Meralco" and "binayaran" and records whether the right receipts appear in the top 10. It also searches a word that exists only in a Confidential folder, and checks that the screen looks exactly like a search for "xyzzy".

---

## 13. Statement of limits

- This note does not fix the database schema, hosting or prices. Names in SQL sketches are placeholders until the Officetul code is reviewed (D-013).
- Size and speed numbers are estimates with stated arithmetic; none is a benchmark.
- Licence readings are research, not legal advice. GPL and BUSL points can go to the D-027 lawyer session if Typesense or Meilisearch Enterprise is ever considered.

---

## 14. Sources (all accessed 2026-10-20)

| ID | Source | URL | Type |
|---|---|---|---|
| S1 | MySQL 8.4 Full-Text Search Functions | https://dev.mysql.com/doc/refman/8.4/en/fulltext-search.html | Primary |
| S2 | MySQL 8.4 Natural Language Full-Text Searches | https://dev.mysql.com/doc/refman/8.4/en/fulltext-natural-language.html | Primary |
| S3 | MySQL 8.4 Boolean Full-Text Searches | https://dev.mysql.com/doc/refman/8.4/en/fulltext-boolean.html | Primary |
| S4 | MySQL 8.4 Full-Text Stopwords | https://dev.mysql.com/doc/refman/8.4/en/fulltext-stopwords.html | Primary |
| S5 | MySQL 8.4 Full-Text Restrictions | https://dev.mysql.com/doc/refman/8.4/en/fulltext-restrictions.html | Primary |
| S6 | MySQL 8.4 Fine-Tuning Full-Text Search | https://dev.mysql.com/doc/refman/8.4/en/fulltext-fine-tuning.html | Primary |
| S7 | MySQL 8.4 ngram Full-Text Parser | https://dev.mysql.com/doc/refman/8.4/en/fulltext-search-ngram.html | Primary |
| S8 | MySQL 8.4 InnoDB Full-Text Indexes | https://dev.mysql.com/doc/refman/8.4/en/innodb-fulltext-index.html | Primary |
| S9 | MySQL 8.4 InnoDB Startup Options and System Variables | https://dev.mysql.com/doc/refman/8.4/en/innodb-parameters.html | Primary |
| S10 | MySQL 9.7 release notes index; "What Is New in MySQL 9.7" | https://dev.mysql.com/doc/relnotes/mysql/9.7/en/ ; https://dev.mysql.com/doc/refman/9.7/en/mysql-nutshell.html | Primary |
| S11 | Oracle MySQL blog, hybrid semantic and keyword search in HeatWave (search excerpt) | https://blogs.oracle.com/mysql/post/hybrid-semantic-keyword-search-in-mysql-heatwave | Secondary |
| S12 | PostgreSQL 18, Controlling Text Search (`ts_rank`, `ts_headline`) | https://www.postgresql.org/docs/current/textsearch-controls.html | Primary |
| S13 | GitHub REST API: releases of meilisearch/meilisearch (v1.55.0), meilisearch/meilisearch-php (v1.17.0), meilisearch/charabia (v0.10.0) | https://api.github.com/repos/meilisearch/meilisearch/releases | Primary (project) |
| S14 | Meilisearch `LICENSE` and `LICENSE-EE` | https://github.com/meilisearch/meilisearch/blob/main/LICENSE ; https://github.com/meilisearch/meilisearch/blob/main/LICENSE-EE | Primary (project) |
| S15 | Meilisearch Enterprise Edition | https://www.meilisearch.com/docs/resources/self_hosting/enterprise_edition | Primary |
| S16 | Meilisearch known limitations | https://www.meilisearch.com/docs/resources/help/known_limitations | Primary |
| S17 | Meilisearch storage | https://www.meilisearch.com/docs/resources/internals/storage | Primary |
| S18 | Meilisearch language support | https://www.meilisearch.com/docs/learn/resources/language | Primary |
| S19 | Meilisearch typo tolerance settings | https://www.meilisearch.com/docs/learn/relevancy/typo_tolerance_settings | Primary |
| S20 | Meilisearch search API | https://www.meilisearch.com/docs/reference/api/search | Primary |
| S21 | Meilisearch filter expression reference | https://www.meilisearch.com/docs/learn/filtering_and_sorting/filter_expression_reference | Primary |
| S22 | Meilisearch settings API | https://www.meilisearch.com/docs/reference/api/settings | Primary |
| S23 | Meilisearch data types (separators) | https://www.meilisearch.com/docs/learn/engine/datatypes | Primary |
| S24 | Meilisearch tenant tokens | https://www.meilisearch.com/docs/learn/security/tenant_tokens | Primary |
| S25 | Meilisearch telemetry | https://www.meilisearch.com/docs/learn/resources/telemetry | Primary |
| S26 | Meilisearch delete documents by filter; v1.2 release (search excerpts) | https://www.meilisearch.com/docs/reference/api/documents/delete-documents-by-filter.md ; https://github.com/meilisearch/meilisearch/releases/tag/v1.2.0 | Secondary |
| S27 | Meilisearch pricing | https://www.meilisearch.com/pricing | Vendor |
| S28 | Charabia crate (Latin pipeline; search excerpt) | https://docs.rs/crate/charabia/0.10.0 | Secondary |
| S29 | Typesense repository: licence GPL-3.0, releases (v30.2), README "Why the GPL license?" | https://github.com/typesense/typesense | Primary (project) |
| S30 | Typesense system requirements | https://typesense.org/docs/guide/system-requirements.html | Vendor |
| S31 | Typesense data access control (scoped keys) | https://typesense.org/docs/guide/data-access-control.html | Primary |
| S32 | Typesense locale guide | https://typesense.org/docs/guide/locale.html | Primary |
| S33 | Typesense 30.1 search API | https://typesense.org/docs/30.1/api/search.html | Primary |
| S34 | Typesense 30.1 collections API | https://typesense.org/docs/30.1/api/collections.html | Primary |
| S35 | Typesense tips for searching common types of data | https://typesense.org/docs/guide/tips-for-searching-common-types-of-data.html | Primary |
| S36 | OpenSearch repository (Apache-2.0; release 3.9.0) | https://github.com/opensearch-project/OpenSearch | Primary (project) |
| S37 | OpenSearch document-level security | https://docs.opensearch.org/latest/security/access-control/document-level-security/ | Primary |
| S38 | Elasticsearch `LICENSE.txt`; releases (v9.5.5) | https://github.com/elastic/elasticsearch/blob/main/LICENSE.txt | Primary (project) |
| S39 | Elasticsearch JVM settings | https://www.elastic.co/docs/reference/elasticsearch/jvm-settings | Primary |
| S40 | Laravel 13.x Scout | https://laravel.com/docs/13.x/scout | Primary |
| S41 | laravel/scout 11.x source: `config/scout.php`, `src/Engines/DatabaseEngine.php`, `composer.json`; release v11.9.0 | https://github.com/laravel/scout/tree/11.x | Primary (source) |
| S42 | Laravel framework 13.x `MySqlGrammar::whereFullText` | https://github.com/laravel/framework/blob/13.x/src/Illuminate/Database/Query/Grammars/MySqlGrammar.php | Primary (source) |
| S43 | Laravel 13.x Queues (jobs and database transactions, unique jobs) | https://laravel.com/docs/13.x/queues | Primary |
| S44 | Transactional outbox pattern (microservices.io) | https://microservices.io/patterns/data/transactional-outbox.html | Secondary (reference) |
| S45 | R15 Tenant isolation and permissions (§5.2, §6) | research/R15-tenant-isolation-and-permissions.md | Internal |
| S46 | R12 OCR engines and benchmark (§5, §6) | research/R12-ocr-engines-and-benchmark.md | Internal |
| S47 | R14 Activity log and integrity (§3, model events) | research/R14-activity-log-and-integrity.md | Internal |
