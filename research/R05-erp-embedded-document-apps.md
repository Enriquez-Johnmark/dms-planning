# R05: Document apps built into ERPs (Odoo, ERPNext, Zoho, Business Central, NetSuite, SAP B1)

**Question:** How do document apps built into ERPs work, what do they offer, and how are they priced? What should Officetul's Documents app copy, avoid, or do differently?
**Researched:** 2026-10-18. **Every source was accessed on 2026-10-18.** Each claim cites a source ID like `[S3]`. The full URLs are in section 12.

**Labels:**
- **Verified fact (VF):** read on an official page, in official docs or in the vendor's own code.
- **Vendor claim (VC):** marketing copy.
- **Anecdotal (AN):** a forum post or review.
- **Open question (OQ):** not confirmed.

**Limits on this research:**
- The web-search quota for the session ran out partway through. After that, only pages whose URLs were already known could be fetched, so a few items stay **OQ** (listed in section 10).
- Some pages blocked fetching or hid their prices:
  - Zoho help pages under `/books/help/` returned HTTP 403, and the Wayback Machine had no copy. I used the India-locale help page [S14] instead.
  - Zoho One and WorkDrive pricing pages load their prices with JavaScript. The amounts did not show, apart from one Zoho One figure.
  - The NetSuite pricing page returned HTTP 403.
  - SAP Help pages load with JavaScript, so I used SAP's Service Layer API reference [S27].

---

## 1. Odoo Documents (v19)

### Edition and price

| Item | Finding | Label |
|---|---|---|
| Edition | The official editions table puts Documents (and its Spreadsheet sub-row) in **Enterprise only**. Community is marked ✗. Sign is also Enterprise only [S2]. | VF |
| Not in open-source code | `addons/documents` does not exist in the public `odoo/odoo` repo on branches 18.0 or 19.0 (GitHub returned 404) [S3]. | VF |
| Plan prices (USD) | <ul><li>One App Free: US$0, one app only.</li><li>Standard: US$8.95 per user per month for the first 12 months for the initial users, then US$11.20. Includes "All apps", Odoo Online only.</li><li>Custom: US$16.40, then US$20.40. Adds Studio, agentic AI, multi-company and the external API, with Online, Odoo.sh or on-premise hosting.</li><li>Light User: US$2.90.</li></ul>[S4] | VF |
| Portal users | "External users (such as clients or suppliers) using Odoo through the portal are not paying users." [S4] | VF |
| Price in PHP | The pricing page's currency settings include PHP (symbol "Php"), but no PHP amounts appeared to the fetcher [S4]. | OQ |
| Storage | "hosting on Odoo Online is free, with unmetered storage up to 100GB" [S4]. The limit is **64 MB per file** on Odoo Online [S1]. | VF |
| Hidden upgrade | Folder **Automations** "requires activating Studio, which may impact your pricing plan" [S1]. Installing Studio moves the customer to the Custom plan [S4]. **AI Auto-sort** needs the Odoo AI app [S1]. | VF |
| OCR cost (IAP, EUR) | <ul><li>Document Digitization: "Digitizing one document consumes one credit"; the page also shows "200 free credits".</li><li>Packs: 100 for €15 (€0.15 each), 1,000 for €120, 10,000 for €1,000, 100,000 for €5,000 (€0.05 each).</li><li>Credits never expire.</li><li>Page limits per document: invoices 8, expenses 1, bank statements 30.</li></ul>[S5] The pricing page also lists "IA to scan vendor bills & expenses" as not included in plans [S4]. | VF |

### Features [S1] unless noted (all VF)

- **Structure:**
  - Sections: Company, My Drive, Shared with me, Recent, Trash.
  - Nested folders.
  - **Shortcuts:** "a pointer to a file, allowing access from multiple folders without duplicating the file."
  - Tags.
  - A **Contact** field. The contact "only has Viewer access rights."
- **Links to ERP records:**
  - A single **"Linked to"** field (model plus record) in the details panel.
  - With *file centralization* on, an attachment uploaded in another app "automatically adds the corresponding record in the Linked to field."
  - From any record's chatter, **Add from Documents** inserts either the raw file or a link that "keeps the document's access rights."
