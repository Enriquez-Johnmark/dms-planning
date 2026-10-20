# Meeting Guide — Costs and Pricing for Documents

**Duration:** about 30–45 minutes. **Detail:** [Section 18](../sections/18-cost-estimates-and-operating-model.md), [Section 14](../sections/14-storage-and-server-requirements.md), [Section 15](../sections/15-backup-retention-and-disaster-recovery.md). **Follows:** [meeting guide 02](02-erp-document-apps-and-pricing.md), where pricing was deferred until costs were known (D-018).

## Meeting Objective

Decide **how Documents is priced**: included in the Officetul subscription, as an add-on, or a mix of both, and what the storage and OCR allowances are.

## Why This Matters to the Business

Documents has a **fixed running cost** of about ₱13,000–21,000 a month (**Estimate**), whether two clients use it or twenty. Pricing must cover that floor, or Documents loses money until there are enough clients.

## Plain-Language Explanation

- **What it costs us.** About ₱900–1,270 per client company per month, including our time to run it, at year-1 scale (20 companies). Most of that is one processing server (virus scanning, previews, OCR) and about 12–16 hours a month of looking after it.
- **What grows.** Storage. Clients never automatically delete documents, so a client's storage only grows. Users cost us almost nothing.
- **What competitors charge:**

  | Competitor | Price |
  |---|---|
  | Odoo | about ₱565–707 per user per month, Documents included |
  | Dext | about ₱1,590 a month |
  | Juan | ₱2,000 a month for its whole accounting product |
  | Xero, Zoho | file storage included |
- **Break-even for a flat add-on.** A flat add-on per company breaks even at about:

  | Add-on price | Companies needed |
  |---|---|
  | ₱1,000 | 24 |
  | ₱1,500 | 16 |
  | ₱2,000 | 11 |

  These are **Estimates**, using a placeholder ₱600 an hour for our time.
- **Hosting and safety.** Hosting is in Singapore (about 30 ms from Manila). Backups go to a separate, locked account in a second region, and restore drills happen every quarter. Clients can be told this plainly.

## Questions to Discuss

1. **Included or add-on?** Do we include Documents in the Officetul subscription with a storage allowance, sell it as an add-on, or include a base and sell extra storage and back-file scanning?
2. **How is Officetul priced today, and what margin do you need?** (Still unanswered from meeting 02.)
3. **Allowances:** is something like 25 GB and 3,000 OCR pages a month per company, with paid top-ups, about right?
4. **Back-file scanning** (e.g. 30 years of old files): quote it as a separate project?
5. **Our time:** what hourly rate should the cost model use for ops time?

## Options and Trade-offs

| Option | Good | Bad |
|---|---|---|
| Included, with storage allowance | Easiest to sell; matches the one-stop pitch | Cost must fit Officetul's margin |
| Flat add-on | Separate revenue line | Must be about ₱1,500–2,000 to cover costs at small scale; competitors include it |
| Per user | Familiar (Odoo) | Costs follow documents, not users |
| Included base + storage and back-file packs | Covers the floor and the growing costs | Slightly more to explain |

## Recommended Direction

**Included base plus storage and back-file packs.** The base keeps the pitch; packs cover the cost that keeps growing. The senior partner decides the numbers.

## Decisions Required

| Decision | If postponed |
|---|---|
| Pricing model | We can't finalise plan limits in the MVP scope (Section 17) |
| Allowances | The app's quota settings stay placeholders |
| Ops hourly rate | The cost model stays an estimate |

## Information Needed Before the Meeting

- Officetul's current price list and target margin.
- Rough number of clients likely to take Documents in year 1.

## Follow-up Actions

- Record the decisions in the decision log (closing OQ-020 and D-018).
- Update the allowances in Section 17.
