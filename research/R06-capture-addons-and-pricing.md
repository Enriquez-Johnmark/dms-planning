# R06: Capture add-ons, PH software price anchors and unit costs

**Research date:** 2026-10-18. All sources accessed 2026-10-18 (see §11; `[Sx]` = source ID).
**Scope:** How accounting tools price receipt and document capture, PHP price anchors for SME software, the published pricing models for document modules, the unit costs we would pass through, and PH VAT. This feeds the pricing decision for Officetul's Documents app.
**Labels:** **Verified fact** (read on an official page or price file) · **Vendor claim** (a vendor's own marketing statement) · **Anecdotal** (third-party or community source, or a search snippet only) · **Estimate** (our calculation) · **Open question**.
**FX used for PHP conversions:** USD 1 ≈ PHP 62.76. This is the October 2026 figure on the BSP daily-rate page [S41]. The table renders badly, so the column reading is uncertain (**Estimate**). Converted figures are marked "≈".
**Constraint:** The session's web-search budget ran out partway through. Some local items (Odoo and SAP B1 partner packages, Kitta) could not be searched and are listed in §9.

---

## 1. Receipt and document capture add-ons for accounting software

| Product | Capture channels | Extraction | Pricing model and published price | Limits | PH presence |
|---|---|---|---|---|---|
| **Dext (Prepare)** | Mobile app, upload, email-in, WhatsApp, bank feeds [S2] (**Vendor claim**) | Data extraction from receipts and invoices (**Vendor claim**) | Business plan: **US$25.21/mo on annual billing (US$302.50/yr), excl. tax**. Price rises with a document/user slider [S1] (**Verified fact**). Practice plan from **US$239.19/mo, starting at 10 clients** [S2] (**Verified fact**). Bank-statement add-on from US$13/mo [S1]. | 250 docs/mo and 5 users on the entry tier; +50 docs per added user [S1][S2] | Listed on the Xero PH app store, but prices are in GBP/AUD/USD/EUR/ZAR only. No PHP pricing found (**Verified fact** [S2]). |
| **Hubdoc (Xero)** | Email forward, mobile photo, scanner, desktop upload [S5] (**Vendor claim**) | OCR plus sync to Xero (**Vendor claim**) | Standalone **US$12/mo** after a 30-day trial [S5] (**Verified fact**). **Included in every Xero PH plan** (Lite, Starter, Standard, Premium) [S3] (**Verified fact**). | No document cap published [S3][S5] | Included on xero.com/ph [S3] |
| **AutoEntry** | Web upload (single or batch), email auto-forward, phone app [S6] (**Vendor claim**) | Invoice, receipt and statement extraction | **Credit packs**, monthly, excl. VAT: Bronze 50 cr **US$13**, Silver 100 **$24**, Gold 200 **$46**, Platinum 500 **$103**, Diamond 1,500 **$298**, Sapphire 2,500 **$469** (US$0.26 down to $0.19 per credit) [S6] (**Verified fact**) | 1 cr per invoice/receipt, 2 per invoice with line items, 3 per bank-statement page. Unlimited users. Credits roll over 90 days. Overage allowed up to 200% [S6] | No PHP price. Currencies are USD/GBP/EUR/CAD/AUD/NZD/ZAR [S6] (**Verified fact**) |
| **Xero Files / attachments** | Upload, forward to the org's Xero email address, Xero mobile app photo [S4] (**Vendor claim**) | "Smart Document Capture" on all US plans, "free whatever your plan" [S4] (**Vendor claim**). In PH, Lite excludes bills [S3]. | Bundled in the Xero plan. **PH plans in USD:** Lite $7, Starter $29, Standard $50, Premium $75 per month (promo 80% off for 3 months) [S3] (**Verified fact**) | File library: **under 25 MB per file** (Xero Central UK, seen only as a search snippet because the page renders in JS) [S7] (**Anecdotal**). Third-party integrators report 10 MB and 10 files per transaction via the API (**Anecdotal**). No total storage cap found (**Open question**). | xero.com/ph exists. Prices in USD, tax treatment not stated [S3][S42] |
| **QuickBooks Online receipt capture** | Forward to a custom `@assist.intuit.com` address (older accounts: `@qbodocs.com`), mobile app, upload [S9] (**Verified fact**) | Receipt matching (**Vendor claim**) | Bundled. US list prices: Simple Start $38, Essentials $85, Plus $140, Advanced $340 per month (50% off for 3 months). Mobile app not on the Free plan [S8] (**Verified fact**) | Email-in attachments 46 KB to 20 MB, PDF/JPG/GIF/PNG, one receipt per file [S9] (**Verified fact**). Community answers say there is no total attachment cap (**Anecdotal**). | quickbooks.intuit.com/ph/pricing exists, but prices load live and could not be captured. PH sales phone +63 28231 3135 [S10] |
| **Zoho Books autoscan** | Documents Inbox with a unique per-org email address, upload, cloud drives (Dropbox, Google Drive and others) [S13] (**Verified fact**) | Autoscan in 15 languages (no Filipino) [S13] | **Per-org plans (USD, monthly / annual):** Free $0; Standard $12/$10; Professional $24/$20; Premium $36/$30; Elite $129/$100; Ultimate $249/$200 [S12] (**Verified fact**). **Autoscan add-on: US$10/mo (US$8 annual) per 50 scans** [S12][S13] | Autoscans per month: Free 50, Standard and Professional 200, Premium and above 1,000 [S12]. Storage limits not stated. | No PH edition. The add-on price table has no PHP row [S12][S13] (**Verified fact**) |

**Takeaways (Estimate):**
- Inside the main accounting products, capture (email-in, mobile, upload) and attachments are **bundled into the subscription**. Xero, QBO and Zoho all do this.
- Vendors meter **AI data extraction**, not storage or upload: Zoho autoscans, Dext document allowances, AutoEntry credits, Odoo IAP.
- Every product here uses one email-in address **per organisation** (Zoho, Xero) or **per user** (QBO, Hubdoc). That is the same pattern as Officetul's planned per-tenant inbox.

## 2. Philippine accounting, ERP and HR software: PHP anchors and document features

| Product | Published price (PHP unless stated) | Billing / tier | Document features found | Label |
|---|---|---|---|---|
| **Juan Accounting (merged JuanTax + Jaz)** | Free ₱0 (5 users). **Essentials ₱2,000/mo** (unlimited admins, 10 custom users). Growth "Let's Talk" [S14] | Monthly or annual. Annual price not published. | Mobile app "snap photos of receipts" [S15]. "Extractions" and "Categorization" tiers, undefined [S14]. JuanTax: "five-year backup of tax returns, attachments and confirmation receipts" [S16]. | **Verified fact** (price); **Vendor claim** (features) |
| **Taxumo** | 8% Plan ₱2,699 / ₱2,499 / ₱1,374. Micro ₱4,995 / ₱3,248 / ₱2,124. SMB ₱6,995 / ₱4,248 / ₱2,749. All labelled "/Quarter" under Quarterly, Annual and 2-Year tabs. Timeout ₱2,699/yr. Retainer ₱3,000/mo plus subscription [S17] | Per-quarter price drops with a longer commitment | No document storage or capture. Only "Digital Books of Accounts" (loose-leaf) [S17] | **Verified fact** (the tab mapping is our reading) |
| **Sprout (HR/payroll)** | Payroll Starter (outsourced) **from ₱15,000/mo for up to 20 employees**, then ₱300–₱400 per employee per month. Raised from ₱7,900. 12-month contract, no trial [S18] | Monthly, 1-year contract | Payroll registers, payslips, statutory reports. No document management mentioned [S18][S19]. Core plans are quote-only [S19]. | **Verified fact** |
| **Xero PH (for comparison)** | Lite ≈ ₱439, Starter ≈ ₱1,820, Standard ≈ ₱3,138, Premium ≈ ₱4,707 per month (USD list converted) | Monthly | Hubdoc and capture included | **Estimate** (conversion) |
| **Zoho Books (for comparison)** | Standard ≈ ₱628–753, Professional ≈ ₱1,255–1,506 per org per month. Autoscan add-on ≈ ₱502–628 per 50 scans | Monthly / annual | Inbox, autoscan | **Estimate** (conversion) |
| **Odoo (for comparison)** | Standard US$8.95 (promo, 12 months) / US$11.20 per user per month ≈ ₱562 / ₱703. Custom US$16.40 / US$20.40. The page shows USD even with `?country=PH` [S20] | Per user per month | Documents app sits in "All apps" on Standard and Custom. Odoo Online "unmetered storage up to 100GB". Vendor-bill OCR needs IAP credits [S20][S22] | **Verified fact** (USD); **Estimate** (PHP) |

**Takeaways (Estimate):**
- Local SME accounting software clusters around **₱2,000/month per organisation** (Juan Essentials).
- Foreign tools sell to PH buyers in **USD** and do not publish PHP prices.
- None of the local products found (Juan, Taxumo, Sprout) markets a document-management module. Receipt photos and attachments are the most they offer.
- No local "DMS add-on sold with an ERP" was found before the search budget ran out (**Open question**).

## 3. Published pricing models for document modules

| Model | Concrete published example | Numbers |
|---|---|---|
| **Per user (DMS inside an all-apps suite)** | Odoo Standard: Documents included [S20] | US$8.95–11.20 per user per month |
| **Users bundled into a document allowance** | Dext Business [S1] | US$25.21/mo: 250 docs and 5 users; +50 docs per added user |
| **Flat fee per organisation** | Hubdoc standalone [S5]; Zoho Books plans [S12]; Juan Essentials [S14] | US$12/mo; US$10–200/mo; ₱2,000/mo |
| **Bundled into the ERP or accounting tier** | Hubdoc in all Xero PH plans [S3]; QBO receipt capture [S8][S9]; Xero Smart Document Capture [S4] | ₱0 extra; capped only by plan features |
| **Quota per tier plus top-up packs** | Zoho autoscans 50 / 200 / 1,000 per month, then US$10 per 50 [S12] | ≈ US$0.20 per extra scan |
| **Prepaid credits per document** | AutoEntry [S6] | US$0.19–0.26 per credit. Statements cost 3 credits per page. |
| **Prepaid credits (IAP) for OCR digitisation** | Odoo IAP [S21] | €15 per 100 docs (€0.15), €120 per 1,000 (€0.12), €1,000 per 10,000 (€0.10), €5,000 per 100,000 (€0.05). 200 free credits. |
| **Storage tiers (infrastructure)** | DigitalOcean Spaces [S28]; Odoo Online [S20] | $5/mo for 250 GiB, then $0.02/GiB; "up to 100GB" unmetered |
| **Pay-per-page (cloud OCR)** | Google, AWS, Azure (§4) | US$1.50 per 1,000 pages for plain-text OCR |

## 4. Cost drivers to pass through (Singapore region where available)

### 4a. Object storage
| Provider | Price (USD) | Notes | Label |
|---|---|---|---|
| **AWS S3 Standard, ap-southeast-1** | **$0.025/GB-mo** for the first 50 TB, then $0.024, then $0.023. S3-IA $0.0138. Glacier Instant Retrieval $0.005. PUT $0.005 per 1,000. GET $0.004 per 10,000. | AWS price-list file published 2026-10-15 [S23] | **Verified fact** |
| AWS internet egress, ap-southeast-1 | $0.12/GB for the first 10 TB beyond the free tier | Page says 100 GB/month free, aggregated across services [S24] | **Verified fact** |
| **DigitalOcean Spaces (SGP1 available)** | $5/mo includes 250 GiB stored and 1,024 GiB egress. Then $0.02/GiB stored and $0.01/GiB egress. Cold storage $0.007/GiB. | Free transfer from SGP1 Spaces to SGP1 Droplets [S28] | **Verified fact** |
| Wasabi | From $7.99/TB-mo (≈ $0.008/GB). No egress or API fees. | Page does not list regions or minimums [S29] | **Verified fact**; APAC region: **Open question** |
| Backblaze B2 | From $6.95/TB-mo. Free egress up to 3× average stored, then $0.01/GB. | Published rate is for US West. No APAC region listed [S30]. | **Verified fact**; APAC: **Open question** |

### 4b. OCR (USD per 1,000 pages)
| Service | Plain text / Read | Structured extraction | Free tier | Label |
|---|---|---|---|---|
| **Google Document AI** Enterprise Document OCR | **$1.50** (1k–5M pages), $0.60 above 5M | Form Parser $30 (first 1M); Layout Parser $10 | First 1,000 pages/mo free | **Verified fact** [S31] |
| Google Cloud Vision DOCUMENT_TEXT_DETECTION | **$1.50** (1k–5M), $0.60 above. PDFs billed per page. | — | First 1,000 units/mo free | **Verified fact** [S32] |
| **AWS Textract, ap-southeast-1** | DetectDocumentText **$1.50** (0–1M), $0.60 above | AnalyzeExpense (receipts/invoices) **$10** (0–1M), $8 above; Forms $50; Tables $15 | — | **Verified fact** [S25] |
| **Azure Document Intelligence, southeastasia** | S0 Read **$1.50** (0–1M), $0.60 above | Prebuilt (invoice/receipt/layout) **$10**; Custom $30 | 500 pages/mo free | **Verified fact** via the Azure Retail Prices API [S33]. The pricing page did not render amounts [S34]. |
| **Tesseract (self-hosted)** | **$0 licence** (Apache 2.0) [S35]. Filipino model `fil.traineddata` exists [S36]. | None. Text only, no field extraction. | — | **Verified fact** |
| Tesseract compute | DO Basic droplet, 2 vCPU / 4 GiB: $0.03571/hr ($24/mo) [S37]. At an assumed ~2–5 CPU-seconds per page, 1,000 pages ≈ 0.6–1.4 CPU-hours ≈ **$0.01–0.03** of droplet time. | — | — | **Estimate** (per-page speed not measured) |

### 4c. Inbound email (Amazon SES receiving)
- SES receiving is available in **ap-southeast-1** (endpoint `inbound-smtp.ap-southeast-1.amazonaws.com`) [S27] (**Verified fact**).
- Price: **$0.10 per 1,000 emails received**, plus **$0.09 per 1,000 incoming 256 KB chunks** [S26] (**Verified fact**).
- Mail Manager processing, if used, costs $0.15 per 1,000 [S26].
- Example: a 1 MB scan email uses 4 chunks, so 1,000 such emails cost about $0.10 + $0.36 = **$0.46** (**Estimate**).

### 4d. Example cost per tenant per month (**Estimate**)
Assumptions: 20 GB stored, 2,000 pages OCR'd per month, 300 inbound emails of about 1 MB each.

| Line | Self-hosted Tesseract | Cloud OCR (Read) | + receipt extraction (300 receipts) |
|---|---|---|---|
| S3 SG storage, 20 GB × $0.025 | $0.50 | $0.50 | $0.50 |
| PUT requests (~2,500) | $0.01 | $0.01 | $0.01 |
| OCR | ~$0.03 | $3.00 | $3.00 |
| SES inbound | $0.14 | $0.14 | $0.14 |
| AnalyzeExpense ($10/1k) | — | — | $3.00 |
| **Total USD** | **≈ $0.68** | **≈ $3.65** | **≈ $6.65** |
| **≈ PHP @ 62.76** | **≈ ₱43** | **≈ ₱229** | **≈ ₱417** |
| + 12% VAT on foreign cloud services (§5) | ≈ ₱48 | ≈ ₱257 | ≈ ₱467 |

Storage and email are trivial costs. **OCR source and AI extraction drive cost**, and only extraction is large enough to need metering.

## 5. Philippine VAT

| Point | Finding | Label / source |
|---|---|---|
| RA 12023 | Approved **2 Oct 2024**. Adds NIRC Sec. 108-A and 108-B. Imposes **12% VAT on digital services** (includes "cloud services" and online platforms). Takes effect 15 days after publication. | **Verified fact** [S38] |
| Applies to resident sellers too | Sec. 108-A makes the DSP "whether resident or nonresident" liable for VAT on digital services consumed in the PH | **Verified fact** [S38] |
| RR 3-2025 | Signed **16 Jan 2025**, BIR records stamp 17 Jan 2025. Effective 15 days after publication in the Official Gazette or on the BIR website, whichever is earlier (Sec. 17). Nonresident DSPs register within 60 days and are **subject to VAT 120 days after effectivity** (Sec. 14). | **Verified fact** [S39] (scanned PDF read page by page) |
| Exact start date for nonresident DSP VAT | Commonly reported as early June 2025. Not confirmed from a primary source this session. It has been in force for over a year either way. | **Open question / Anecdotal** |
| Resident VAT-registered DSP (e.g. Officetul's seller) | Files and pays VAT under the normal Title III rules, "whether or not its buyer is engaged in business" (RR 3-2025 Sec. 6(A)). Rate is 12% of gross sales (Sec. 4). | **Verified fact** [S39] |
| B2B purchases from nonresident DSPs (AWS, Google, Azure) | The PH business buyer **withholds and remits the 12% VAT** within 10 days after month-end (reverse charge) | **Verified fact** [S38][S39] |
| VAT threshold | Sales or receipts not over **₱3,000,000** a year are VAT-exempt (Sec. 109(BB) as amended by TRAIN) | **Verified fact** [S40] |

**Implication (Estimate):**
- If the selling entity is VAT-registered, Officetul's Documents fee carries **+12% VAT**. Quotes should say "VAT-exclusive" or "VAT-inclusive" explicitly.
- Cloud costs paid to foreign providers carry a 12% VAT outflow.
- Whether the reverse-charged VAT can be claimed as input tax is for the company's accountant to confirm (**Open question**).

---

## 6. (a) Pricing models: pros and cons for an ERP-embedded module

| Model | Real example | Pros (ERP-embedded) | Cons |
|---|---|---|---|
| Bundled in ERP tier | Hubdoc in Xero PH [S3]; QBO [S9]; Odoo Documents in Standard [S20] | Matches market norms. Simplest for the partner to sell as a one-stop shop. Drives adoption and ERP-record linking. | No direct revenue. Heavy scanners are subsidised. |
| Per user | Odoo $8.95–11.20/user [S20] | Scales with firm size. Familiar to buyers. | Discourages adding occasional users (partners, clerks). Weak link to cost. |
| Flat fee per organisation | Hubdoc $12 [S5]; Juan ₱2,000 [S14] | Predictable. Easy to quote in PHP. | Large tenants underpay. |
| Storage tiers (GB) | DO Spaces 250 GiB/$5 [S28]; Odoo 100 GB [S20] | Directly tracks a real cost. Paper-heavy firms understand "space". | Storage is very cheap (≈ ₱1.6/GB-month on S3 SG), so tiers look like a money grab if set low. |
| Quota per tier plus top-up | Zoho 50 / 200 / 1,000 scans, $10 per 50 [S12] | Caps the expensive feature (AI extraction). Upsell path. | Needs usage metering and billing UI. |
| Prepaid credits per document | AutoEntry [S6]; Odoo IAP [S21] | Matches variable cost exactly | Friction. SMEs dislike running out of credits. Harder for a reseller to quote. |

## 7. (b) PHP price anchors

| Anchor | PHP / month | Basis | Label |
|---|---|---|---|
| Juan Essentials (full accounting) | ₱2,000 per org | Published | **Verified fact** [S14] |
| Taxumo SMB (tax filing) | ₱2,749–6,995 per quarter | Published | **Verified fact** [S17] |
| Sprout Payroll Starter | ₱15,000 (≤ 20 employees), +₱300–400 per employee | Published | **Verified fact** [S18] |
| Hubdoc standalone | ≈ ₱753 | US$12 | **Estimate** |
| Dext Business (250 docs, 5 users) | ≈ ₱1,582 | US$25.21 annual billing | **Estimate** |
| AutoEntry Bronze (50 docs) | ≈ ₱816 | US$13 | **Estimate** |
| Zoho autoscan add-on (50 scans) | ≈ ₱502–628 | US$8–10 | **Estimate** |
| Xero PH Starter / Standard | ≈ ₱1,820 / ₱3,138 | US$29 / US$50, Hubdoc included | **Estimate** |
| Odoo Standard, per user | ≈ ₱562–703 | US$8.95–11.20 | **Estimate** |

## 8. (c) Unit costs

| Cost | Unit price (USD) | ≈ PHP | Source |
|---|---|---|---|
| Storage, S3 Standard SG | $0.025 / GB-month | ₱1.57 | [S23] |
| Storage, DO Spaces overage | $0.02 / GiB-month (first 250 GiB in the $5 base) | ₱1.26 | [S28] |
| Storage, Wasabi / B2 (US-priced) | ≈ $0.008 / $0.007 per GB-month | ₱0.50 / ₱0.44 | [S29][S30] |
| OCR text (Google / Textract / Azure Read) | $1.50 / 1,000 pages | ₱94 | [S31][S25][S33] |
| OCR text, self-hosted Tesseract | ≈ $0.01–0.03 / 1,000 pages (compute only) | ≈ ₱1–2 | [S35][S37] (**Estimate**) |
| Receipt/invoice extraction (Textract Expense / Azure prebuilt) | $10 / 1,000 pages | ₱628 | [S25][S33] |
| Odoo IAP digitisation (resale benchmark) | €50–150 / 1,000 docs | — | [S21] |
| Inbound email, SES SG | $0.10 / 1,000 emails + $0.09 / 1,000 × 256 KB chunks | ₱6 + ₱6 | [S26] |
| Egress, AWS SG | $0.12 / GB (beyond the free tier) | ₱7.53 | [S24] |

## 9. (d) Implications for pricing Officetul's Documents app

1. **Recommendation:** Include the core Documents app in the Officetul ERP subscription, at least in the mid and upper tiers. This covers upload, the per-tenant scan-to-email inbox, phone capture, folders and metadata, permissions, activity log, versions, ERP record links and OCR text search.
   - Every major competitor bundles capture and attachments (Xero/Hubdoc, QBO, Zoho, Odoo).
   - The senior partner's one-stop-shop pitch is stronger with "documents included".
2. **Recommendation:** Run search OCR on self-hosted Tesseract (Filipino and English models).
   - Cost per tenant is a few pesos per month (§4d), which makes bundling viable.
   - Keep a cloud OCR adapter as a fallback for quality ($1.50 per 1,000 pages).
3. **Working assumption:** Use **storage quotas per tenant** as the main fair-use guardrail, sold in generous blocks (tens of GB included).
   - The real cost is about ₱1.6 per GB-month plus VAT. Price add-on blocks at a margin, but do not lead with them.
4. **Recommendation:** If AI field extraction is ever offered (e.g. reading receipt fields into an expense), **meter it**: a monthly quota per tier plus top-up packs, following Zoho's model.
   - Its unit cost ($10 per 1,000 pages) is about 7× plain OCR and 300× Tesseract.
   - The market already meters this: Zoho ≈ $0.20/scan, AutoEntry $0.19–0.26, Odoo €0.05–0.15.
5. **Working assumption:** If the Documents app must be sold as a separate add-on, the market anchors put a flat fee per tenant at roughly **₱500–₱1,600 per month**. That spans Zoho's autoscan pack, Hubdoc and Dext Business.
   - It should stay well below Juan Essentials (₱2,000) so it does not look like a second accounting system.
   - This is an inference, not a validated price.
6. **Recommendation:** Quote in **PHP**, flat **per organisation** (with per-user pricing only if the rest of Officetul is per user), and say explicitly whether **12% VAT** is included.
   - The foreign competitors' USD pricing is a selling point for a local vendor.
   - Budget for the 12% VAT on AWS/Google/Azure bills in the cost model.

## 10. (e) Open questions

- Officetul's current ERP pricing model (per user or per org) and the senior partner's margin. Both are needed to decide between bundling and an add-on.
- Juan's Growth-plan price, and what "Extractions" means (OCR?) [S14].
- Published PHP packages from local **Odoo or SAP Business One partners**, **Kitta** or other local cloud accounting, and any local DMS add-on sold with an ERP. Not researched because the search budget was exhausted.
- Sprout core HRIS per-employee pricing (quote-only), and whether its "201 file" feature counts as document management.
- Xero file-library and per-transaction limits (Xero Central pages render in JS). QBO PH live PHP prices.
- Whether Wasabi or Backblaze have Singapore or other APAC regions, and their minimum-duration rules.
- The exact date RR 3-2025 was published and the date nonresident DSP VAT began. Whether reverse-charged VAT on cloud bills is creditable input VAT for Officetul.
- Data Privacy Act and client expectations about storing tenant documents in Singapore rather than in the PH.
- Real Tesseract throughput on Officetul's server class. The per-page CPU figure is assumed.

## 11. (f) Sources (all accessed 2026-10-18)

| ID | Source |
|---|---|
| S1 | Dext US pricing, https://www.dext.com/us/pricing/prepare-direct/ |
| S2 | Dext on the Xero App Store (PH), https://apps.xero.com/ph/app/dext |
| S3 | Xero PH pricing, https://www.xero.com/ph/pricing-plans/ |
| S4 | Xero US Smart Document Capture, https://www.xero.com/us/accounting-software/capture-data-with-hubdoc/ |
| S5 | Hubdoc pricing, https://www.hubdoc.com/pricing |
| S6 | AutoEntry pricing, https://www.autoentry.com/pricing |
| S7 | Xero Central "Upload a file to the file library" (UK), https://central.xero.com/s/article/Upload-a-file-to-the-file-library-UK. JS page that could not be fetched; figure taken from a search snippet. |
| S8 | QuickBooks US pricing, https://quickbooks.intuit.com/pricing/ |
| S9 | Intuit help, "Email receipts to QuickBooks Online", https://quickbooks.intuit.com/learn-support/en-us/help-articles/email-receipts-to-quickbooks-online/00/513560 |
| S10 | QuickBooks PH pricing, https://quickbooks.intuit.com/ph/pricing/ (prices load live and were not captured) |
| S12 | Zoho Books pricing, https://www.zoho.com/books/pricing/ (zoho.com/ph/books/pricing returns 404) |
| S13 | Zoho Books Documents help, https://www.zoho.com/books/help/documents/documents.html |
| S14 | Juan pricing, https://juan.ac/pricing |
| S15 | Juan home, https://juan.ac/ |
| S16 | JuanTax home, https://juan.tax/ |
| S17 | Taxumo pricing, https://www.taxumo.com/pricing |
| S18 | Sprout Payroll Starter, https://discover.sprout.ph/payroll-starter/ |
| S19 | Sprout home, https://sprout.ph/ (no pricing page; /pricing returns 404) |
| S20 | Odoo pricing, https://www.odoo.com/pricing (also checked `?country=PH`) |
| S21 | Odoo IAP Document Digitization, https://iap.odoo.com/iap/in-app-services/259 |
| S22 | Odoo 18 docs, invoice digitization, https://www.odoo.com/documentation/18.0/applications/finance/accounting/vendor_bills/invoice_digitization.html |
| S23 | AWS Price List API, S3 ap-southeast-1 (pub. 2026-10-15), https://pricing.us-east-1.amazonaws.com/offers/v1.0/aws/AmazonS3/current/ap-southeast-1/index.json. The S3 pricing web page did not render rates. |
| S24 | AWS Price List API, AWSDataTransfer ap-southeast-1, https://pricing.us-east-1.amazonaws.com/offers/v1.0/aws/AWSDataTransfer/current/ap-southeast-1/index.json |
| S25 | AWS Price List API, AmazonTextract ap-southeast-1 (pub. 2026-09-11), https://pricing.us-east-1.amazonaws.com/offers/v1.0/aws/AmazonTextract/current/ap-southeast-1/index.json |
| S26 | Amazon SES pricing, https://aws.amazon.com/ses/pricing/, plus the AmazonSES ap-southeast-1 price list (Message:Receive $0.0001, ReceivedChunk $0.00009) |
| S27 | AWS General Reference, SES endpoints, https://docs.aws.amazon.com/general/latest/gr/ses.html |
| S28 | DigitalOcean Spaces pricing, https://docs.digitalocean.com/products/spaces/details/pricing/ |
| S29 | Wasabi pricing, https://wasabi.com/pricing |
| S30 | Backblaze B2 pricing, https://www.backblaze.com/cloud-storage/pricing |
| S31 | Google Document AI pricing, https://cloud.google.com/document-ai/pricing |
| S32 | Google Cloud Vision pricing, https://cloud.google.com/vision/pricing |
| S33 | Azure Retail Prices API, Document Intelligence, southeastasia, https://prices.azure.com/api/retail/prices (filter `contains(productName,'Document Intelligence') and armRegionName eq 'southeastasia'`) |
| S34 | Azure Document Intelligence pricing page, https://azure.microsoft.com/en-us/pricing/details/ai-document-intelligence/ (rendered "$-"; free tier of 500 pages taken from here) |
| S35 | Tesseract repository, https://github.com/tesseract-ocr/tesseract |
| S36 | Tesseract Filipino model, https://raw.githubusercontent.com/tesseract-ocr/tessdata_fast/main/fil.traineddata |
| S37 | DigitalOcean Droplet pricing, https://www.digitalocean.com/pricing/droplets |
| S38 | RA 12023 (Lawphil), https://lawphil.net/statutes/repacts/ra2024/ra_12023_2024.html (Official Gazette page returned 403) |
| S39 | BIR RR No. 3-2025, https://bir-cdn.bir.gov.ph/BIR/pdf/RR%20No.%203-2025.pdf (scanned PDF read page by page) |
| S40 | RA 10963 (TRAIN) (Lawphil), https://lawphil.net/statutes/repacts/ra2017/ra_10963_2017.html |
| S41 | BSP daily PHP/USD rate, https://www.bsp.gov.ph/statistics/external/day99_data.aspx |
| S42 | Xero PH home, https://www.xero.com/ph/ |