- **Centralization rules:**
  - Turned on by default when each app is installed. Accounting gets one sub-folder per journal type, and centralization "cannot be disabled for Accounting documents." HR gets one sub-folder per employee.
  - Changes to centralization settings affect only new files.
  - Deleting the source record "moves its attachments to the trash in the Documents app."
  - Odoo 19 made the Accounting–Documents link "automated and enabled by default" [S6].
- **Capture:**
  - Upload and drag-and-drop.
  - **An email alias per folder.** Optionally, an alias can create an activity (type plus assignee) when a file arrives.
  - In v19, emails sent to an alias with no attachment become a file [S6].
- **Document requests:** Creates a placeholder file in a folder, with an assignee activity and reminder emails (one at a time or in bulk) from the Activity view.
- **Sharing:**
  - Roles are Viewer or Editor, per user or per partner.
  - "Access through link" can be marked Discoverable.
  - Each permission can have an **expiration date**.
  - Portal users see their files in the customer portal.
  - In v19, rights changes, shares and renames are logged in the chatter, and a warning appears when a move changes access rights [S6].
- **Versions and locking:**
  - "Manage versions: View all versions of the file in upload order", with download or upload of a version.
  - **Lock**: "Protect the file from any modifications."
  - Odoo does not document an "official/approved version" marker. **OQ**
- **Workflow:**
  - Per-folder **Actions on Select**: server-action buttons, for example create a vendor bill.
  - Automations through Studio.
  - AI Auto-sort.
  - Split and merge PDFs.
  - Odoo 19 adds server actions that create journal entries from documents [S6].
- **OCR:** Files in the Finance folder → Create Vendor Bill / Customer Invoice → "Send for Digitization", which uses IAP credits.
- **Retention:** Trash is kept for 30 days by default and can be configured. **No retention schedule or legal hold is documented.** **OQ / likely absent**
- **Full-text search inside PDFs:** The official docs don't say whether this exists. Several third-party "search in attachments" and "searchable OCR PDF" modules exist on the App Store [S7]. Treat the feature as **OQ** for core.
- **Spreadsheets:** Odoo spreadsheets live in Documents (Enterprise) [S1][S2].

### Complaints (AN)
- On Community, Documents "is an Enterprise Edition App" and can't be installed [S8].
- No desktop sync for Odoo Online. The accepted forum answer says third-party modules can't be installed on Standard Online and need the Custom plan [S9].
- One user on v17 couldn't give users admin rights over only their own workspace [S10].
- A third-party vendor says there is "no consistent in-record viewer" for PDF and Office files where attachments live [S11]. This is **VC**, from a competitor's sales page.

---

## 2. ERPNext / Frappe

