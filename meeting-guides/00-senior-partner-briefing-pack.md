# Senior Partner Briefing Pack — Officetul Documents

**For:** the senior partner. **From:** the project owner. **Meeting:** about 60–75 minutes, or two shorter sessions (Part 1 first). **Prepared:** 2026-10-21.

This pack replaces reading six separate meeting guides. It puts every decision that is waiting for you on one page, most urgent first. The detail stays in the guides and sections linked from each item.

---

## The project in one minute

- **What:** a **Documents app inside Officetul** that turns clients' paper into searchable, organised records linked to their Officetul transactions: a receipt with its expense, a contract with its customer. It is sold as part of the one-stop ERP, not as a separate product.
- **Where we are:** discovery is complete. The project owner has agreed 110 decisions covering features, security, privacy, hosting, backups, costs, scope, architecture, risks and roadmap. **No code has been written.** The build starts only after you approve the scope and team (item 1).
- **Headline numbers** (all **Estimates**):

  | Item | Estimate |
  |---|---|
  | Team | Two full-time developers |
  | Our own pilot | About 8 months after your approval (range 7–10) |
  | First external client | About 3 months after our own pilot starts |
  | Running cost per client company | About ₱900–1,270 a month, including our time |
  | Fixed running cost | About ₱13,000–21,000 a month, whatever the number of clients |
- **What makes it sellable:**
  - search inside every scan with no per-page fees;
  - an "official version" marker;
  - Philippine-ready document types (BIR 2307, invoices, permits);
  - documents attached to any Officetul record;
  - a tamper-evident activity log.

---

## Decision sheet

Fill in the answer column during the meeting. Items 1–4 are **urgent**: each month they wait, every date moves by a month (risk RK-74).

| # | Decision | Recommended | Your answer |
|---|---|---|---|
| **1** | **Approve the MVP scope, two full-time developers and the ~8-month timeline** | Approve | |
| **2** | **Is our company registered with the NPC, and do we have a Data Protection Officer?** Who owns the privacy tasks before the first client? | Check now; registration started by month 3 | |
| **3** | **Name a lawyer and a tax adviser, with a budget**, for one or two short sessions | Name both by month 2 | |
| **4** | **Repository access and the technical questionnaire:** the developer gets Officetul repository access on the day you approve; the Officetul maintainer answers the questionnaire first | Agree | |
| 5 | **Pricing:** included in Officetul with a storage allowance, an add-on, or a mix | Included base + extra storage and back-file scanning packs | |
| 6 | **Allowances and our hourly rate:** e.g. 25 GB and 3,000 OCR pages a month per company; ops time at ₱600 an hour (placeholder) | Confirm or change | |
| 7 | **Second-client gate, in writing:** no second client before staff support access is built, the second restore drill is done, and client 1 has run 6 weeks cleanly | Agree | |
| 8 | **Words we never use** in sales: "court-admissible", "tamper-proof", "BIR-compliant", "certified true copy", "go paperless" (D-029), and "Officetul is ISO certified" (D-028) | Agree | |
| 9 | **Pilot:** our company first for 2–4 weeks, then one client that is not a law firm, has plenty of paper, uses Officetul Expenses, and isn't onboarded in the holiday or tax season | Agree; name candidate clients | |
| 10 | **Document types and first industry pack** after the MVP (accounting, law or construction) | Confirm the standard list; pick the pack from your client mix | |
| 11 | **ISO (future goal, not blocking):** which certificate and when | ISO 27001 for the company covering Officetul, started when a client or tender asks, or after the pilot | |

---

## Part 1 — Start the build (items 1–4, about 30 minutes)

### 1. Scope, team and timeline

The first version covers:
- capture by upload and phone;
- the review Inbox;
- documents with versions and an official marker;
- folders and permissions;
- search inside scans;
- links to Officetul records;
- the "expenses without receipt" report;
- a protected activity log;
- virus scanning;
- locked backups.

**Not in the first version:** approval chains, reminders, retention rules, share links, the client portal, AI data extraction, e-signatures, and scan-to-email (after the pilots).

**Why 8 months, not 4–6:** security, privacy, integrity and backup work added since the first estimate. With one developer it takes about twice as long. The 4–6 month figure in earlier documents is out of date.

