# 01 — Business Problem and Product Opportunity

## 1. Purpose

Decide **whether there is a problem worth solving, for whom, and why us**, before any product or technology decision.

## 2. Status

**Awaiting senior partner sign-off.** Grilling Rounds 1–4 with the project owner are complete (2026-10-18). Decisions D-001 to D-017 are in the [Decision Log](../DECISION-LOG.md).

Research basis: [R01](../research/R01-ph-sme-market-and-demand-drivers.md) (market and regulation), [R02](../research/R02-dms-alternatives-landscape.md) (alternatives), [R03](../research/R03-iso-standards-primer.md) (ISO), and R04 (scanner capture, in progress). No customer interviews were done, by decision (D-004).

## 3. Executive Summary

**What we will build** *(decided)*: a **Documents app inside Officetul**, the existing multi-tenant Laravel/Inertia/React ERP. It sits beside Accounting and Inventory, in the style of Odoo or ERPNext, and the senior partner sells it as part of the one-stop ERP (D-002, D-007, D-010).

**For whom** *(decided)*: Officetul's existing SME clients across industries (law, accounting, construction and others), most of whom work on paper (D-003). The first pilot is our own company, then one client (D-017).

**The problem it solves** *(Working assumption, to be validated by the pilot)*: clients can't quickly find paper documents, can't link them to the transactions they support, and can't tell who has seen or changed them.

**Why us** *(Working assumption)*: Google Drive and Microsoft 365 are the real competitors at about ₱300–1,000 per user per month, but **they can't attach a receipt to an accounting entry in Officetul**. Enterprise DMS products cost about ₱6,800 per user per month locally (R02). An app already inside the client's ERP has a built-in sales channel and a feature generic tools can't match.

**What we must not claim** *(Verified fact, R03)*: software cannot be "ISO-certified", and the module does not make clients legally compliant. The module is built so its controls can support company-level ISO/IEC 27001 evidence (D-011). ISO certification is a future goal owned by the senior partner (D-028).

**The main remaining risks**: demand is unproven (D-006), and the module is being designed before anyone sees Officetul's code (RK-07).

## 4. Business Context

| Factor | What we know | Label |
|---|---|---|
| Product home | Officetul: multi-tenant Laravel + Inertia + React ERP, organised as apps | Verified fact (owner) |
| Code access | Repository access is granted after discovery, planning and design are approved by the senior partner | Verified fact (owner) |
| Sales model | The senior partner sells one ERP product as a one-stop shop | Verified fact (owner) |
| Clients | SMEs across industries; most are paper-heavy | Owner's observation, not measured |
| Team and timeline | 1–2 developers; pilot about 4–6 months after scope approval | Decision D-005 |
| Prior customer evidence | None recorded | Verified fact (D-006) |

**Problem statement** *(agreed by the owner; Working assumption until the pilot)*:

> Officetul's SME clients keep most business documents on paper, or scattered across email, chat apps and shared drives. They can't find a document quickly, can't connect it to the Officetul record it supports (an expense, an invoice, a customer, a project), and have no record of who saw or changed it. Paper is also exposed to loss, flood and staff turnover. A Documents app inside Officetul lets them capture paper by upload, email or phone; search by the text inside it; attach it to Officetul records; control who sees it; and keep a full history.

## 5. Verified Research Findings

### 5.1 Market size (R01 §1)

| Finding | Label |
|---|---|
| 1,241,733 MSMEs in 2023, 90.43% of them micro | Verified fact, from a search excerpt [S-001] |
| Formal-sector counts: 970 legal establishments (2022), 2,512 construction establishments (2024). Small sole proprietors are excluded. | Verified fact, from excerpt [S-002, S-003] |
| No credible data on SME cloud use, IT budgets, or document loss at private firms | Open question |

**Implication:** the relevant market for this module is **Officetul's own client base**, not the national SME count. Its size is unknown to this project. *(Open question, captured in the [ERP questionnaire](../questionnaires/erp-technical-questionnaire.md) Q16.)*

### 5.2 Regulatory context (R01 §3)

These matter for **features and claims**, not as the main sales trigger, because the module is sold horizontally.

| Area | What it means for the module | Label |
|---|---|---|
| Data Privacy Act + NPC Circular 2023-06 | Baseline: access control, MFA for sensitive data, backups, 72-hour breach notification, disposal duties | Verified fact [S-010, S-011] |
| E-Commerce Act s.13 | An accurate, accessible electronic copy can satisfy a legal duty to keep records. This supports the module's value, but doesn't make it sufficient. | Verified fact [S-006] |
| BIR (RA 11976, RR 7-2024) | Records are kept 5 years. **Manual and loose-leaf books must be kept in hard copy**, so digitising them doesn't replace the paper (D-008). | Verified fact (law); details secondary [S-004] |
| Rules on Electronic Evidence | A document's evidentiary weight depends on how reliably it was stored. Activity logs and file integrity help. | Verified fact [S-007] |
| Law-firm changes (e-filing, notarial PDFs, eNotarization) | Possible future industry pack | Secondary sources [S-008, S-009] |

