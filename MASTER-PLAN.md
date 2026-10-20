# Master Plan

**Guiding principle:** build the right product before building the product.

**Destination:** a partner-approved discovery package (problem, customer, scope, legal position, architecture, costs, risks, roadmap; ISO certification is a future goal owned by the senior partner, D-028) that is mature enough to hand to implementation. Only then is a coding-agent implementation prompt written.

## Working method

1. **One major section at a time.** Each section produces two outputs: a research and decision document (`sections/`) and a short meeting guide (`meeting-guides/`).
2. **Research before recommending.** Facts are gathered by research agents against primary sources and saved in `research/`. Each claim is cited and labelled (see [README](README.md#statement-labels)).
3. **Decisions go through grilling.** Every decision that needs the owner or partner uses the `grilling` skill: numbered rounds, each question with a recommended answer worded so "yes" accepts it. Facts are looked up, never asked.
4. **Silence is not approval.** A decision enters [DECISION-LOG.md](DECISION-LOG.md) only when the owner or partner confirms it.
5. **Gates.** A phase closes only when its exit condition is met or the project owner explicitly waives it.

## Installed skills and where they are used

Inspected 2026-10-18 (mattpocock-skills plugin v1.3.1, installed via AI Hero).

| Skill | Who can invoke | Use in this project |
|---|---|---|
| `grilling` | Claude | Every decision round with the owner/partner. Mandatory (owner's global rule). |
| `research` | Claude | Background research agents → cited notes in `research/`. |
| `domain-modeling` | Claude | From Phase 6 (lifecycle): glossary terms (document, revision, file…) and ADRs. |
| `prototype` | Claude | Optional, Phases 6–8: throwaway state-model or UI sketch to react to. |
| `codebase-design`, `tdd` | Claude | Implementation only (after Phase 13). Not used during discovery. |
| `wayfinder` | **Owner only** | Optional: could turn this plan into a decision-ticket map. Needs an issue tracker (`/setup-matt-pocock-skills`). |
| `to-questionnaire` | **Owner only** | Useful for an async questionnaire to the senior partner or interviewees. |
| `to-spec`, `to-tickets` | **Owner only** | Phase 13 handoff: spec and tracer-bullet tickets. |
| `grill-me`, `handoff` | **Owner only** | `grill-me` = start grilling yourself; `handoff` = context handover between sessions. |

"Owner only" skills are marked `disable-model-invocation`: Claude cannot start them. The owner types `/<skill-name>` to run them.

## Phases and gates

| # | Phase | Main outputs | Exit condition | Status |
|---|---|---|---|---|
| 0 | Setup and workflow | This workspace, skills inventory, tracking files | Workflow, scope, and decision process documented | **Done (pending owner review)** |
| 1 | Business problem and opportunity | Section 01 | Partner agrees on the problem statement and target-customer hypothesis, or names the evidence needed to decide | **Owner agreed (D-001 to D-017); awaiting senior partner sign-off** |
| 2 | Market and competitors *(re-scoped)* | Section 02, competitor appendix | ERP-embedded document apps understood; module pricing approach proposed | **Owner agreed (D-018 to D-022); pricing deferred to Section 18** |
| 3 | Philippine SME workflows *(re-scoped)* | Section 03, from desk research + client document profile | Prioritised document types and Officetul record links for the MVP | **Owner agreed (D-023 to D-027)** |
| 4 | ISO certification objectives | Section 04, ISO mapping | Certification-support objectives and gaps documented | **Drafted. ISO is a future goal; senior partner decides; not blocking (D-028)** |
| 5 | Legal, authenticity, records governance | Sections 10–11 | Legal assumptions, required controls, questions for a lawyer recorded | **Owner agreed (D-029 to D-034); professional review pending** |
| 6 | Lifecycle, versioning, approvals | Sections 05, 08 | Lifecycle rules agreed | **Owner agreed (D-035 to D-043)** |
| 7 | Files, upload security, OCR | Sections 06, 07, 12; file-format matrix | File-processing requirements documented | **Owner agreed (D-044 to D-051); OCR benchmark pending** |
| 8 | Security, audit, isolation | Sections 09, 12 | Critical controls and audit objectives agreed | Not started |
| 9 | Database, search, storage, ERP | Sections 13, 16 | Data and integration decisions approved | Not started |
| 10 | Infrastructure, cost, backup, DR | Sections 14, 15, 18 | Operating model is financially and operationally realistic | Not started |
| 11 | MVP scope | Section 17 | Partner approves scope and release boundaries | Not started |
| 12 | Architecture and roadmap | Sections 19–21, ADRs | Requirements and architecture mature enough to plan implementation | Not started |
| 13 | Implementation handoff | Spec, tickets, test plan, coding prompt | All prior gates closed or explicitly waived | Not started |

The order follows dependencies: who the customer is (1–3) decides which workflows, laws, and standards matter (4–5), which decide the document model (6), which decides the technical design (7–10) and scope (11–13). Any change to the order is recorded in [DISCUSSION-TRACKER.md](DISCUSSION-TRACKER.md) with its reason.

## Ground rules

- No application code until Phase 13, unless the owner explicitly waives the gates.
- No assumptions about the ERP's code, schema, data, hosting, or APIs. The repository is not available.
- MySQL stays the default unless documented requirements justify a change.
- Prefer a modular monolith and the fewest moving parts that meet confirmed requirements.
- Never claim that the DMS makes anyone ISO-certified or legally compliant.
- Legal conclusions are research, not legal advice. Items needing a Philippine lawyer are flagged.
