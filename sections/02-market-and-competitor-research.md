# 02 — ERP-Embedded Document Apps and Module Pricing

## 1. Purpose

Learn from ERPs that already have a documents feature: what to **copy**, what to **avoid**, where Officetul's Documents app can **differ**, and how to **price** it.

Re-scoped on 2026-10-18 from "should we build at all?" because D-002 and D-010 settled that we build a native app inside Officetul. The general DMS landscape is in [R02](../research/R02-dms-alternatives-landscape.md) and summarised in [Section 01](01-business-problem-and-opportunity.md) §5.3.

## 2. Status

**Owner agreed (Round 5, D-018 to D-022); pricing deferred to Section 18.** Research: [R05](../research/R05-erp-embedded-document-apps.md) (ERP document apps) and [R06](../research/R06-capture-addons-and-pricing.md) (capture add-ons, pricing, unit costs), both 2026-10-18. The R05 agent's web-search quota ran out partway through, so NetSuite and SAP Business One are thin.

## 3. Executive Summary

- **Everyone includes document capture in the base price.** Xero (Hubdoc is included in its Philippine plans), QuickBooks, Zoho Books and Odoo do not charge separately for storing and attaching documents. **What they meter is AI data extraction**, i.e. reading the supplier, date and amount off a receipt. *(Verified fact, R06 §1, §3)*
- **Odoo, the closest model to Officetul, locks Documents to its paid Enterprise edition** (US$11.20 per user per month), and folder automations need the more expensive Custom plan. OCR is charged per document. *(Verified fact, R05 §1)*
- **No ERP reviewed has an "official version" marker, retention rules or legal hold,** and most don't search the text inside files in the core product. These are open ground for Officetul. *(Verified fact for the products and pages checked, R05 §6)*
- **The underlying costs are tiny.** Self-hosted OCR, storage and inbound email for a typical tenant come to **about ₱48 per month** including VAT, or about ₱257 with cloud OCR. *(Estimate; R06 §4, assuming 20 GB stored, 2,000 pages and 300 emails per month)*
  - **Update 2026-10-21:** this counted only variable costs. Including the fixed processing server and ops time, one company costs about **₱900–1,270 a month** at year-1 scale. See [Section 18](18-cost-estimates-and-operating-model.md).
- **Peso price anchors** *(Verified fact or Estimate, R06 §7)*:
  - Juan accounting: ₱2,000/month per organisation.
  - Receipt-capture add-ons: roughly ₱500–1,600/month.
  - Odoo: roughly ₱560–700 per user per month.

**Recommendation:** include the Documents app in Officetul's subscription rather than selling it as a separate add-on. Compete on four things Odoo and others lack:
1. documents included, with no per-document OCR fee;
2. search inside every scan;
3. an "official version" marker;
4. Philippine-ready document types and reports.

## 4. Business Context

The senior partner sells Officetul as a one-stop shop (D-002). The pricing question is whether Documents is **a reason to buy Officetul** (included) or **a separate line on the invoice** (add-on). Officetul's own pricing model (per user or per company, tiers) is not yet known to this project. *(Open question)*

## 5. Verified Research Findings

### 5.1 How ERP document apps work (R05)

| Product | Price / edition | Links to records | Email-in | OCR / extraction | Versions | Retention |
|---|---|---|---|---|---|---|
| **Odoo Documents 19** | Enterprise only; US$11.20/user/mo (Custom $20.40 for automations) | One main linked record, plus links from other records | One address per folder | €0.05–0.15 per document | Upload history plus lock; **no official marker** | 30-day trash only |
| **ERPNext** | Free / open source | Exactly one record per file; permissions copied from that record | None in core | Experimental add-on | **None** | None |
| **Zoho Books** | US$0–200 per organisation/mo | Attach to transactions | One address per organisation | 50–1,000 scans/mo by plan; $8–10 per extra 50 | Not documented | **Clearing the inbox deletes permanently** |
| **Business Central** | US$80–110/user/mo | One-to-one, in **two separate stores** | Via Outlook or an AI agent (billed in credits) | Third-party or paid AI | None documented | None |
| **NetSuite** | Quote only | **Many-to-many** | US only | US only | Not confirmed | Not confirmed |

### 5.2 Patterns worth copying (R05 §8)