### 5.3 Alternatives (R02)

| Alternative | Price signal | Why a client might still choose us |
|---|---|---|
| Google Drive / Microsoft 365 | ~₱300–1,000 per user/month (Estimate) | No link to Officetul records; document control needs expert setup |
| Enterprise DMS via local resellers | ~₱6,800 per user/month (DocuWare via 1doc.ph) | Price and implementation effort |
| Vertical SaaS (Clio, TaxDome, Procore…) | $39–65 per user/month or quote | Not localised; separate from the client's ERP |
| **ERP-embedded document apps** (e.g. Odoo Documents) | Not yet researched | **Closest model to ours. Section 02 studies these first.** |

### 5.4 ISO (R03)

- Software can't be certified. Organisations are certified against management-system standards by accredited certification bodies. *(Verified fact)*
- ISO 9001:2026 was published 2026-09-16. ISO/IEC 27001:2022 (with its 2024 amendment) is current. The Philippine Accreditation Bureau lists no accredited 27001 certification bodies, so 27001 certification in the Philippines comes from certification bodies accredited abroad. *(Verified fact [S-012, S-013])*
- The owner reports that the senior partner "aims to be ISO certified", whether as the company or as the software, which is still undecided. The design stance is settled (D-011). ISO certification is a **future goal**; the senior partner decides the standard and timing, and it **does not block this project** (D-028; [Section 04](04-iso-certification-strategy.md)).

## 6. Technical Analysis

Only what Section 01's decisions imply. The detail belongs to later sections.

- **Native Officetul app** (D-010). Same login, roles, tenancy and stack. Open-source tools are used as components only (an OCR engine, a virus scanner).
- **MySQL** is the strong default, because it is Officetul's database. *(Working assumption until Phase 9)*
- **Capture with nothing installed at the client** (D-015): browser upload, a per-tenant scan-to-email inbox, and phone capture. Everything lands in a review Inbox. [R04](../research/R04-scanner-capture-options.md) confirms the approach:
  - Scan-to-email is the only near-universal channel that needs nothing installed at the client. Every business multifunction printer checked has it; USB-only scanners don't. *(Verified fact)*
  - **The catch is the scanner's outgoing mail.** Microsoft 365 turns off simple password sign-in for devices sending mail by default at the end of December 2026, and Google already blocks it. Older scanners that send through the client's own email account may stop working. R04 suggests the DMS run its own authenticated mail-submission service with per-client credentials. That is a new service to operate and secure, so it is a **Phase 7 decision**, not settled here.
  - Receiving email is cheap: Amazon SES (Singapore region) or Postmark. NPC Circular 2023-06 §23 requires secure (TLS) email transmission. *(Verified fact)*
- **Tenant isolation must match Officetul's own method.** Until the questionnaire is answered, that method is unknown (RK-07).

## 7. Alternatives and Trade-offs

Resolved for this section:

| Question | Options considered | Outcome |
|---|---|---|
| Commercial model | Standalone product / ERP module / service / resell / don't pursue | **ERP module** (D-002): the senior partner sells one product |
| How the module is made | Native build / embed an open-source DMS engine / scanning service first | **Native build in Officetul** (D-010): one stack and one permission model for a small team |
| Scanner integration | Upload / scan-to-email / phone / PC agent ("script") / browser scanning kit | **Upload + email + phone** (D-015): nothing to install or maintain at clients |
| Validation | Interviews / samples and calls / pilot | **Pilot**, prepared by the owner's [client document profile](../questionnaires/client-document-profile.md) (D-012) |

## 8. Recommendation

1. **Senior partner signs off on this section**, using the [meeting guide](../meeting-guides/01-business-problem-and-opportunity.md). *(Partner decision required)*
2. **The owner fills in the client document profile** and **sends the ERP questionnaire** to Officetul's maintainer. *(Recommendation)*
3. **Start Section 02**, re-scoped: how ERP-embedded document apps work and are priced (Odoo Documents, ERPNext/Frappe, Zoho), and how to price the Documents app inside Officetul. *(Recommendation)*

## 9. Cost and Operational Implications

