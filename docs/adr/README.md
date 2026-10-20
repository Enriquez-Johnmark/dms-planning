# Architecture decision records — Officetul Documents

Short records of decisions that are **hard to reverse**, **surprising without context**, and **the result of a real trade-off**. The full architecture is in [R21](../../research/R21-architecture-proposal.md). Confirmed decisions themselves live in [DECISION-LOG.md](../../DECISION-LOG.md); an ADR explains one or more of them and never overrides the log.

| ADR | Title | Decisions | Status |
|---|---|---|---|
| [0001](0001-native-module-in-shared-database.md) | Documents is a native Officetul module in the shared database | D-002, D-007, D-010, D-075 | Accepted 2026-10-21 |
| [0002](0002-mysql-fulltext-behind-search-port.md) | MySQL FULLTEXT search first, behind a search port | D-064 | Accepted 2026-10-21 |
| [0003](0003-fail-closed-company-guard-and-composite-keys.md) | Fail-closed company guard plus composite company keys | D-052, D-075 | Accepted 2026-10-21 |
| [0004](0004-hash-chained-activity-log-with-locked-digest.md) | Hash-chained Activity log with a locked daily Log digest | D-060–D-063 | Accepted 2026-10-21 |
| [0005](0005-locked-cross-region-backups-and-purge-ledger.md) | Locked backups in a second region, with a purge ledger and restore marker | D-079–D-085 | Accepted 2026-10-21 |
| [0006](0006-folders-only-access.md) | Folders alone grant access; no per-Document permissions | D-053, D-093 | Accepted 2026-10-21 |

## Conventions

- File name: `NNNN-short-slug.md`, numbered in order of writing.
- Sections: Title, Status (with date and decision IDs), Context, Decision, Consequences (good and bad), Alternatives considered. Under about 40 lines.
- Terms follow [GLOSSARY.md](../../GLOSSARY.md).
- A changed decision gets a new ADR that marks the old one "Superseded by NNNN"; old ADRs are not deleted.
- "Accepted" means the project owner confirmed the underlying decisions. Senior partner approval of scope (D-089) and the Officetul code review (D-013) may still change ADRs 0001 and 0003.
- The decision log's header still names `appendices/architecture-decisions.md` as the ADR home; this folder replaces it.