| Pattern | From | What it means for Officetul |
|---|---|---|
| Email address per folder | Odoo | Each company gets an inbox address; optionally `bills@…` goes straight to the Finance Inbox |
| The Inbox is a to-do list | Zoho, Business Central | Unfiled documents stay visible until someone files or links them |
| "Records with no supporting document" report | Business Central | e.g. **"Expenses without a receipt"**, valuable for BIR audits and cheap to build |
| Document requests | Odoo | A placeholder ("send us your 2307"), an assignee and reminders. Fits law and accounting firms chasing clients. |
| Link instead of copy | Odoo, NetSuite | One file can be linked to several records without duplicates |
| A person confirms extracted data | Zoho, NetSuite, Business Central | Never post an expense from OCR automatically |
| Store identical files once | Frappe | Saves storage; also detects duplicate uploads |

### 5.3 Pitfalls to avoid (R05 §9)

- **Hiding Documents behind the top tier** (Odoo) or **surprise per-document OCR charges** (Odoo, Zoho, Business Central).
- **Two file stores for the same record** (Business Central). Users can't find files.
- **Permissions only by folder** (NetSuite) or **only by record** (ERPNext). Both leave gaps.
- **Permanent deletion from the Inbox** (Zoho). Always allow recovery.
- **Small file limits** (Zoho 5 MB). Scanned contracts and drawings are bigger.
- **OCR with no progress indicator.** Users think the upload failed.

### 5.4 Pricing models (R06 §6)

| Model | Example | Fit for Officetul |
|---|---|---|
| **Included in the ERP subscription** | Xero/Hubdoc, QuickBooks, Odoo (Enterprise) | **Best fit for the one-stop pitch**; no separate revenue |
| Flat add-on per company | Hubdoc standalone ≈ ₱750; Juan ₱2,000 (whole product) | Simple to quote; a second line on the invoice |
| Per user | Odoo ≈ ₱560–700 | Discourages giving access to occasional users |
| Storage blocks | Odoo 100 GB; DigitalOcean 250 GB for $5 | Tracks a real cost; storage is so cheap that small quotas look petty |
| Monthly quota plus top-up packs | Zoho scans, $10 per 50 | **Good for AI extraction** (a later feature) |
| Prepaid credits | AutoEntry, Odoo IAP | SMEs dislike running out |

### 5.5 Unit costs (R06 §8)

| Cost | Unit price | ≈ PHP |
|---|---|---|
| Storage (AWS S3, Singapore) | $0.025 per GB per month | ₱1.57 |
| Plain OCR, cloud (Google, AWS, Azure) | $1.50 per 1,000 pages | ₱94 |
| Plain OCR, self-hosted Tesseract | Server time only | ≈ ₱1–2 per 1,000 pages (Estimate) |
| AI receipt/invoice extraction | $10 per 1,000 pages | ₱628 |
| Inbound email (Amazon SES, Singapore) | $0.10 per 1,000 emails plus a size charge | ≈ ₱12 per 1,000 |

**VAT** *(Verified fact, R06 §5)*: RA 12023 and RR 3-2025 put 12% VAT on digital services. Officetul's prices carry 12% VAT if the seller is VAT-registered, and Officetul's own AWS, Google and Azure bills carry 12% VAT under the reverse charge.

## 6. Technical Analysis

These are product principles only. The data model is designed in Phases 6 and 9.

