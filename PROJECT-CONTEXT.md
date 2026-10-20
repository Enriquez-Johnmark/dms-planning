# Project Context

Baseline as of 2026-10-18. Update this file when facts change, and link to the decision or source that changed them.

## People

| Role | Responsibility in discovery |
|---|---|
| Project owner | Drives discovery, reviews documents, relays questions to the senior partner, owns technical decisions. |
| Senior partner | Business decision-maker. Approves problem, customer, scope, and budget. **Owns the ISO decision** (D-028). |
| Claude | Researches, drafts documents, runs grilling rounds, keeps the trackers current. Does not decide. |

## What we know

| Item | Label |
|---|---|
| The owner has an existing ERP built with Laravel, Inertia.js, React, and MySQL. | Verified fact (stated by owner) |
| The ERP repository, schema, data, server, API docs, and deployment are **not** available to this project. | Verified fact (stated by owner) |
| **ISO certification is a future goal.** Which standard and when is **the senior partner's decision and does not block discovery** (D-028). The Documents app is built ISO-ready in the meantime (D-011). | Verified fact (stated by owner) |
| Candidate segments: Philippine law firms, accounting/bookkeeping firms, construction and project-based firms, other document-heavy organisations. | Verified fact that these are *candidates*; none is a confirmed customer |
| The discovery workspace started empty on 2026-10-18. No prior research exists in it. | Verified fact |
| The DMS will be sold as a module of the ERP. The senior partner sells the ERP as a one-stop shop. | Decision D-002 |
| The ERP's clients are SMEs across industries (law, accounting, construction, others), and most work heavily on paper. | Owner's observation (D-003, D-006); not measured |
| The owner describes the goal as "digitising" clients' paper documents. The owner wants both an archive and document control (Round 2 Q2). | Owner's statement |
| The ERP is called **Officetul**. It is **multi-tenant**, built on Laravel with Inertia and React, and organised as apps (Accounting, Inventory, …) in the style of Odoo or ERPNext. | Verified fact (owner, Round 2 Q7) |
| **Repository access comes only after discovery, planning and design are complete.** How tenancy is implemented (shared database with a tenant column, a database per tenant, a package) is unknown. | Verified fact (owner); a major design risk |
| Which ISO standard and when: the senior partner decides ([Section 04](sections/04-iso-certification-strategy.md)); the app stays ISO-ready (D-011). | Future goal, owned by the senior partner (D-028) |
| The idea of scanners uploading straight into the DMS raised concerns: who supplies scanners, and who maintains and secures any "script". | Owner's statement (Round 2 Q3) |

## What we are assuming for now

| Assumption | Why it matters | How it gets validated |
|---|---|---|
| ~~MySQL is retained for the DMS.~~ **Now decided (D-074):** MySQL, 8.4 LTS minimum. | Avoids a second database platform for a small team. | Phase 9 comparison done (R17 §8). |
| A small development team (size unknown) will build and run the product. | Shapes every complexity and hosting choice. | Round 1 grilling. |
| ~~The DMS might integrate with the ERP, but must not depend on sharing its database.~~ **Superseded:** Documents is a native Officetul app sharing its database (D-002, D-010), with a published contract for other apps (D-075, Section 16). | — | Phase 9 integration contract done. |
| Philippine SMEs have limited IT budgets and staff. | Drives pricing, hosting, and admin burden. | R01 research + customer interviews. |

All of these are **Working assumptions**. None is a decision.

## What we don't know (highest impact first)

1. Who the first paying customer is: the partner's own firm, the ERP's existing customers, or new SMEs.
2. What business problem is painful enough that someone pays to fix it.
3. ~~What "ISO certification" means to the senior partner~~: handed to the senior partner, not blocking (D-028).
4. Whether any customer conversations have already happened.
5. Budget, timeline, and team size.
6. Who the ERP's users are and whether they would be the first DMS customers.

Full list: [OPEN-QUESTIONS.md](OPEN-QUESTIONS.md).

## What we must not assume

- That building a new DMS is better than adopting or extending an existing one.
- That law, accounting, and construction firms need the same product.
- That a DMS can make an organisation ISO-certified or legally compliant.
- Anything about the ERP's code, schema, data model, users, or hosting.
- That multi-tenancy, OCR, AI, or a separate search engine is needed.
- That the latest upload is the authoritative version of a document.
- That a file hash proves who signed a document, whether it is true, or whether it is legally valid.
- That any customer interview has happened. None has been recorded.
