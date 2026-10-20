# 0005 — Locked backups in a second region, with a purge ledger and restore marker

**Status:** Accepted, 2026-10-21. Decisions D-079, D-080, D-081, D-082, D-083, D-084, D-085 (also D-073, D-098c–e).

## Context

Losing documents is worse than paper. Backups must survive stolen main-account admin keys, ransomware and the loss of the Singapore region. Yet erasure (RA 10173) and exit deletion must stick, even though no one can edit a locked backup to remove one company. A point-in-time restore also rolls back the Activity log, while the locked Log digests still prove the lost entries existed.

## Decision

- **Backups:**
  - daily copies of the database and Original files go to a **separate backup account** with a **compliance-locked vault** in a **second region** (Tokyo proposed);
  - daily copies are kept 35 days and monthly copies 120 days, and nothing in backups is older than 120 days;
  - Log digests are replicated there too, but they are not backups (10 years);
  - Derived files, quarantine and exports are not backed up.
- **Restore safety:** every permanent deletion, disposal, erasure and exit deletion writes IDs to a **purge ledger** outside the database. After every restore, the ledger is replayed before anyone logs in (except purges caused by our own bug). The Activity log continues with a **restore marker**.
- **Recovering deleted Documents:** only for "clean-up/mistake" deletions, which "may be" recoverable for up to 120 days.
- **Testing:**
  - a monthly automatic restore check;
  - a quarterly drill;
  - a yearly full rebuild by the person who didn't write the runbook;
  - the first drill before the first external client.

## Consequences

- **Good:**
  - Survives stolen keys and region loss.
  - Deleted data never returns, and the log stays honest.
  - About ₱2,200 a month in year 1 (Estimate).
- **Bad:**
  - Locks can't be shortened, so exited data lingers in backups until about day 157; the deletion certificate says so.
  - The region is named in the data processing addendum.
  - The ledger must itself survive region loss (R21 §8.2, decision 2).
  - The 2-day disaster target needs infrastructure as code (C-15).
  - Drills need two people.

## Alternatives considered

- **Same-account or same-region copies:** lost with the keys or the region.
- **Per-company keys (crypto-shredding)** for faster exit deletion: about $1–3 per company per month. Only if a contract requires it (D-059).
- **Never restoring deletions:** unhelpful for honest mistakes.
- **Always restoring them:** breaks erasure.