- **Link model.** One document can be linked to many Officetul records, each link labelled (e.g. "receipt for", "contract of"). This avoids ERPNext's one-record limit and Business Central's two stores.
- **Permission principle.** The folder decides who can see a document. A link to a record grants visibility only where the link type is set to allow it, and the screen says why a user can see it. Detailed design comes in Phase 8.
- **OCR cost.** Self-hosted Tesseract by default keeps OCR nearly free, which is what makes "included, no per-document fee" affordable. The engine is chosen in Phase 7. *(Working assumption)*
- **Generous file size limit** (well above Zoho's 5 MB), set in Phase 7.

## 7. Alternatives and Trade-offs

| Question | Options | Trade-off |
|---|---|---|
| Pricing | Included / add-on / per user / storage tiers | "Included" sells Officetul; "add-on" earns separately but weakens the one-stop pitch |
| Links | One record per document / many records per document | Many is more flexible; a little more design work |
| Permissions | Folder only / record only / folder plus opt-in record links | The combination is safest and most flexible, but harder to explain. Showing "why you can see this" fixes that. |
| Extra MVP items | "Missing receipt" report; PH document types; document requests | The first two are cheap and high-value; requests belong with the client portal |

## 8. Recommendation

1. **Include Documents in Officetul's subscription.** Use a generous storage allowance per company as the fair-use limit, with paid top-up blocks. Never charge per page for OCR. Charge only for **AI extraction**, when it exists (a later release), as a monthly quota plus top-up packs. *(Recommendation)*
2. **Many-to-many labelled links** between documents and Officetul records. *(Recommendation)*
3. **Folder-based permissions plus opt-in visibility through record links**, with a "why can I see this?" explanation. *(Recommendation; detail agreed in Phase 8, D-053)*
4. **Add two cheap, high-value items to the MVP:**
   - an **"Expenses without a receipt" report**;
   - **Philippine preset document types**: Official Receipt, Sales Invoice, BIR 2307, SEC GIS, Mayor's/Business Permit, Contract, and so on.

   This amends D-014 if approved. *(Recommendation)*
5. **Document requests go in the next release,** together with the client portal. *(Recommendation)*
6. **Positioning line for the senior partner:** "Documents included in Officetul: search inside every scan, attach files to any transaction, and always know which version is official. No per-page fees." *(Recommendation)*

## 9. Cost and Operational Implications

- Including Documents adds roughly **₱50–260 per tenant per month** in running costs, depending on the OCR choice *(Estimate, R06)*, plus development and support.
- Heavy users are handled by the storage allowance and paid blocks.
- **Price quotes must state whether 12% VAT is included.**
- The full per-tenant cost model is built in Phase 10 (Section 18).

## 10. Risks and Limitations

| ID | Risk | Mitigation |
|---|---|---|
| RK-12 | **Including Documents loses revenue** if it takes a lot of support or storage. | Storage allowance plus paid blocks; Phase 10 cost model; revisit after the pilot. |
| RK-13 | **Opt-in record-link visibility exposes sensitive documents**, e.g. a contract linked to a customer seen by all sales staff. | Off by default per link type; "why can I see this"; Phase 8 threat model. |
| RK-14 | **Self-hosted OCR quality** is too low on poor scans. | Benchmark in Phase 7; keep a cloud OCR option. |

**Limitations:** Odoo's peso pricing, NetSuite, SAP Business One and local ERP partners are not covered. The ₱500–1,600 add-on range is an inference from foreign products, not a tested price.

## 11. Questions for the Senior Partner

See the [meeting guide](../meeting-guides/02-erp-document-apps-and-pricing.md).

## 12. Decisions Required

Grilling Round 5: pricing (included vs add-on), links, permission principle, MVP additions, document requests. Plus one fact: how Officetul is priced today.

## 13. Confirmed Decisions

| ID | Decision |
|---|---|
| D-018 | **Pricing deferred** until the cost estimate (Section 18). §8 item 1 remains a recommendation only. |
| D-019 | Many-to-many labelled links between documents and Officetul records |
| D-020 | Folder-based permissions plus opt-in visibility through record links, with a "why can I see this" explanation |
| D-021 | MVP adds the "Expenses without a receipt" report and Philippine preset document types |
| D-022 | Document requests go in the next release, with the client portal |

## 14. Open Questions

- Officetul's current pricing model and the senior partner's margin (Round 5 Q1).
- Whether Odoo searches inside PDFs, and whether any ERP offers retention (R05 §11).
- Local Odoo or SAP Business One partner packages in PHP (R06 §10).

## 15. Dependencies

- **Depends on:** Section 01 decisions.
- **Feeds:** Section 05/08 (links, official version), Section 07 (OCR engine), Section 12 (permissions), Section 17 (MVP), Section 18 (cost and pricing).

## 16. Sources

Details are in R05 and R06. Key sources are in the [Source Register](../sources/SOURCE-REGISTER.md) as S-015 to S-022.

## 17. Next Steps

**Acceptance criteria for closing Section 02:**
- [x] Pricing approach: **deferred to Section 18** by the owner (D-018)
- [x] Features to copy and to avoid agreed (D-019, D-020, D-022)
- [x] MVP changes recorded as an amendment to D-014 (D-021)
- [ ] Senior partner informed of the positioning line