| Item | Finding | Label |
|---|---|---|
| Attachment model | Every upload is a **`File` record**. It has `attached_to_doctype`, `attached_to_name`, `attached_to_field`, `folder`, `is_private`, `content_hash`, a `blob` link to `File Blob`, `file_size`, and flags for Dropbox and Google Drive backups [S12]. **A file belongs to at most one record.** Folders are themselves `File` rows with `is_folder`, and there are built-in Home and Attachments folders [S12]. | VF (source code) |
| Permissions | `has_permission()` allows access in this order: Administrator → public files (read) → owner → files explicitly shared → **otherwise inherits from the attached document** (the code loads the referenced record to check) [S12]. Private files sit under `/private/files` and are served only after this check. | VF (source code) |
| Limits | System Settings has `max_file_size` (MB), `allowed_file_extensions`, and options to strip EXIF data and restrict public uploads. The default limit is **25 MB** (`get_max_file_size`) [S13]. | VF |
| Versioning | The `File` doctype has `track_changes: 1`, so metadata changes are logged. **There is no file-content versioning.** Replacing a file creates a new File row. Deduplication uses a content hash and a shared blob [S12]. | VF (code); "no content versioning" is my inference from the schema |
| Frappe Drive | <ul><li>AGPL-3.0, 738 stars. Features: folder uploads, previews, sharing with users, groups or the public, per-user quotas, activity logs, and a "Writer" editor with manual and automatic versions.</li><li>**The repo was archived and set read-only in 2026: "Frappe Drive is archived and is now part of Suite."** Last release: v0.3.0 (2025-10-08) [S15].</li><li>`frappe/suite` is labelled **"[WIP]"** and bundles Drive, Writer, Sheets, Slides, Meet, Mail and Calendar [S16].</li><li>Drive is still listed **free** on the Frappe Cloud Marketplace (Frappe Tech; v15, v16, Nightly; about 7.1k installs) [S17].</li><li>The repo doesn't describe any **link to ERPNext record attachments** or any OCR [S15].</li></ul> | VF |
| OCR | Nothing in core. The best-known add-on is `Monogramm/erpnext_ocr` (MIT, Tesseract), which labels itself **"Experimental"**; 108 stars, last push 2026-02-17 [S18]. Other OCR and DMS apps found are hobby-scale (0–2 stars) [S18]. | VF |
| Hosting price | Frappe Cloud runs from **US$5/mo (₹410)** for a site and **US$40/mo (₹3,600)** for a server. The pricing page states no storage amount [S19]. ERPNext itself is open source (GPL), with no per-user licence. | VF / OQ for storage |

---

## 3. Zoho Books, WorkDrive and Zoho One

| Item | Finding | Label |
|---|---|---|
| Documents module | <ul><li>"The Documents Inbox is the primary folder." Inbox files "are yet to be attached to a transaction and hence act as a reminder."</li><li>"Zoho Books Documents module generates a **unique email address for your organisation**." Vendor receipts and bills can be forwarded to it, and bank statements can be emailed in (CSV, PDF, XLS, OFX, QIF, CAMT.053) [S14].</li><li>**Every** emailed or uploaded document "goes through automatic data capturing" (autoscan, 15 languages). Failures are marked *Unreadable* [S14].</li><li>Folders can carry access permissions.</li><li>Role permissions: View, Upload, Delete, and Manage Folder.</li><li>"Once the document is cleared from your inbox, you will not be able to retrieve it." [S14]</li></ul> | VF |
| Matching | Bills and expenses are created side by side with the document image. Zoho suggests matching documents by vendor name, date and amount [S20]. | VC |
| Portal | Customers "view, download, and upload files tied to their transactions" through the client portal [S20]. | VC |
| Attachment limits | "You can attach up to **10 files (5 MB each)** to the invoices." Sources: device, cloud (WorkDrive, Google Drive, Dropbox, OneDrive), or the Documents module [S21]. Whether a WorkDrive file is copied or linked is **OQ**. | VF / OQ |
| Pricing (USD, per organisation per month, billed annually) | <ul><li>Free $0 (1 user)</li><li>Standard $10 (3 users)</li><li>Professional $20 (5)</li><li>Premium $30 (10)</li><li>Elite $100 (10)</li><li>Ultimate $200 (15)</li></ul>**Autoscans:** 50 a month on Free and trial, 200 on Standard and Professional, 1,000 on Premium and above. The add-on is $8 a month (annual) for 50 extra scans [S22]. The comparison grid includes "Create folders / Upload documents / Attach files to transactions and contacts / Auto forward receipts and bills / Auto forward bank statements" [S23]. | VF |
| WorkDrive (separate product) | <ul><li>Starter, Team and Business plans, minimum 3 users.</li><li>Storage: 1 TB, 3 TB or 5 TB base.</li><li>Starter includes "Unlimited File Versions", "Advanced Search with OCR and Object Detection" and "Customisable Data Retention Policies".</li><li>**Prices are rendered by JavaScript and were not visible** [S24].</li></ul> | VF (features) / OQ (price) |
| Zoho One | Flexible-user Standard plan: **US$90 per user per month** (annual licensing). The all-employee price was not shown, and the page didn't confirm which apps are included [S25]. | VF / OQ |
| Complaints | <ul><li>A user waited 1.5 hours for an OCR scan [S26].</li><li>Only Inbox files are autoscanned, not files placed straight into folders (Zoho reply as summarised in search results, not opened).</li><li>WorkDrive can't be picked from the invoice email composer. Zoho said it has no immediate plans to add this (seen in search results, not opened).</li></ul> | AN |