Detail: [Section 17](../sections/17-mvp-scope-and-prioritisation.md), [Section 21](../sections/21-implementation-roadmap.md).

### 2. NPC registration and the privacy tasks

Before any external client stores documents with us, the company needs:
- NPC registration as a processor;
- a named DPO;
- a standard data processing agreement with 24-hour breach notice;
- a published list of our cloud providers;
- a written incident procedure.

NPC processing time is unknown, so start early. Detail: [guide 10–11](10-11-legal-authenticity-and-privacy.md), [Section 11](../sections/11-philippine-legal-and-regulatory-research.md).

### 3. Lawyer and tax adviser

About 40 short questions wait for professionals, for example:
- whether scans can replace paper receipts for the BIR;
- what we may promise clients;
- exit and backup terms;
- cloud hosting for law firms.

The questions are already written: [legal and tax question list](../questionnaires/legal-and-tax-questions.md). Gate B (first client) needs these answers by about month 9.

### 4. Repository access and the questionnaire

Everything was designed without seeing Officetul's code, and that is our **top risk** (RK-07). The code review in the first two weeks of the build confirms or corrects the design. The maintainer should answer the [technical questionnaire](../questionnaires/erp-technical-questionnaire.md) **before** approval. Urgent items: Q1–Q3, Q18–Q22, Q27–Q28 and the MySQL version.

---

## Part 2 — Commercial and client rules (items 5–9, about 25 minutes)

### 5–6. Pricing and allowances

| Flat add-on per company | Companies needed to break even (**Estimate**) |
|---|---|
| ₱1,000 | about 24 |
| ₱1,500 | about 16 |
| ₱2,000 | about 11 |

- Users cost almost nothing to serve. Storage keeps growing, because documents are never deleted automatically. That's why the allowances are in GB and OCR pages, not users.
- Competitor prices:

  | Competitor | Price |
  |---|---|
  | Odoo | about ₱565–707 per user per month, Documents included |
  | Dext | about ₱1,590 a month |
  | Juan | ₱2,000 a month for its whole accounting product |

Detail: [guide 18](18-costs-and-pricing.md), [Section 18](../sections/18-cost-estimates-and-operating-model.md). Pricing can be decided in a later meeting. Until then, the first client can run on written free-pilot terms.

### 7. Second-client gate

For the first client, our staff can't open their documents at all; help is by screen-share. Approval-based staff access with emergency rules is built before client 2. Promising client 2 earlier would leave them without emergency help (RK-82, RK-85).

### 8. What we never claim

The app helps clients organise and protect records. It does not make anyone compliant, certified or court-proof. Detail: [Section 10](../sections/10-authenticity-and-certified-true-copies.md) and [guide 10–11](10-11-legal-authenticity-and-privacy.md).

### 9. Pilot

Our own company goes first, so that bugs are found on our own paper and we get a live sales demo. The first client should not be a law firm until the lawyer confirms cloud storage of client confidences.

---

## Part 3 — Later choices (items 10–11, about 10 minutes)

- **10. Document types:** see [guide 03](03-document-types.md) and the [document type catalogue](../appendices/document-type-catalogue.md).
- **11. ISO:** a future goal, and your decision. It doesn't block anything. See [guide 04](04-iso-certification.md).

---

## Top risks you own

From the [risk register](../sections/19-risk-register.md):

| Risk | What it means | What helps |
|---|---|---|
| RK-74 | Approval arrives late | Decide items 1–4 first |
| RK-23 | No NPC registration or DPO when the first client is ready | Start now |
| RK-80 | No lawyer or tax adviser named | Name them by month 2 |
| RK-72/76 | Two developers not available full-time | Confirm names and budget at approval |
| RK-82 | Sales promise more than is ready | Items 7 and 8 |

---

## After the meeting

The project owner records each answer in the [decision log](../DECISION-LOG.md). Then:
- open questions OQ-020 (pricing) and OQ-024 (NPC) are closed or updated;
- the build starts at Gate 0;
- the handoff package (specification, tickets, test plan) is prepared for the developers.

Items left blank stay open. Silence isn't taken as approval.
