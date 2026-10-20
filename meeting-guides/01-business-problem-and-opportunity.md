# Meeting Guide — Business Problem and Product Opportunity

**Duration:** 30–45 minutes. **Detail and sources:** [Section 01](../sections/01-business-problem-and-opportunity.md).

## Meeting Objective

The senior partner **confirms or changes** the direction the project owner agreed on 2026-10-18, and says how Officetul is priced today and who could do the legal and tax sessions. ISO has its own guide.

## Why This Matters to the Business

These decisions fix what we build for the next 4–6 months. Changing them now is cheap; changing them after development starts is not.

## Plain-Language Explanation

- **What we'd build:** a **Documents app inside Officetul**, next to Accounting and Inventory. You sell one ERP, as you do now. The Documents app is part of it.
- **What it does for clients:**
  - Get paper in three ways: drag a scanned PDF in, press "Scan to Email" on the office printer, or take a photo with a phone.
  - Find any document by typing words that appear inside it.
  - **Attach documents to Officetul records**, e.g. a receipt to an expense or a contract to a customer.
  - Control who can see each folder, and keep a history of who opened or changed what.
- **Why clients would pay:** Google Drive is cheap, but it can't attach a receipt to an accounting entry in Officetul. Proper document systems sold locally cost around ₱6,800 per user per month.
- **Your scanner concerns are answered:**
  - We supply no scanners. Clients use what they have, or a phone.
  - **No script is installed on client computers.** Everything runs on our server, maintained like the rest of Officetul.
  - Scans sent by email land in an Inbox for a person to check. Only approved senders are accepted, and everything is virus-scanned.
- **ISO:** we plan to pursue ISO certification in the future, and you decide which standard and when. Software itself cannot be certified; **a company** is, by an accredited auditor. The Documents app is built ISO-ready in the meantime. Details are in the [ISO meeting guide](04-iso-certification.md).

## Questions to Discuss

1. **Do you approve the direction?** A Documents app inside Officetul, for all our SME clients, built by 1–2 developers, with a pilot about 4–6 months after the scope is approved.
2. **Is the first version right?** It covers capture, search, folders, permissions, history, and attaching documents to Officetul records. Approval chains, reminders and retention rules come in the next version.
3. **Pilot:** does our own company use it for 2–4 weeks before a client does? Which client comes next?
4. **ISO (future goal):** use the separate [ISO meeting guide](04-iso-certification.md). Which standard and when is your decision (D-028), and it does not hold up the project.
5. **Repository access:** once you approve the plan and the design, the developer gets Officetul repository access before any building starts. Agreed?

## Options and Trade-offs

| Topic | Agreed option | Alternative set aside | Why |
|---|---|---|---|
| Product form | Documents app inside Officetul | Separate product | One product is easier to sell |
| Scanning | Upload, email, phone | Program installed on every client PC | No client-side maintenance or security burden |
| Hardware | None from us | Supplying scanners | Stay a software company |
| Validation | Our own pilot, then one client | Interviewing clients | Interviews aren't feasible |

## Decisions Required

| Decision | If postponed |
|---|---|
| Approve direction (Q1–Q3) | Section 02 and later work may be wasted |
| ISO goal (Q4) | Nothing blocks: see the [ISO meeting guide](04-iso-certification.md) |
| Repository access timing (Q5) | Design can't be checked against the real code |

## Notes and Partner Feedback

| Question | Senior partner's answer | Date |
|---|---|---|
| 1. Direction | | |
| 2. First-version scope | | |
| 3. Pilot | | |
| 4. ISO (see the ISO guide) | | |
| 5. Repository access | | |
| Other comments | | |

## Follow-up Actions

| Action | Owner | Due |
|---|---|---|
| Fill in the [client document profile](../questionnaires/client-document-profile.md) | Project owner | |
| Send the [Officetul technical questionnaire](../questionnaires/erp-technical-questionnaire.md) to the ERP maintainer | Project owner | |
| Relay the senior partner's answers to Claude | Project owner | |
| Section 02: ERP-embedded document apps and module pricing | Claude | After sign-off |