---

## 4. Microsoft Dynamics 365 Business Central

| Item | Finding | Label |
|---|---|---|
| Two separate systems | **(a) Attachments FactBox**: files, links and notes on most cards, documents and lines. **(b) Incoming Documents**: an inbox-style register. "Files attached with the Incoming Documents feature aren't included on the Attachments tab" [S28][S29]. | VF |
| Attachments behaviour | <ul><li>Attachments "stay attached to the card or document while it's processed", for example from sales order to posted invoice.</li><li>On partial invoicing, the attachment goes only to the *final* invoice.</li><li>Deleting an un-invoiced order deletes its attachments.</li><li>PDF and image preview.</li><li>"Attach as PDF" snapshots a document.</li><li>Attachments are **not** output when printing [S28].</li></ul> | VF |
| Incoming Documents | <ul><li>Create from file, a manual record, or **Create from Camera** in the mobile app.</li><li>Records can be linked to non-posted documents *and* to posted vendor, customer and G/L entries.</li><li>"Find posted documents without incoming document records" is an audit-gap finder.</li><li>Records can be marked *Processed*.</li><li>**OCR comes from external providers** found on Microsoft Marketplace. Some providers offer forward-to-email capture [S29].</li></ul> | VF |
| AI email capture | The **Payables Agent** watches a mailbox, preferably a shared one. Each PDF becomes an *Inbound E-Document*. Extraction uses **Azure Document Intelligence**. It drafts a purchase invoice for a supervisor to review. **Billed in Copilot Credits.** Limits: ≤10 attachments per email, ≤10 pages and ≤5 MB per PDF, 100 emails and 500 invoices a day. "Doesn't support approval flows" [S30]. | VF |
| Storage | <ul><li>**80 GB per tenant**, plus 3 GB per Premium licence, 2 GB per Essentials licence and 1 GB per Device licence.</li><li>Files and blobs in the database count against this.</li><li>Going over **does not stop transactions** but blocks creating new environments.</li><li>Extra capacity comes in 1 GB and 100 GB add-ons [S31].</li></ul> | VF |
| External storage | <ul><li>Attachments can be offloaded to **Azure Blob, Azure Files or SharePoint**, with folders by environment, company and table, and a unique ID appended to each file name.</li><li>"Microsoft doesn't back up or manage those files". If a file is deleted outside BC, the link breaks.</li><li>Storage Sync migrates files in either direction [S32].</li><li>OneDrive: "Open in OneDrive" or "Share" **copies** the file to OneDrive [S33].</li></ul> | VF |
| Price (USD, per user per month, paid yearly) | Essentials **$80**, Premium **$110**, Team Members **$8**. "Prices shown are for informational purposes only." [S34] | VF |

---

## 5. NetSuite File Cabinet and SAP Business One (brief)

### NetSuite

| Item | Finding | Label |
|---|---|---|
| File Cabinet | <ul><li>Files are stored in folders and attached to customers, transactions and so on.</li><li>"You can't restrict edit access at the file level… only to the folder."</li><li>Every file gets **system notes** (an audit trail). JSON, XML and JS files also get a content hash [S35].</li></ul> | VF |
| Linking | **Many-to-many.** On a record's Communication > Files subtab, choose "Attach Existing Files" from the Cabinet or upload into a folder and attach [S36]. | VF |
| Permissions | <ul><li>Requires the Documents and Files permission.</li><li>Users without it can open a file by URL **only if they can access the related record**, for example a Customer Center user viewing an invoice file [S37].</li><li>Folders can be restricted by group, subsidiary, department or location.</li><li>New subfolders inherit the parent's restriction, but later changes to the parent don't flow down [S38].</li></ul> | VF |
| Capture and OCR | **Bill Capture:** email in (through the Transaction Email Capture SuiteApp) or upload PDF, JPEG or PNG files. Extraction uses the OCI Document Understanding generative model ("Built with Llama"). The bill and file show side by side. Existing approval workflows still apply [S39]. **Available only in the United States.** Limits: one bill per file, ≤20 attachments per email, ≤8 MB per file [S40]. | VF |
| Versioning, storage, price | Not confirmed. The pricing page blocked fetching (403) [S41]. | OQ |

