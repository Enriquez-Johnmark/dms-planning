# DMS Discovery Workspace

Research, decisions, and planning for a possible Document Management System (DMS) for Philippine SMEs.

**This workspace holds no application code.** No implementation starts until the research, requirements, major architecture decisions, and MVP scope are approved by the project owner and the senior partner. See [MASTER-PLAN.md](MASTER-PLAN.md) for the approval gates.

## Start here

| If you want to… | Read |
|---|---|
| Understand what we're doing and in what order | [MASTER-PLAN.md](MASTER-PLAN.md) |
| See what we know, assume, and don't know | [PROJECT-CONTEXT.md](PROJECT-CONTEXT.md) |
| See which section is being discussed now | [DISCUSSION-TRACKER.md](DISCUSSION-TRACKER.md) |
| See what has been decided | [DECISION-LOG.md](DECISION-LOG.md) |
| Check what a word means (Document, Version, Official version…) | [GLOSSARY.md](GLOSSARY.md) |
| See what is still unanswered | [OPEN-QUESTIONS.md](OPEN-QUESTIONS.md) |
| Find research notes | [RESEARCH-INDEX.md](RESEARCH-INDEX.md) |
| Check a citation | [sources/SOURCE-REGISTER.md](sources/SOURCE-REGISTER.md) |
| Prepare for a partner meeting | Start with the [senior partner briefing pack](meeting-guides/00-senior-partner-briefing-pack.md); detail in [meeting-guides/](meeting-guides/) |

## Folder layout

```text
document-management-system/   (repository root)
├── README.md               ← this file
├── MASTER-PLAN.md          ← phases, gates, working method
├── PROJECT-CONTEXT.md      ← known / assumed / unknown / must-not-assume
├── GLOSSARY.md             ← agreed vocabulary (domain-modeling skill)
├── DISCUSSION-TRACKER.md   ← status of every section
├── DECISION-LOG.md         ← confirmed decisions only
├── OPEN-QUESTIONS.md       ← unanswered questions, with owner
├── RESEARCH-INDEX.md       ← list of research notes
├── research/               ← raw research notes (R01, R02, …), cited
├── sources/                ← source register (one row per source)
├── sections/               ← one research & decision document per section
├── meeting-guides/         ← one short partner meeting guide per section
├── questionnaires/         ← for the Officetul maintainer and the project owner
└── appendices/             ← matrices, ADRs, traceability (created when needed)
```

Change from the suggested structure: a `research/` folder was added for raw research notes, so section documents can summarise findings and link to them instead of repeating them. Appendices are created only when they first have real content.

## Statement labels

Every important statement carries one label:

- **Verified fact**: supported by a cited, reliable source.
- **Working assumption**: accepted for now, not yet verified.
- **Estimate**: a calculation or projection from stated assumptions.
- **Recommendation**: a proposed course of action, not yet approved.
- **Partner decision required**: needs an authorised stakeholder's choice.
- **Open question**: not enough information to conclude.
- **Out of scope**: explicitly excluded from the current phase.