- **No hardware cost or client-side support** (D-015, D-016).
- **New running costs** the module adds to Officetul hosting: file storage, OCR processing, virus scanning, an inbound-email service, and backups of files as well as the database. These are estimated in Phase 10.
- **Price ceiling:** clients will compare the module's price with Google Drive pricing, so hosting and support must stay cheap. *(Estimate)*
- **ISO (goals A and C):** recurring certification-body audit costs. No credible Philippine SME figure was found. *(Open question)*

## 10. Risks and Limitations

Seed entries for the risk register (Section 19).

| ID | Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|---|
| RK-01 | **Weak demand.** Clients don't adopt or pay for the module. | Medium | High | Own-company pilot, then one client; small MVP; measure use. |
| RK-02 | **Incumbent good enough.** Clients keep using Drive or chat apps. | Medium | Medium | Lead with links to Officetul records, which generic tools can't offer. |
| RK-03 | **ISO over-claiming.** Marketing says "ISO-certified software". | Medium | Medium | Claims language from R03 §7; Section 04. |
| RK-05 | **Source quality.** Several figures came from search excerpts or secondary sources. | High (known) | Medium | Re-check in a browser before external use. |
| RK-07 | **Designing blind.** The module design clashes with Officetul's tenancy, roles, storage or queues. | High | High | ERP questionnaire; narrow integration points; code review before build (D-013). |
| RK-08 | **No pre-pilot validation.** Wrong assumptions about documents surface late. | Medium | Medium | Client document profile; small MVP; revision window after the pilot. |
| RK-09 | **Clients destroy paper originals** after scanning. | Medium | High | Never advise it (D-008); legal check per document type in Phase 5. |
| RK-11 | **Scanners can't send email** after Microsoft 365 and Google drop password-only device sign-in (Microsoft: end of Dec 2026). | Medium–High | Medium | Upload and phone capture always work. Decide on a DMS-run mail-submission service in Phase 7. Record scanner models during the pilot. |
| RK-10 | **Scan-to-email abuse.** Junk or malware is sent to a tenant inbox. | Medium | Low–Medium | Approved senders only, virus scanning, review Inbox (D-015). |

RK-04 (market too thin) and RK-06 (no ERP channel information) were closed by D-002 and D-003.

## 11. Questions for the Senior Partner

See the [meeting guide](../meeting-guides/01-business-problem-and-opportunity.md): sign-off on D-001 to D-017, plus the ISO goal.

## 12. Decisions Required

| Decision | Owner | Status |
|---|---|---|
| Sign-off on this section's decisions | Senior partner | Pending |
| ISO goal (standard and timing) | Senior partner | Future goal; senior partner decides; not blocking (D-028) |

## 13. Confirmed Decisions

D-001 to D-017 in the [Decision Log](../DECISION-LOG.md), confirmed by the project owner on 2026-10-18. The main ones:

| ID | Decision |
|---|---|
| D-002, D-007 | A Documents app inside Officetul, sold as part of the one-stop ERP |
| D-003 | Horizontal: all Officetul SME clients |
| D-005 | 1–2 developers; pilot about 4–6 months after scope approval |
| D-008 | Never advise destroying paper originals |
| D-010 | Native build in the Officetul codebase |
| D-011 | Controls designed to support company-level ISO 27001 evidence |
| D-014 | MVP: capture, folders/metadata, OCR search, permissions, activity log, official-version history, **links to Officetul records** |
| D-015, D-016 | Upload + scan-to-email + phone capture; nothing installed at clients; no hardware |
| D-017 | Pilot on our own company, then one client |

## 14. Open Questions

[OPEN-QUESTIONS.md](../OPEN-QUESTIONS.md): OQ-003 (ISO goal: future, senior partner decides), OQ-008 (Officetul internals), OQ-009 and OQ-010 (questions for a lawyer), OQ-011 and OQ-012 (ISO data and costs), OQ-017 (client scanners).

## 15. Dependencies

- **Feeds:** Section 02 (ERP-embedded competitors, pricing), Section 04 (ISO), Section 06/07 (capture, OCR), Section 16 (Officetul integration), Section 17 (MVP scope).
- **Depends on:** senior partner sign-off; the ERP questionnaire answers before Phase 9.

## 16. Sources

Full citations with access dates are in the research notes. Primary sources used here are in the [Source Register](../sources/SOURCE-REGISTER.md) as S-001 to S-014.

## 17. Next Steps

**Acceptance criteria for closing Section 01:**
- [x] Problem statement and target-customer hypothesis agreed by the owner (§4, D-003)
- [x] ISO stance recorded: ISO-ready design (D-011); certification is a future goal owned by the senior partner, not blocking (D-028)
- [x] Build approach recorded (D-002, D-010); "don't build" was considered and set aside
- [x] Validation plan agreed: a pilot instead of interviews (D-012, D-017)
- [ ] **Senior partner sign-off**
