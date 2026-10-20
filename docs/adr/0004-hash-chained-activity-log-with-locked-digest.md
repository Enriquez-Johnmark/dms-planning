# 0004 — Hash-chained Activity log with a locked daily Log digest

**Status:** Accepted, 2026-10-21. Decisions D-060, D-061, D-062, D-063 (also D-029, D-084).

## Context

Clients need to show who touched a record and that it hasn't changed since arrival (Rules on Electronic Evidence, Rule 7 "controls, tests and checks"). Laravel log packages offer no tamper-evidence and miss bulk updates (R14 §3). AWS QLDB ended in 2025. Anyone with database access, including Officetul staff, could rewrite plain log rows unnoticed.

## Decision

- **Two logs:** the company's Activity log, and Officetul's Security log. Neither holds document content or search terms.
- **Own small writer:** entries are written in the **same transaction** as the change.
- **No log, no access:** a download or preview whose entry can't be written is refused.
- **Protection:** the app's database user may only insert and read log rows. Blocking triggers back this up.
- **Chain:** entries are hash-chained per company by an asynchronous sealer.
- **Redactable payload:** old and new values and personal details sit in a separate payload, so they can be redacted without breaking the chain.
- **Daily Log digest:** it goes to compliance-mode Object Lock storage, kept 10 years, and is replicated to the backup account.
- **Nightly verifier:** a failed check is a security incident.
- **Restores:** after a restore, a restore marker records the lost period. Lost entries are never rewritten (D-084).
- **Claim:** the product may say "tamper-evident for changes after upload" only once the chain, the digest and the verifier are live. It never says "tamper-proof" or "immutable".

## Consequences

**Good:**
- A rewrite by anyone with database access becomes detectable.
- The cost is pennies a month per company (Estimate).
- A one-Document extract can be checked on its own.

**Bad:**
- Needs separate database users, which is Officetul platform work (D-096).
- Entries deleted in the seconds before sealing are not detected.
- Exit deletion and payload redaction need a controlled way past the insert-only rule (R21 §8.2, decision 3).
- Digests already written can never be changed or removed early.
- A partial-day gap after a restore can be checked only for internal consistency.

## Alternatives considered

- **spatie or owen-it packages:** no tamper-evidence.
- **Grants and chain without an external digest:** a full rewrite would go undetected.
- **RFC 3161 time-stamps now:** moved to the next release.
- **A managed ledger database:** none available on our stack.