### SAP Business One

| Item | Finding | Label |
|---|---|---|
| Model | An `Attachments2` object holds `Attachments2_Lines` (`FileName`, `FileExtension`, `SourcePath`, …). Business objects such as ProjectManagements, Resources and TransportationDocument point to it through an **`AttachmentEntry`** integer. That is **one attachment set per record**, and a set can hold many files [S27]. | VF (API reference) |
| Storage, versioning, OCR, DMS | The files sit in a configured attachments folder on the server, not in the database. Native document management is minimal, and DMS and OCR come from partner add-ons. | OQ (common knowledge; I couldn't open SAP Help to confirm) |

---

## 6. Patterns across the products

| Dimension | Odoo | ERPNext | Zoho Books | Business Central | NetSuite | SAP B1 |
|---|---|---|---|---|---|---|
| Document ↔ record | One "Linked to" record per document, plus chatter links that keep the document's access rights, plus folder shortcuts [S1] | Exactly 1 record per File [S12] | Attach from the Documents module to transactions. **Whether one file can attach to several transactions is unclear (OQ).** [S14] | 1:1 attachment rows. Incoming Doc ↔ document or posted entries [S28][S29] | **Many-to-many** (attach existing) [S36] | 1 attachment set per record (AttachmentEntry) [S27] |
| Where files live | Odoo filestore, 100 GB on Online [S4] | Local `private/files` or `public/files`, deduplicated blobs [S12] | Zoho cloud | **Inside the database** (counts toward quota), or offloaded to Azure or SharePoint [S31][S32] | File Cabinet [S35] | Server folder (OQ) |
| Versions | Upload-order versions plus Lock [S1] | Metadata log only [S12] | Not documented (OQ) | None documented | System notes, no versioning confirmed (OQ) | OQ |
| Email-in | **One alias per folder**, which can create an activity [S1] | None in core | **One address per organisation**, plus bank-statement email [S14] | Outlook add-in, OCR-provider email, or a Payables Agent mailbox [S29][S30] | Bill Capture email (US only) [S39] | OQ |
| Inbox / review | Folder plus activities, and a Requested column | — | **Inbox = attachment not yet used**, a reminder queue [S14] | Incoming Documents list with *Processed* flag [S29] | Scanned Vendor Bills [S39] | — |
| Permissions | Folder and file Viewer/Editor, link access, per-person expiry [S1] | **Inherit from the attached record** [S12] | Module roles plus folder permissions [S14] | ERP permission sets | **Folder-only**, with the record-access URL fallback [S37][S38] | OQ |
| Quota | 64 MB per file; 100 GB on Online [S1][S4] | 25 MB per file default [S13] | 5 MB per file, 10 files per invoice [S21] | 80 GB + 2–3 GB per user, database [S31] | OQ | OQ |

**What the patterns show:**
1. Two models compete. In the **record-owned** model (ERPNext, SAP B1, BC Attachments), access comes from the record and files can't be shared or reused. In the **folder-owned** model (Odoo, NetSuite, Zoho), files are reusable but permissions follow the folder, which creates gaps. Odoo, the most mature, mixes both: the folder governs access, there's one "Linked to" record, and links from records keep the document's own access rights.
2. **Email-in plus extraction is now an AI and metered feature everywhere:**
   - Odoo: €0.05–0.15 per document.
   - Zoho: 50–1,000 scans a month by plan.
   - BC: Copilot Credits.
   - NetSuite: US only.
3. **Hardly any embedded app has retention or legal hold.** Only standalone storage products do: WorkDrive offers "Customisable Data Retention Policies" [S24], and BC can offload to SharePoint [S32].

---

## 7. Comparison table

| Product | Edition / price | Record links | Email-in | OCR / extraction | Versions | Approvals | Retention | Storage quota |
|---|---|---|---|---|---|---|---|---|
| **Odoo Documents 19** | Enterprise only. Standard US$11.20, Custom US$20.40 per user per month (intro $8.95 / $16.40). Automations need Studio, so the Custom plan [S1][S2][S4] | One "Linked to" record, plus links from records that keep access rights; auto-centralised from Accounting and HR [S1] | Alias per folder [S1] | IAP €0.05–0.15 per document; AI auto-sort through the Odoo AI app [S1][S5] | Upload-order history, plus Lock [S1] | Not native in Documents (Studio automations) [S1] | 30-day trash only (OQ for more) [S1] | 100 GB on Online; 64 MB per file [S1][S4] |
| **ERPNext + Drive** | Free and open source; Frappe Cloud from US$5/mo [S19] | 1 record per File; permissions inherited [S12] | None in core | Experimental third-party Tesseract app [S18] | Metadata log; Drive Writer docs only [S12][S15] | ERPNext workflows (not file-specific; OQ) | None found | 25 MB per file default; per-user quotas in Drive [S13][S15] |
| **Zoho Books** | US$0–200 per organisation per month (annual) [S22] | Attach to transactions and contacts; matching suggestions [S14][S23] | One address per organisation [S14] | Autoscan 50/200/1,000 a month; add-on $8 for 50 [S22] | OQ | OQ | Cleared Inbox items are unrecoverable [S14] | 5 MB per file, 10 per invoice [S21] |
| **Zoho WorkDrive** | Price not visible (JavaScript) [S24] | Picked from Books [S21] | OQ | OCR search [S24] | Unlimited versions (Starter) [S24] | OQ | Configurable policies [S24] | 1–5 TB base [S24] |
| **Business Central** | Essentials US$80, Premium $110, Team Members $8 per user per month [S34] | Attachments 1:1; Incoming Doc ↔ document or posted entries [S28][S29] | OCR provider, Outlook add-in, Payables Agent [S29][S30] | Third-party OCR; Azure Document Intelligence via the agent (Copilot Credits) [S29][S30] | None documented | Purchase approval workflows apply after the agent [S30] | None documented; external storage is customer-managed [S32] | 80 GB + 2–3 GB per user, database [S31] |
| **NetSuite** | Quote only (pricing page 403) [S41] | Many-to-many [S36] | Bill Capture (US) [S39][S40] | OCI generative model [S39] | OQ | SuiteApprovals for bills [S39] | OQ | OQ |
| **SAP B1** | OQ | 1 set per record [S27] | OQ | Partners (OQ) | OQ | OQ | OQ | OQ |

---

## 8. Patterns worth copying (Recommendation)

1. **One email address per folder, set to create a task when a file arrives** (Odoo [S1]). For Officetul, give each tenant an inbox address plus optional per-folder addresses, for example `bills@…` going to Finance/Inbox.
2. **Treat the inbox as a to-do list: files sitting there are still unfiled** (Zoho [S14]; BC *Processed* flag [S29]). Track Inbox items until someone files or links them, and allow bulk "mark processed".
3. **An audit tool that finds ERP records with no supporting document** (BC "Find posted documents without incoming document records" [S29]). For PH audit and BIR work, an "expenses without receipt" report has high value and is cheap to build.
4. **Document requests: a placeholder file, a named assignee, and reminders** (Odoo [S1]). This maps to law and accounting firms chasing client papers, and leads into the client portal.
5. **Links from a record that keep the document's access rights**, rather than copies (Odoo "Add from Documents → link" [S1]). Copying is what BC/OneDrive [S33] and possibly Zoho/WorkDrive do, and it creates duplicates.
6. **Extract data side by side, then a person confirms the draft** (Zoho [S20], NetSuite [S39], BC Payables Agent [S30]). Never post from OCR automatically.
7. **Expiry dates on each permission, and log every rights change in the activity log** (Odoo v19 [S1][S6]).
8. **Store each file once by content hash, even when it's linked several times** (Frappe `content_hash` and `File Blob` [S12]).
9. **Tell the user when an action will change who can see a file** (Odoo v19 warning [S6]).

## 9. Pitfalls and complaints to avoid

1. **Hiding the documents app or its automations behind the top tier.** In Odoo, Documents is Enterprise only and automations force the Custom plan [S2][S4]. (AN: Community users can't install it at all [S8].)
2. **Two separate file stores for the same record** (BC Attachments vs Incoming Documents [S28][S29]). Users can't find files. Use one document store with typed links.
3. **Permissions only at folder level** (NetSuite: "can't restrict files on an individual basis" [S37]; parent changes don't reach existing subfolders [S38]). Silent permission drift.
4. **Permissions only from the record** (ERPNext [S12]). A file can't be shared outside the record's audience or reused without duplication.
5. **Destructive inbox clearing** (Zoho: "you will not be able to retrieve it" [S14]). Always soft-delete.
6. **Charging per document for OCR without warning** (Odoo IAP [S5], Zoho caps [S22], BC Copilot Credits [S30]). Users only learn about it at the bill.
7. **Small per-file limits** (Zoho 5 MB [S21], BC agent 5 MB and 10 pages [S30]). Scanned contracts and construction drawings are bigger.
8. **Offloaded storage that is "your problem"** (BC: Microsoft doesn't back it up, and broken links follow deletion [S32]).
9. **Relying on an add-on ecosystem that may disappear** (Frappe Drive archived into a WIP Suite [S15][S16]; ERPNext OCR "Experimental" [S18]).
10. **OCR delays without status** (AN [S26]).

## 10. Where Officetul's Documents app could differ (Working assumption)

- **Built in on every tier, not an upsell.** Documents is a selling point for paper-heavy PH firms. Odoo locks it to Enterprise, and ERPNext's equivalent is archived.
- **Links as their own records, many-to-many** (document ↔ any ERP record, with a relationship label such as "receipt for", "contract of"). Copy NetSuite's reuse [S36] and Odoo's links that keep access rights [S1]. Avoid single-owner attachment models.
- **Combined permissions:** a user can see a document if they have folder or document rights, **or** if they can see a linked record and the link type allows it. Show the reason in the UI ("visible because linked to Expense #123"). No competitor reviewed explains this.
- **A first-class "official version" marker plus revision history.** None of the products reviewed documents this. Odoo has only upload-order versions and Lock [S1].
- **Full-text OCR search in every PDF and image, filtered by permissions, with no per-document fee.** Charge for AI *extraction* only, with a visible meter. Competitors meter OCR or don't search file contents in core.
- **Retention and legal hold in the next release.** This would set it apart from every ERP-embedded app reviewed. Only standalone storage such as WorkDrive has retention policies [S24].
- **PH-specific defaults:** document types for BIR 2307, official receipts, SEC/GIS and permits, with expiry reminders. An "unsupported expense" report like BC's.

## 11. Open questions

1. Odoo Documents prices in PHP for the Philippines, and whether intro discounts apply there.
2. Does Odoo core do full-text search inside PDFs in Documents, or only metadata?
3. Does Odoo, or any product reviewed, support retention or legal hold natively?
4. Zoho: can one document attach to several transactions? Are WorkDrive files copied or linked? What are the WorkDrive and Zoho One all-employee prices? Is Autoscan available for PH organisations?
5. NetSuite: File Cabinet storage limits, versioning behaviour, and whether Bill Capture will reach the PH market.
6. SAP B1: attachments folder behaviour, versioning, and partner DMS pricing (SAP Help couldn't be read).
7. Is there an approval workflow for BC Incoming Documents?
8. Frappe Suite: timeline, and whether Drive will link to ERPNext record attachments.

## 12. Sources (all accessed 2026-10-18)

| ID | URL |
|---|---|
| S1 | https://www.odoo.com/documentation/19.0/applications/productivity/documents.html |
| S2 | https://www.odoo.com/page/editions |
| S3 | https://api.github.com/repos/odoo/odoo/contents/addons/documents?ref=19.0 (and ref=18.0), both 404 |
| S4 | https://www.odoo.com/pricing |
| S5 | https://iap.odoo.com/iap/in-app-services/259 |
| S6 | https://www.odoo.com/odoo-19-release-notes |
| S7 | https://apps.odoo.com/apps/modules/19.0/sync_document_ocr ; https://apps.odoo.com/apps/modules/13.0/smile_attachment |
| S8 | https://odoo.com/forum/documents-9/documents-app-and-approvals-in-odoo15-community-197759 ; https://odoo.com/hi_IN/forum/help-1/is-odoo-documents-dms-avaliable-in-ce-142029 |
| S9 | https://www.odoo.com/forum/documents-9/windows-desktop-sync-with-odoo-online-documents-284362 |
| S10 | https://www.odoo.com/forum/documents-9/access-rights-for-individual-user-workspaces-248256 |
| S11 | https://ecosire.com/ar/apps/odoo/documents-preview |
| S12 | https://github.com/frappe/frappe/blob/develop/frappe/core/doctype/file/file.json ; …/file/file.py |
| S13 | https://github.com/frappe/frappe/blob/develop/frappe/core/api/file.py ; …/core/doctype/system_settings/system_settings.json |
| S14 | https://www.zoho.com/in/books/help/documents/documents.html (US/global help returned 403; no Wayback copy) |
| S15 | https://github.com/frappe/drive (and the GitHub API for its releases and archived status) |
| S16 | https://github.com/frappe/suite |
| S17 | https://cloud.frappe.io/marketplace/apps/drive |
| S18 | https://github.com/Monogramm/erpnext_ocr ; GitHub repo search "ocr frappe" |
| S19 | https://frappe.io/cloud/pricing |
| S20 | https://www.zoho.com/books/document-management/ |
| S21 | https://www.zoho.com/sa/books/kb/invoices/multiple-attachments.html |
| S22 | https://www.zoho.com/books/pricing/ |
| S23 | https://www.zoho.com/books/pricing/pricing-comparison.html |
| S24 | https://www.zoho.com/workdrive/pricing.html |
| S25 | https://www.zoho.com/one/pricing.html |
| S26 | https://help.zoho.com/portal/en/community/topic/ocr-scan-delays |
| S27 | https://help.sap.com/doc/056f69366b5345a386bb8149f1700c19/10.0/en-US/Service%20Layer%20API%20Reference.html |
| S28 | https://learn.microsoft.com/en-us/dynamics365/business-central/ui-how-add-link-to-record |
| S29 | https://learn.microsoft.com/en-us/dynamics365/business-central/across-income-documents |
| S30 | https://learn.microsoft.com/en-us/dynamics365/business-central/payables-agent |
| S31 | https://learn.microsoft.com/en-us/dynamics365/business-central/dev-itpro/administration/tenant-admin-center-capacity |
| S32 | https://learn.microsoft.com/en-us/dynamics365/business-central/across-store-document-attachments-externally |
| S33 | https://learn.microsoft.com/en-us/dynamics365/business-central/across-onedrive-overview |
| S34 | https://www.microsoft.com/en-us/dynamics-365/products/business-central/pricing |
| S35 | https://docs.oracle.com/en/cloud/saas/netsuite/ns-online-help/chapter_N541319.html |
| S36 | https://docs.oracle.com/en/cloud/saas/netsuite/ns-online-help/section_N490137.html |
| S37 | https://docs.oracle.com/en/cloud/saas/netsuite/ns-online-help/section_4799276714.html |
| S38 | https://docs.oracle.com/en/cloud/saas/netsuite/ns-online-help/section_N542073.html |
| S39 | https://docs.oracle.com/en/cloud/saas/netsuite/ns-online-help/article_164726334180.html |
| S40 | https://docs.oracle.com/en/cloud/saas/netsuite/ns-online-help/article_0417034348.html |
| S41 | https://www.netsuite.com/portal/products/erp/pricing.shtml (403, blocked) |
