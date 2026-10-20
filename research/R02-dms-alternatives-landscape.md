# R02 — DMS Alternatives Landscape (first-pass snapshot)

- **Purpose:** frame the "should we build a DMS for Philippine SMEs at all?" discussion. The initial candidate segments are law firms, accounting/bookkeeping firms and construction contractors.
- **Scope:** a landscape snapshot, not a deep comparison.
- **Research date:** 2026-10-18. **Every URL in this document was accessed 2026-10-18** unless a different date is noted inline (for example, a Wayback snapshot).

**Claim labels**

- **[VF] Verified fact.** Confirmed on an official product page, official docs, a GitHub/GitLab/PyPI/Docker Hub record, or a primary legal text.
- **[VF-2] Verified fact (secondary).** Confirmed through reputable secondary professional sources (Big-4 tax alerts, law-firm commentary, news) but not the primary text.
- **[VC] Vendor claim.** A vendor's or reseller's self-description that we have not independently confirmed.
- **[AN] Anecdotal.** Practitioner reviews (G2, Capterra, Reddit, forums) or third-party price estimates. These indicate common complaints only.
- **[OQ] Open question.** We could not verify it, or sources conflict.

**Feature terms**

- **Native:** included in the product or edition named.
- **Paid edition:** only in a higher commercial tier.
- **Add-on:** a separately priced module.
- **Plugin/app:** a separately installed extension, official or third-party.

**Method note**

- Some official pages blocked automated fetching (HTTP 403 or timeouts): clio.com/pricing, netdocuments.com, taxdome.com/pricing, autodesk.com, microsoft.com/en-ph, docs.paperless-ngx.com, and laserfiche.com/pricing on re-check. Each fallback is noted where it was used (Wayback snapshot, the docs source in the repo, official help center, or search snippet).
- No price in this document is invented. Where a vendor publishes no price, it says "contact sales".

---

## 1. Open-source / self-hosted

### 1.1 Paperless-ngx
- **License and release:** GPL-3.0. Latest release v3.3.2, published 2026-10-16. The repo was pushed 2026-10-18 and has about 46.8k stars. **[VF]** https://github.com/paperless-ngx/paperless-ngx/releases
- **Permissions:** global permissions plus per-object owner/view/edit permissions for users and groups. **[VF]** https://docs.paperless-ngx.com/usage/#permissions (read from the docs source in the repo, because the site returned 403)
- **Versioning:** native "document file versions" since 3.0.0. Versions hold the file and its OCR text; metadata stays on the root document. **[VF]** https://docs.paperless-ngx.com/usage/#document-file-versions
- **Approval workflow:** not native. "Workflows" are trigger→action automation (assign metadata or permissions, email, webhook), not review/approve. **[VF]** https://docs.paperless-ngx.com/usage/#workflows
- **Audit trail:** native. There is a per-document History tab, and the audit log has been on by default since 2.7. **[VF]** https://docs.paperless-ngx.com/usage/#document-history
- **OCR:** native (OCRmyPDF + Tesseract). Extra languages are added through `PAPERLESS_OCR_LANGUAGES`. The docs do not name Filipino, but Tesseract ships `fil` (§5.1). **[VF]** for the mechanism, **[OQ]** for Filipino quality. https://docs.paperless-ngx.com/configuration/#PAPERLESS_OCR_LANGUAGES
- **Multi-tenancy:** none documented. It is a single instance with per-user ownership. **[VF]**
- **Hosting:** Docker Compose with Redis/Valkey and PostgreSQL (recommended), MariaDB or SQLite. It is light enough for a Raspberry Pi. **[VF]** https://docs.paperless-ngx.com/setup/
- **Positioning:** Paperless-ngx is designed as a personal or small-office *archive*. It is not a collaborative document-control system. **[OQ]** whether SMEs would accept it without approvals.

### 1.2 Mayan EDMS
- **License:** GPL-2.0-only per the LICENSE file and PyPI. The features page wrongly calls it "permissive". **[VF]** https://gitlab.com/mayan-edms/mayan-edms/-/blob/master/LICENSE
- **Releases:** the latest is 4.12.2 (PyPI, 2026-09-16), preceded by 4.12 (2026-08-17). Older branches still received patches in 2026. **[VF]** https://pypi.org/project/mayan-edms/#history
- **Maintenance risk:** the official repo is on GitLab, last active 2026-10-12. The GitHub mirror says it is "no longer maintained and very outdated". Copyright and trademark belong to a single individual. **[VF]** https://github.com/mayan-edms/Mayan-EDMS
- **Features (all native):**
  - Role-based ACLs with granular permissions.
  - Versions with revert, and check-out/check-in.
  - A state-machine workflow engine that can model review and approval, with escalation and expiry.
  - Every action logged as an event for auditing.
  - **[VF]** https://docs.mayan-edms.com/chapters/features.html
- **OCR:** Tesseract backend (pluggable) on distributed workers. **[VF]** Filipino support is **[OQ]** (should work if `fil` is installed).
- **Multi-tenancy:** not documented. **[OQ]**
- **Hosting:** heavy. The official Compose file runs PostgreSQL, RabbitMQ, Redis, optional Elasticsearch, Traefik and 6+ Celery workers. **[VF]** https://gitlab.com/mayan-edms/mayan-edms/-/blob/master/docker/docker-compose.yml

### 1.3 OpenKM (Community vs Professional)
- **Licensing change (important):**
  - The GitHub repo is **archived**. Its last source release is v6.3.13 under GPL-2.0. **[VF]** https://github.com/openkm/document-management-system
  - From 7.0 onward, Community Edition is "distributed in binary format only, free of charge", so **it is no longer open source**. **[VF]** https://www.openkm.com/blog/openkm-community-edition-v70-a-new-stage-for-open-source-document-management.html (2026-03-03)
- **Current binary:** Docker image `openkm/openkm-ce:7.0.4`, pushed 2026-09-21. **[VF]** https://hub.docker.com/r/openkm/openkm-ce
- **Edition differences (official comparison table):**
  - Versioning and OCR: **Community yes**.
  - Activity log (audit): **Community no**, Professional/Cloud yes.
  - Multi-tenant: Community no.
  - E-signature, records management and document expiration: Professional/Cloud only.
  - Workflow: the Community cell contradicts itself (alt text "Yes", icon "No"). **[OQ]**
  - **[VF]** https://www.openkm.com/en/comparison-of-versions.html
- **Pricing:** none published; contact sales. **[VF]**

### 1.4 LogicalDOC (Community vs commercial)
- **License and release:** Community Edition is LGPL-3.0. **[VF]** https://github.com/logicaldoc/community
  - The latest CE binary is 9.2.0 (SourceForge, 2025-08-11).
  - The repo is active, with commits on 2026-10-16 and tags 9.2.3 and 9.3, but no newer CE binaries were found.
  - **[VF]** https://sourceforge.net/projects/logicaldoc/files/distribution/
- **Edition matrix:**
  - Versioning, users/groups, folder security and the events log are in **all editions including CE**.
  - **OCR is not in CE** (Business and above).
  - **Workflow is Enterprise/Cloud only**.
  - Retention and digital signature are Enterprise/Cloud only.
  - Multi-tenant is not in CE or Business. The Enterprise cell is blank. **[OQ]**
  - **[VF]** https://www.logicaldoc.com/features/comparison-matrix
- **Pricing:** "Get a Quote" for all paid editions. **[VF]**

### 1.5 Alfresco Community Edition (Hyland)
- **License and release:** LGPL-3.0. The latest GA community image is 26.2.0 (Docker Hub, 2026-07-21). **[VF]** https://hub.docker.com/r/alfresco/alfresco-content-repository-community
- **Permissions:** native site roles (Manager, Collaborator, Contributor, Consumer) plus ACLs. **[VF]** https://github.com/Alfresco/docs-alfresco/blob/master/content-services/community/using/permissions.md
- **Versioning:** native, with revert. **[VF]** https://github.com/Alfresco/docs-alfresco/blob/master/content-services/community/using/content/manage.md
- **Approval:** native "Review and Approve" workflows (single reviewer, multiple reviewers, group, pooled). **[VF]** https://github.com/Alfresco/docs-alfresco/blob/master/content-services/community/using/tasks.md
- **Audit:** native but **disabled by default**; enable it with `audit.alfresco-access.enabled=true`. **[VF]** https://github.com/Alfresco/docs-alfresco/blob/master/content-services/community/admin/audit.md
- **OCR and multi-tenancy:** neither is documented for Community. **[OQ]**
- **Hosting:** the heaviest option. Hyland's README asks for at least 13 GB RAM (16 GB recommended) and says Compose is "not ready for production". **[VF]** https://github.com/Alfresco/acs-deployment
- **Enterprise pricing:** contact sales. **[OQ]**

### 1.6 Nextcloud (de-facto DMS)
- **License and release:** AGPL-3.0. v35.0.2, v34.0.5 and v33.0.10 were all released 2026-10-15. **[VF]** https://github.com/nextcloud/server/releases
- **Versioning:** native (files_versions, with automatic expiry). **[VF]** https://github.com/nextcloud/documentation/blob/master/admin_manual/configuration_files/file_versioning.rst
- **Audit:** the bundled `admin_audit` app, which is optional. **[VF]** https://github.com/nextcloud/documentation/blob/master/admin_manual/configuration_server/logging_configuration.rst
- **Approval:** the official **Approval** app (plugin, AGPL-3.0, v3.3.2, 2026-09-17). Admins define who can request and who can approve. **[VF]** https://github.com/nextcloud/approval
- **Retention:** the official **Files retention** app (plugin). It deletes files after a set period; it is not records-management holds. **[VF]** https://github.com/nextcloud/files_retention
- **OCR:** not native. The **Workflow OCR** app is a *third-party* plugin (OCRmyPDF + Tesseract, so `fil` is possible). **[VF]** https://github.com/R0Wi-DEV/workflow_ocr
- **Hosting:** PHP plus MariaDB/MySQL/PostgreSQL. 512 MB RAM per process is recommended. **[VF]** https://github.com/nextcloud/documentation/blob/master/admin_manual/installation/system_requirements.rst
- **Enterprise pricing:** EUR per user **per year**, excluding VAT, **minimum 100 users**. Standard is €71.29, Premium €104.99 and Ultimate €204.75 (100-user prices). This is irrelevant for 5–30-seat SMEs. **[VF]** https://nextcloud.com/pricing/

### 1.7 Seafile
- **Licenses:** seafile-server is AGPLv3 with an OpenSSL exception, and the seahub web UI is Apache-2.0. Pro is proprietary. **[VF]** https://github.com/haiwen/seafile-server/blob/master/LICENSE.txt and https://github.com/haiwen/seahub/blob/master/LICENSE.txt
- **Release:** CE v14.0.9 (2026-09-22) and Pro v14.0.9 (2026-10-12). **[VF]** https://github.com/haiwen/seahub/tags
- **Audit:** **Pro only**. **[VF]** https://github.com/haiwen/seafile-admin-docs/blob/master/manual/administration/auditing.md
- **Multi-tenancy:** documented. **[VF]** https://github.com/haiwen/seafile-admin-docs/blob/master/manual/config/multi_tenancy.md (which edition has it is **[OQ]**)
- **Approval workflow:** none found. **[OQ]**
- **OCR:** image OCR only, through the Seafile AI extension, which needs an LLM. It is not a document OCR pipeline. **[VF]** https://github.com/haiwen/seafile-admin-docs/blob/master/manual/extension/seafile-ai.md
- **Versioning:** file history is generally known to exist but was not re-checked. **[OQ]**
- **Hosting:** Docker and MySQL. CE needs at least 2 GB RAM; Pro needs at least 4 GB plus a search engine. **[VF]** https://github.com/haiwen/seafile-admin-docs/blob/master/manual/setup/setup_pro_by_docker.md
- **Pro pricing:** **3 users free**. Up to 9 users is USD 100 total per year. 10–249 users is USD 48 per user per year. **[VF]** https://www.seafile.com/en/pricing/

**Open-source takeaway.** Only Alfresco CE and Mayan EDMS ship *native* review/approval workflows in a free edition. Both are operationally heavy, and Mayan has single-maintainer risk. Paperless-ngx is the healthiest project but has no approvals. OpenKM CE is no longer open source. LogicalDOC CE lacks OCR and workflow. **[VF]**, based on the sources above.

---

## 2. Commercial DMS/ECM and vertical tools

Unless marked "PHP", "€", "£" or "AUD", **$ means USD**, and pages that show "$" without a currency code are flagged.

### 2.1 General DMS/ECM

**DocuWare**
- **Target:** SMB to mid-market, use-case led (AP, HR, contracts). **[VC]**
- **Price:** no public price card. DocuWare's own FAQ says Cloud "typically costs between $30 and $125+ per user per month", annual subscription. Add-ons (IDP, e-signature, connectors) are priced separately. On-prem is a perpetual license plus about 20% a year. **[VF]** (vendor-published range) https://start.docuware.com/faq/docuware-pricing
- **Doc control:**
  - Version control: native.
  - Workflow Manager (approvals): included in Cloud, but needs extra licenses on-prem.
  - Audit logs: native; *tamper-proof* audit trails are optional and activated through Support.
  - Full-text OCR indexing: included.
  - **[VF]** https://start.docuware.com/en-gb/cloud-on-premises-hybrid-deployment ; https://start.docuware.com/blog/product-news/tamper-proof-audit-trails-enhanced-integrity-protection-for-your-documents
- **PH presence:** 1doc.ph (Makati) calls itself "your official DocuWare Gold Reseller". **[VC]** It is not confirmed in DocuWare's directory, which could not be loaded. https://1doc.ph/pricing/
  - **PHP prices per month, per plan (not per user):** Silver ₱33,970 (5 named users), Gold ₱76,970 (15), Platinum ₱174,970 (40). Annual billing "save 17%". On-prem "from PHP 3,000,000". **[VF]** (as published by the reseller)
  - Paid support plans are **separate**: ₱11,600 to ₱38,280 per month. **[VF]** https://1doc.ph/pricing/
- **Complaints:** cost, and a steep learning curve for workflows and indexing. **[AN]** https://capterra.com/p/220466/DocuWare/reviews/

**M-Files**
- **Price:** Essentials is **€65 per seat per month**, "typically billed annually". Enterprise is a custom quote. **[VF]** https://www.m-files.com/pricing/
- **Doc control:**
  - Version control, graphical workflows and an audit capability are in Essentials.
  - The *extended* audit log and custom versioning schemes are Enterprise only.
  - OCR is an optional module. **[VC]**
  - **[VF]** https://www.m-files.com/pricing/ ; https://www.m-files.com/supplemental/document-management-solution/
- **PH presence:** none found. **[OQ]** (partner directory not checked)
- **Complaints:** search usability, and losing on-prem vault access after the subscription lapsed. **[AN]** https://www.capterra.com/p/122215/M-Files/reviews

**Laserfiche**
- **Target:** mid-market, enterprise and government.
- **Price (USD, billed annually):**
  - Professional: $73 per user per month, minimum 5 users.
  - Business: $93 per user per month, minimum 25 users.
  - Business+: $42 per user per month, minimum 500 users.
  - Starter and self-hosted: contact sales.
  - Implementation is excluded.
  - **[VF]** https://www.laserfiche.com/pricing/ (fetched by a sub-agent; my re-check got a 403)
- **Doc control:**
  - Versioning, a no-code approval designer and a basic audit trail are in all plans.
  - Advanced Audit Trail and Records Management are **add-ons on Professional**.
  - Batch scanning with OCR is included. **[VF]** (same URL)
- **PH presence:** no Philippine solution provider found. The contact form lists the Philippines, and the nearest office is Hong Kong. **[OQ]** https://www.laserfiche.com/contact/

**Zoho WorkDrive**
- **Target:** SMB file collaboration rather than full ECM.
- **Price:** USD per user per month, minimum 3 users (annual / monthly).
  - Starter: $2.50 / $3.
  - Team: $4.50 / $6.
  - Business: $9 / $11.
  - The mapping of prices to plans is inferred from Zoho's pricing JSON.
  - **[VF]**/**[OQ]** mapping. https://www.zoho.com/workdrive/pricing.html ; https://www.zoho.com/sites/zweb/json/pricing/workdrive-pricing-val.json
  - **No PHP currency** in Zoho's price file (it does include SGD, MYR and IDR). **[VF]**
- **Doc control:**
  - Unlimited versions and OCR in search: all plans.
  - Custom workflows: Business only.
  - Admin audit trail: **Business only**.
  - **[VF]** https://www.zoho.com/workdrive/pricing.html
- **Data centres:** US, EU, IN, AU, CN, JP, AE, CA and SA. There is no PH or SG data centre. **[VF]** https://www.zoho.com/workdrive/developer/docs/api/v1/getting-started-multi-dc-support.html
- **PH presence:** press reports describe Zoho expanding in the Philippines. **[VF-2]** https://gmanetwork.com/news/money/companies/921148/zoho-expanding-presence-in-philippines/story

### 2.2 Legal vertical

**Clio Manage**
- **Target:** solo to mid-size law firms.
- **Price:** the plans are now Starter, Core, Signature and Elite.
  - Only Starter is priced: "Starting at $49/user". The billing term was not visible.
  - The other plans say "Get pricing".
  - **[VF]** Wayback snapshot of 2026-10-05 (live page returned 403): https://web.archive.org/web/20261005012636/https://www.clio.com/pricing/
- **Doc control:**
  - Version history: native. https://help.clio.com/hc/en-us/articles/14983859054619-Search-and-Download-Documents
  - **Native OCR "not supported"** (older help article); OCR comes through integrations. **[VF]**/**[OQ]** whether this is still current. https://support.clio.com/hc/en-us/articles/360006217913-Does-Clio-Support-Automatic-OCR-
  - No native document-approval workflow found. **[OQ]**
- **Data regions:** US, CA and AU (plus IE/UK in the selector). There is no Asia region. **[VF]** https://help.clio.com/hc/en-au/articles/53337803186715-Understand-Clio-Work-Data-Residency
- **PH presence:** none found. **[OQ]**

**NetDocuments**
- **Price:** contact sales. **[VF]** https://www.netdocuments.com/
  - Third-party estimate: about $50–65 per user per month. **[AN]** https://lexworkplace.com/netdocuments-pricing/
- **Doc control:** native check-out/in, versions with an "Official Version", an Approve option, and a per-document history (audit). **[VF]** https://support.netdocuments.com/s/article/205217500
  - OCR exists, but whether it is an add-on is **[OQ]**.
- **Regions:** older releases cite Azure storage in Singapore and Hong Kong. **[VC]** https://www.netdocuments.com/press-releases/ndflexstore
- **PH presence:** none found. There is an Asia page. https://www.netdocuments.com/who-we-serve/asia
- **Complaints:** slow preview and inconsistent search. **[AN]** https://www.g2.com/compare/netdocuments-vs-imanage-drive

**iManage Work**
- **Target:** large and global law firms ("81% of AmLaw 200"). **[VC]** https://imanage.com/
- **Price:** contact sales. **[VF]**
- **Doc control:**
  - Versions, check-out/in and a Timeline activity log: native. **[VF]** https://docs.imanage.com/work-help/10.10.1/en/Accessing_document_versions.html
  - Workflow through Tracker.
- **Regions:** "10 data regions" including Singapore. **[VC]** https://imanage.com/resources/resource-center/blog/the-imanage-cloud-delivering-consistent-productivity-everywhere/
- **PH presence:** none found.

**PracticePanther**
- **Price:** USD per user per month (annual / monthly).
  - Solo: $49 / $59.
  - Essential: $69 / $79.
  - Business: $89 / $99.
  - Business Pro: $114 / $124.
  - **[VF]** https://www.practicepanther.com/pricing/
- **Doc control:**
  - Version history: native.
  - E-signature: from the Business plan up.
  - **No OCR, approval workflow or audit trail advertised.** It syncs with Drive, OneDrive, Dropbox and Box.
  - **[VF]** https://practicepanther.com/document-management
- **PH presence:** none found.

**PH-local legal software found** (all **[VC]**, none verified for document-control features):
- LawSys by Intelliseven, Quezon City. https://lawsys.intelliseven.com.ph/
- MyLegalWhiz. https://www.mylegalwhiz.com/
- Beveron Smart Lawyer Office. https://www.beveron.com/case-management-software/case-management-software-Philippine.html

### 2.3 Accounting / bookkeeping vertical

**SmartVault**
- **Price:** "$" per user per month with no currency code shown, minimum 2 users.
  - Accounting Pro: $65 annual / $85 monthly.
  - Accounting Unlimited: $85 / $110.
  - **[VF]** https://www.smartvault.com/pricing/
- **Doc control:**
  - Unlimited version history, activity log (audit), client portal and document requests: all plans.
  - **E-signature is a paid add-on except on Unlimited.**
  - OCR is not listed.
  - **[VF]** (same URL)
- **PH presence:** none found.

**Karbon**
- **Target:** accounting firms; it is practice/workflow management first.
- **Price:** USD per user per month (annual / monthly).
  - Team: $59 / $79.
  - Business: $89 / $99.
  - Enterprise: custom.
  - **No PHP** among the currency options.
  - **[VF]** https://karbonhq.com/pricing/
- **Doc control:**
  - Document management, client portal and client requests.
  - Native e-signature with an audit trail. **[VF]** https://help.karbonhq.com/en/articles/10239224-esignature-overview-in-karbon
  - Version control is not stated. **[OQ]**
- **PH presence:**
  - Manila and Cebu are listed as hiring locations. **[VF]** https://job-boards.greenhouse.io/karbon/jobs/5675724004
  - Listed on Xero's PH app store. **[VF]** https://apps.xero.com/ph/app/karbon
  - Note that Karbon courts PH *offshore staff serving foreign firms*, not PH-domestic firms. **[OQ]**

**TaxDome**
- **Price (US/CA):** per user **per year**, billed upfront. Essentials $800, Pro $1,000, Business $1,200. **[VF]** https://help.taxdome.com/article/1800-taxdome-pricing-plans-us-ca
  - Non-US countries get a single plan priced per country (EU, AU/NZ, UK, JP, BR). **No PH pricing.** **[VF]** https://fr.help.taxdome.com/article/926-eu-br-taxdome-pricing-faq
- **Doc control:**
  - Per-document audit trail. **[VF]** https://taxdome.com/document/docs-basic-view-history-audit-trail
  - Native e-signature; KBA is US-only. **[VF]** https://help.taxdome.com/article/287-knowledge-based-authentication-kba
  - OCR is **[OQ]**.
- **Complaints:** learning curve and no phone support. **[AN]** https://www.capterra.com/p/186749/TaxDome/reviews?page=4

**Suralink**
- **Target:** audit and tax engagement teams.
- **Price:** quote-based. **[VF]** https://www.suralink.com/pricing
- **Features:**
  - CORE includes request lists, OCR and archiving.
  - E-signature and secure file sharing are **paid add-on modules**.
  - **[VF]** (same URL)
- **PH presence:** none found.

**PH-relevant note.** None of the four accounting tools shows PHP pricing, Philippine tax forms or any BIR workflow. TaxDome's localisation covers the US, CA, EU, UK, AU/NZ, JP and BR. **[VF]** for absence on the pages checked, **[OQ]** for whether PH firms use them anyway.

### 2.4 Construction vertical

**Procore**
- **Price:** "upfront annual fee by product and based upon your Annual Construction Volume", unlimited users, no public prices. **[VF]** https://www.procore.com/pricing
  - Third-party estimate: about $4.5k to $60k+ per year. **[AN]** https://costbench.com/software/construction-management/procore
- **Doc control:**
  - Approval workflows in Document Management update revision status. **[VF]** https://v2.support.procore.com/en-au/process-guides/document-management-getting-started-guide/document-approvals-with-workflows-overview
  - Transmittals with change history. **[VF]** https://support.procore.com/products/online/user-guide/project-level/transmittals/tutorials/create-a-transmittal
  - Drawing OCR (number, title and revision) supports English, French, German and Spanish only. **[VF]** https://v2.support.procore.com/en-au/product-manuals/drawings-project/tutorials/upload-drawings
- **Data:** a Singapore storage zone is available. **[VF]** https://v2.support.procore.com/en-au/faq-where-and-how-does-procore-store-customer-information
- **PH presence:**
  - Customer Makati Development Corp. (Ayala). **[VC]** https://www.procore.com/en-sg/casestudies/makati-development-corporation
  - A 2023 MOU with the Philippine Constructors Association. **[VF-2]** https://www.cimdata.com/en/industry-summary-articles/item/21270-procore-launches-digital-transformation-program-with-the-philippine-constructors-association-to-upskill-local-construction-workforce
- **Complaints:** "expensive" and "steep learning curve". **[AN]** https://www.g2.com/products/procore

**Autodesk Construction Cloud / Docs (rebranded "Autodesk Forma", March 2026)**
- **Rebrand:** Docs is now "Forma Data Management". **[VF]** https://construction.autodesk.com/go/acc-to-forma-info/
- **Price:** "Get a Quote". **[VF]** https://construction.autodesk.com/pricing/
  - UK G-Cloud listing: £460 per licence per year. **[VF]** (government listing) https://assets.applytosupply.digitalmarketplace.service.gov.uk/g-cloud-15/documents/720416/174842602650469-pricing-document-2026-01-28-1750.odt
- **Doc control:**
  - Review and approval workflows. **[VF]** https://www.autodesk.com/learn/ondemand/module/review-and-approve-documents-in-the-cloud-using-autodesk-docs
  - Transmittals. **[VF-2]**
- **Data regions:** US, EU and AU; no Asia region found. **[VF]** https://www.autodesk.com/blogs/construction/your-questions-answered-autodesk-introduces-regional-data-storage-option-in-australia/
- **PH presence:**
  - Partner CIM Technologies. **[VF]** https://www.autodesk.com/support/partners/cim-technologies-inc/1455
  - First Balfour adoption reported in 2025. **[VF-2]** https://context.ph/2025/09/28/first-balfour-pioneers-ai-in-ph-construction/

**Oracle Aconex**
- **Target:** large and infrastructure projects.
- **Price:** no public price; per-project or enterprise licensing. **[AN]** (third-party) https://redresscompliance.com/oracle-aconex-per-project-vs-enterprise-licensing
- **Doc control:**
  - "Unalterable" audit trail. **[VC]** https://www.oracle.com/news/announcement/new-oracle-aconex-capabilities-improve-project-transparency-and-control-2026-04-13/
  - Native review/approval workflows and transmittals. **[VF]** https://help.aconex.com/high-compliance-environments/what-is-aconex/
- **Data:** an ASIA1 (Singapore) instance. **[VF]** https://help.aconex.com/faqs/what-are-the-links-for-each-instance/
- **PH presence:** the Malolos–Clark Railway (Acciona–EEI JV). **[VC]** https://www.oracle.com/customers/acciona/

**Fieldwire (Hilti)**
- **Price:** USD per user per month, annual billing.
  - Basic: $0 (5 users, 3 projects).
  - Pro: $39.
  - Business: $64.
  - Business Plus: $89.
  - **RFIs and submittals are Business Plus only.**
  - **[VF]** https://www.fieldwire.com/pricing/
- **OCR:** sheet OCR (auto-naming and versioning of sheets). **[VF]** https://help.fieldwire.com/hc/en-us/articles/360040931972
- **Audit trail:** not listed. **[OQ]**
- **PH presence:** none found.

**Construction takeaway.** For the *top* of the PH construction market, the gap is already served (MDC, First Balfour, Malolos–Clark). The tools are priced for large contractors (ACV-based or quote-only). Whether small and mid PH contractors use any of them is an **[OQ]**.

---

## 3. The real incumbent: general cloud storage

### 3.1 Pricing (PHP)
- **Google Workspace:** the official PH-locale pricing page shows **USD only**, per user per month on a 1-year commitment. Business Starter $7, Standard $14, Plus $22; Enterprise is contact sales. **[VF]** https://workspace.google.com/intl/en_ph/pricing.html
  - A PH reseller (Z.com) lists Business Starter at ₱3,360 per user per year (about ₱280 a month). **[VC]** https://web.z.com/ph/google-workspace/
  - Another reseller blog estimates ₱509–612 a month including VAT, so reseller figures conflict. **[AN]** https://leadsmonky.com/best-google-workspace-reseller-in-philippines/
- **Microsoft 365:** microsoft.com/en-ph timed out or blocked fetches, so **official PHP pricing was not verified**. **[OQ]**
  - Secondary sources report US list prices from 2026-07-01 of Basic $7, Standard $14 and Premium $22 per user per month, annual. **[AN]** https://www.makios.com/articles/microsoft-365-pricing-changes-business-owners
- **Dropbox:** the plans page shows "$" without a country. Standard $15 and Advanced $24 per user per month; Enterprise by quote. PHP not shown. **[VF]** https://www.dropbox.com/plans

### 3.2 Native document-control features by tier

| Capability | Google Workspace | Microsoft 365 (SharePoint/OneDrive) | Dropbox |
|---|---|---|---|
| Version history | Uploaded files: versions may be purged after 30 days or 100 newer versions unless "Keep forever" **[VF]** https://support.google.com/drive/answer/2409045 | Configurable by count/age; major/minor versions, **require check-out**, **content approval** in libraries (all SharePoint plans) **[VF]** https://learn.microsoft.com/en-us/sharepoint/document-library-version-history-limits ; https://support.microsoft.com/office/enable-and-configure-versioning-for-a-list-or-library-1555d642-23ee-446a-990a-bcab618c7a37 | 180 days (Standard), 1 year (Advanced) **[VF]** https://www.dropbox.com/plans |
| Approvals | Drive approvals in Business Standard/Plus and Enterprise; **not in Business Starter** **[VF]** https://knowledge.workspace.google.com/admin/drive/manage-approvals | Teams Approvals app (needs a Power Automate/M365 licence; stored in Dataverse, **no backup**) **[VF]** https://learn.microsoft.com/en-us/microsoftteams/approval-admin | Not listed on plans page **[OQ]** |
| Retention / legal hold | Vault only in Business Plus and Enterprise (not Starter/Standard) **[VF]** https://knowledge.workspace.google.com/vault/getting-started/vault-overview | Business Premium: manual retention labels, basic retention, Standard eDiscovery **[VF]** https://www.microsoft.com/licensing/guidance/Microsoft-Purview ; Basic/Standard **[OQ]** | Not verified **[OQ]** |
| Audit log | Drive audit/investigation listed for Enterprise/Frontline/Education editions; whether Business tiers are covered is **[OQ]** https://knowledge.workspace.google.com/admin/reports/drive-log-events | Purview Audit Standard: 180-day retention; Premium: 1 year (10 years with add-on) **[VF]** https://learn.microsoft.com/en-us/purview/audit-solutions-overview | File-event audit log on **Advanced** only **[VF]** https://www.dropbox.com/plans ; Standard only logs sign-in/security events **[AN]** https://www.dropboxforum.com/t5/Plans-and-Subscriptions/Dropbox-Standard-Plan-Activity-Log/m-p/267479 |
| OCR | "Open with Google Docs" OCR, ≤2 MB files, layout often lost **[VF]** https://support.google.com/drive/answer/176692 | Pay-as-you-go via Azure: OCR $0.001/page, e-signature $2/request **[VF]** https://learn.microsoft.com/en-us/microsoft-365/syntex/syntex-pay-as-you-go-services | Not verified **[OQ]** |
| Data residency | US or Europe only (no Asia option) **[VF]** https://knowledge.workspace.google.com/admin/compliance/data-covered-by-data-regions | APAC geography includes SG/HK/JP/MY/AU/KR; **no PH** **[VF]** https://learn.microsoft.com/en-us/microsoft-365/enterprise/m365-dr-overview | US/AU/JP only **[VF]** https://help.dropbox.com/security/data-transfers-asia-pacific |

### 3.3 What they lack for formal document control
This is synthesis from §3.2, so treat it as **[OQ]** until tested with users:
- **No controlled-document conventions out of the box:** no document numbering, no revision schemes (Rev A/B/0) and no review or expiry dates.
- **No read-and-acknowledge step.**
- **No formal transmittal or submittal registers** for construction.
- **No matter- or client-centric structure** with conflict checks for law firms.
- **Audit and retention are gated to higher tiers:** Business Plus, Business Premium/E5 or Dropbox Advanced.
- **No PH data region.**
- **Microsoft 365's raw primitives are strong:** check-out, content approval, major/minor versions. The gap is likely *configuration and usability for SMEs*, not missing capability. **[OQ]**

---

## 4. Philippine-local DMS/ECM vendors and resellers

The local supply found is **thin and enterprise- or government-oriented**. There is little to no public pricing apart from 1doc.ph, and I found **no SME-focused, self-serve SaaS DMS from a Filipino startup**. "Absence" here means not found in a first-pass search, so it is an **[OQ]**.

| Vendor | What they offer | Evidence |
|---|---|---|
| 1doc.ph (1wan.ph), Makati | DocuWare Gold Reseller (self-claimed); PHP plan pricing (see §2.1); claims "BIR-compliant archiving" without citing a regulation | **[VC]** https://1doc.ph/ ; https://1doc.ph/pricing/ |
| Paperless Trail Inc. — Archive One | Local DMS (SaaS, on-prem, private Azure); RBAC, audit trail, full-text search, scanning services; targets HR, LGUs, hospitals; no pricing | **[VC]** https://archive-one.net/document-management-software-philippines-archive-one |
| Infobuilder Technologies, Pasig | OpenText Documentum/Exstream/InfoArchive integrator; banks, insurers, legal, real estate | **[VC]** https://infobuilder.ph/ |
| U-BIX Corporation | Own DMS "DocuVu" plus scanning and indexing; named enterprise clients | **[VC]** https://ubix.com.ph/document-management/ |
| Copylandia | DOKMEE capture/ECM distributor | **[VC]** https://copylandia.com/dokmee/ |
| Edamame Inc. | Kintone partner; blog claims Kintone from ₱1,050 per user per month and DocuWare about ₱2,500 per user per month (biased source) | **[VC]**/**[OQ]** https://edamame-jp.com/kintone-philippines/en/blog/document-management-system-philippines/ |
| CIM Technologies | Autodesk partner (construction CDE) | **[VF]** https://www.autodesk.com/support/partners/cim-technologies-inc/1455 |
| LawSys, MyLegalWhiz, Beveron | PH legal case/practice management (not DMS-first) | **[VC]** (§2.2) |
| Laserfiche / M-Files PH partners | None found | **[OQ]** |

---

## 5. Philippine-specific gaps

### 5.1 Language (Filipino OCR)
- **Filipino OCR is *not* a gap at the engine level.** **[VF]**
  - **Tesseract:** ships `fil.traineddata` (tessdata, best and fast) and a legacy `tgl`. https://github.com/tesseract-ocr/tessdata ; https://github.com/tesseract-ocr/tessdoc/blob/main/Data-Files-in-different-versions.md
  - **Google Cloud Vision:** supports Filipino (`fil`) and Tagalog (`tl`). Cebuano is experimental. https://docs.cloud.google.com/vision/docs/languages
  - **Azure Document Intelligence Read/Layout:** reads printed Filipino, Cebuano, Hiligaynon, Iloko, Pampanga, Pangasinan and Bikol. **Handwritten Filipino is not supported.** https://learn.microsoft.com/en-us/azure/ai-services/document-intelligence/language-support/ocr
- **Possible narrow gaps:**
  - Products that *expose* OCR language choice, or tune for mixed English/Filipino scanned court and BIR forms.
  - Handwriting.
  - Procore drawing OCR, which is limited to EN/FR/DE/ES.
  - Whether any of these matter to buyers is **[OQ]**. Most PH legal, tax and construction documents are in English.

### 5.2 Data residency
- **The Data Privacy Act (RA 10173) does not impose general data localisation.** Cross-border transfer is allowed, and the PH controller stays accountable. **[VF-2]** https://resourcehub.bakermckenzie.com/en/resources/global-data-and-cyber-handbook/asia-pacific/philippines/topics/data-localization-and-regulation-of-non-personal-data
- **NPC Advisory 2024-01** provides model contractual clauses for cross-border transfers. **[VF]** https://privacy.gov.ph/wp-content/uploads/2024/06/Published-NPC-Advisory-No.-2024-01-Contractual-Clauses-for-Cross-Border-Transfers_30May24.pdf
- **No AWS, Azure or Google Cloud region exists in the Philippines.** AWS has a Manila Local Zone, and a proposed ~$5B PH region was pitched in July 2026 but not announced. **[VF-2]** https://newsbytes.ph/2026/07/08/marcos-jr-meets-aws-execs-over-proposed-5-b-cloud-investment-in-ph/
- **Implication:** "PH data residency" is a *preference/trust* argument, not a legal requirement for private SMEs. It would need local hosting (local DC or on-prem), which competitors such as Archive One and 1doc.ph on-prem already offer. Whether buyers will pay for it is **[OQ]**.

### 5.3 PHP pricing
- **No global vendor checked publishes official PHP pricing.** That covers Google (USD on the PH page), Zoho (no PHP), Karbon (no PHP), TaxDome (no PH plan) and Dropbox ("$"). Microsoft was not verifiable. **[VF]**, per the sources above.
- **PHP pricing exists only through resellers** (1doc.ph, Z.com). **[VF]**
- **Price anchor:** 1doc.ph's DocuWare entry plan is ₱33,970 a month for 5 users, about ₱6,800 per user per month before separate support. That is far above Google Workspace reseller prices (about ₱280–612 per user per month). **[VF]** for the published numbers; the comparison is arithmetic.

### 5.4 BIR and local compliance
- **Retention is now 5 years, not 10.** The Ease of Paying Taxes Act (RA 11976) amended NIRC Sec. 235, and RR 7-2024 implements it. Books and records are kept 5 years from the day after the filing deadline (longer if a protest or refund is pending). **[VF-2]** https://www.grantthornton.com.ph/insights/articles-and-updates1/tax-notes/eopt-is-here-updates-on-the-preservation-of-book-of-accounts-and-changes-in-taxpayer-registration/ ; https://kpmg.com/ph/en/insights/2024/01/special-intax-january-2024-issue-1-volume-3.html
  - The old 10-year rule (RR 17-2013 as amended by RR 5-2014) is superseded. Some 2025–26 blogs still cite 10 years. **[VF]** https://dev.lawyerly.ph/laws/78650
- **Electronic-storage standard:** RR 5-2014 Sec. 2-A required accurate imaging, indexing, integrity and tamper detection, and legible printable reproductions. Whether it survives under RR 7-2024 is **[OQ]**. https://bdplaw.com.ph/news/amending-revenue-regulations-no-17-2013-dealing-%E2%80%9Cpreservation-books-accounts-and-other
- **E-invoicing (EIS):** the first-wave deadline stays **31 Dec 2026** (RR 11-2025 as amended by RR 26-2025, with RMC 98-2026). **[VF-2]** https://www.grantthornton.com.ph/technical-alerts/tax-alert/2026/bir-issues-guidelines-on-electronic-invoicing-retains-31-dec-2026-deadline/
  - This is an *invoicing-system* obligation, not a DMS feature. Its relevance to the DMS is **[OQ]**.
- **"BIR-compliant archiving" claims:** made by 1doc.ph without citing a rule. **[VC]** No global product advertises BIR-specific features. **[VF]** (absence on pages checked)
- **Law firms:**
  - Rules on Electronic Evidence (A.M. 01-7-01-SC): authenticity and integrity of electronic documents must be proven, which is where audit trails and hashes help. **[VF]** https://lawphil.net/courts/supreme/am/am_01-7-01_2001.html
  - CPRA (A.M. 22-09-01-SC) Canon III Sec. 56: turnover of client files on termination. **[VF-2]** https://batasnatin.com/laws/cpra-c3-section-56-accounting-and-turn-over-upon-termination
  - Rules on Electronic Notarization (A.M. 24-10-14-SC, effective 2025-03-09). **[VF-2]** https://accralaw.com/2025/03/26/from-paper-to-pdf-the-digital-shift-in-philippine-notarization/
  - **No fixed law-firm file-retention period was found.** **[OQ]**

### 5.5 Local support
- **Local support exists only through resellers and integrators:** 1doc.ph, Infobuilder, U-BIX, CIM Technologies, Google Workspace resellers. **[VC]**
- **Some global vendors have regional reach:** Procore (Singapore hub plus the PCA programme) and Karbon (PH staff). **[VF]**/**[VF-2]**
- Whether SMEs perceive a *support* gap is **[OQ]**; it needs interviews.

---

## (a) One-page comparison table

Prices: USD unless noted; "pu/m" means per user per month. "CS" means contact sales. PH presence: **L** = local reseller/partner found, **C** = PH customer or programme evidence, **–** = none found.

| Product | Segment | Deployment | Starting price (official) | Versioning | Approvals | Audit | OCR | PH presence |
|---|---|---|---|---|---|---|---|---|
| Paperless-ngx | Personal/small-office archive | Self-host | Free (GPL-3.0) | Native (v3+) | No | Native | Native (Tesseract; `fil` possible) | – |
| Mayan EDMS | General EDMS | Self-host | Free (GPL-2.0) | Native + check-out | Native (state-machine workflow) | Native events | Native (Tesseract) | – |
| OpenKM CE / Pro | General DMS | Self-host / cloud | CE free binary (not OSS since 7.0); Pro CS | Native | **[OQ]** CE | Pro only | Native | – |
| LogicalDOC CE / commercial | General DMS | Self-host / cloud | CE free (LGPL); paid CS | Native | Enterprise only | Events log (all) | Business+ only | – |
| Alfresco Community | ECM | Self-host (≥13 GB RAM) | Free (LGPL) | Native | Native review/approve | Native (off by default) | **[OQ]** | – |
| Nextcloud | File sync + collab | Self-host | Free (AGPL); Enterprise from €71.29/user/**yr**, min 100 users | Native | Official plugin | Optional bundled app | 3rd-party plugin | – |
| Seafile | File sync | Self-host | CE free; Pro free ≤3 users, $48/user/**yr** | **[OQ]** | None found | Pro only | Image only (AI ext.) | – |
| DocuWare | SMB/mid-market DMS | Cloud / on-prem | Vendor range $30–125+ pu/m; **PH reseller ₱33,970/mo for 5 users** | Native | Cloud native; on-prem licence | Native (tamper-proof optional) | Native | **L** (1doc.ph, self-claimed) |
| M-Files | Mid-market ECM | Cloud / on-prem | €65/seat/m (annual) | Native | Native | Native; extended = Enterprise | Optional module | – |
| Laserfiche | Mid/enterprise/government | Cloud / self-host | $73 pu/m, annual, min 5 | Native | Native | Basic native; advanced add-on | Native | – (HK office) |
| Zoho WorkDrive | SMB file collaboration | Cloud | $2.50 pu/m annual, min 3 | Native | Custom = Business | Business only | Native (search) | – (no PHP, no Asian DC except JP/IN/AU) |
| Clio Manage | Small/mid law | Cloud | From $49/user (term **[OQ]**) | Native | None found | **[OQ]** | Not native | – |
| NetDocuments | Law (all sizes) | Cloud | CS | Native + check-out | "Approve" option | Native | **[OQ]** add-on | – |
| iManage Work | Large law | Cloud / on-prem | CS | Native + check-out | Tracker workflows | Native timeline | **[OQ]** | – (SG region) |
| PracticePanther | Small law | Cloud | $49 pu/m annual | Native | None | Not advertised | None | – |
| SmartVault | Accounting firms | Cloud | $65 pu/m annual, min 2 | Native, unlimited | Doc requests; e-sign add-on | Native | Not listed | – |
| Karbon | Accounting practice mgmt | Cloud | $59 pu/m annual | **[OQ]** | E-sign approvals | E-sign audit | **[OQ]** | C (PH staff, not PH clients) |
| TaxDome | Tax/accounting firms | Cloud | $800/user/**yr** (US) | **[OQ]** | E-sign | Native per-doc | **[OQ]** | – |
| Suralink | Audit/tax engagements | Cloud | CS | Not listed | Request lists; e-sign add-on | AI audit trail (module) | Native | – |
| Procore | GC/owners (mid+) | Cloud | CS (by annual construction volume) | Native drawing revisions | Native workflows, transmittals | Native | Drawings (EN/FR/DE/ES) | **C** (MDC, PCA MOU) |
| Autodesk Docs (Forma) | AEC | Cloud | CS (UK G-Cloud £460/licence/yr) | Native | Native review/approve | **[OQ]** | **[OQ]** | **L/C** (CIM Tech, First Balfour) |
| Oracle Aconex | Large infrastructure | Cloud | CS | Native | Native, transmittals | Native "unalterable" | **[OQ]** | **C** (Malolos–Clark) |
| Fieldwire | Field teams/subs | Cloud | Free tier; Pro $39 pu/m annual | Native sheets | RFIs/submittals = Business Plus | **[OQ]** | Sheet OCR | – |
| Google Workspace | Everyone | Cloud | $7 pu/m (USD on PH page); reseller ~₱280+ | Native (30d/100 for uploads) | Std+ only | Enterprise-tier (Business **[OQ]**) | Basic (≤2 MB) | **L** (resellers) |
| Microsoft 365 / SharePoint | Everyone | Cloud | PHP **[OQ]**; ~$7 Basic (secondary) | Strong (major/minor, check-out) | Content approval; Teams Approvals | Purview Std 180d; Premium 1y | Pay-as-you-go $0.001/page | **L** (resellers) **[OQ]** |
| Dropbox | Everyone | Cloud | $15 pu/m Standard | 180 days / 1 year | **[OQ]** | Advanced only | **[OQ]** | – |
| Archive One (PH) | PH govt/HR/hospitals | SaaS / on-prem | CS | **[OQ]** | **[OQ]** | Claimed | Claimed scanning | **Local** |

## (b) Problems already solved well by existing products
1. **Version history and file recovery** are commodity features. They are native in every cloud suite and in nearly every DMS listed. **[VF]**
2. **Formal check-out and content approval** are available in SharePoint in all M365 business plans, Alfresco CE and Mayan. **[VF]** The gap is setup and usability, not capability. **[OQ]**
3. **Engine-level Filipino OCR** is solved (Tesseract `fil`, Google Vision, Azure for printed text). **[VF]**
4. **Large-project construction document control** (transmittals, revisions, approvals, audit) is mature in Procore, Aconex and Autodesk, which already have PH reference customers and Asian data hosting. **[VF]**/**[VC]**
5. **Client portals, document requests and e-signature for accounting firms** are mature globally (SmartVault, TaxDome, Karbon), though not localised for PH. **[VF]**
6. **Enterprise ECM in PH** is served through integrators and resellers (DocuWare via 1doc.ph, OpenText via Infobuilder, U-BIX DocuVu). **[VC]**
7. **Cheap self-hosted archives** are available for technical SMEs (Paperless-ngx, Seafile Pro free for 3 users, Nextcloud). **[VF]**

## (c) Possible gaps — evidence strength

| Possible gap | Evidence | Strength |
|---|---|---|
| **Price gap:** nothing between "free Drive/SharePoint" (~₱300–1,000 pu/m) and the PH DocuWare reseller entry (₱33,970/mo for 5 users, ~₱6,800 pu/m) that offers document control | Published prices in §2.1 and §3.1 | **Medium.** The prices are verified, but we have not shown that SMEs *want* to pay for the middle. |
| **No PHP pricing or self-serve PH billing** from global SaaS | §5.3 | **Medium-strong** for the fact; **weak** that it blocks purchase (resellers bridge it). |
| **No PH-localised accounting-firm portal** (BIR forms, 5-year retention, PH tax calendar) | No vendor found; TaxDome localises for 7+ markets but not PH (§2.3) | **Medium** for absence; **unknown** demand. |
| **Small/mid PH contractors underserved** (Procore/Aconex priced for large GCs) | Pricing models in §2.4; PH references are all large firms | **Weak-medium.** Fieldwire's free and Pro tiers may already cover them. |
| **SMB legal DMS with approvals and audit, PH-priced** | Clio and PracticePanther lack approvals/OCR; NetDocuments and iManage are quote-only and enterprise; local legal tools are case-management-first | **Weak-medium.** PH law-firm demand is unvalidated. |
| **PH data residency** | No hyperscaler PH region; Google/Dropbox/Zoho have no SEA region; the DPA does not *require* localisation | **Weak** as a legal driver; **[OQ]** as a trust/sales driver. |
| **Configured "document control on top of M365/Google"** (numbering, revision codes, acknowledgments, transmittals) | Primitives exist but no SME-ready conventions (§3.3) | **Weak-medium.** This is an inference, so test it in interviews. |
| **Filipino OCR** | Engines support it | **Not a gap** for printed text; handwriting is **[OQ]**. |
| **Local support** | Only resellers found | **Weak.** Needs interviews. |

## (d) What evidence would invalidate building a new product
- **Satisfied incumbents:** interviews show target SMEs are satisfied with Drive or SharePoint folders plus naming conventions, or would not pay more than about their current Workspace/M365 per-seat cost for document control.
- **Low SharePoint switching cost:** M365 Business Standard/Premium users can get check-out, content approval and audit with a cheap one-off configuration from a local IT partner. The problem is then services, not product.
- **Mid-price tier already exists:** PH resellers (1doc.ph, Archive One, Kintone partners) quote SME-viable prices well below the published DocuWare plans, or a sub-₱1,000 pu/m PH offering already exists that we missed.
- **Construction is served:** small and mid PH contractors already use Fieldwire, Procore (PCA programme) or Autodesk Docs, or don't do formal transmittals at all.
- **Accounting is served:** PH accounting firms already use TaxDome, Karbon or SmartVault, or BIR-relevant workflows (EIS, 5-year retention) are handled by accounting/CAS software rather than a DMS.
- **No law-firm digitisation pull:** law-firm interviews show document volume or regulatory drivers (e-notarization, electronic evidence) are too weak to make firms digitise beyond case-management tools.
- **Residency is a non-factor:** buyers are indifferent to data residency and PHP billing, which removes two of the "local" differentiators.
- **Hyperscaler or global entry:** AWS announces a PH region and Google/Microsoft add PHP self-serve pricing, which narrows local differentiation.
- **Thin margins:** willingness-to-pay in interviews falls below the cost of support and hosting at SME scale.

## (e) Source list (all accessed 2026-10-18 unless noted)

**Open source**
- https://github.com/paperless-ngx/paperless-ngx/releases
- https://docs.paperless-ngx.com/usage/ (read from the docs source in the repo)
- https://docs.paperless-ngx.com/configuration/#PAPERLESS_OCR_LANGUAGES
- https://docs.paperless-ngx.com/setup/
- https://gitlab.com/mayan-edms/mayan-edms/-/blob/master/LICENSE
- https://pypi.org/project/mayan-edms/#history
- https://github.com/mayan-edms/Mayan-EDMS
- https://docs.mayan-edms.com/chapters/features.html
- https://gitlab.com/mayan-edms/mayan-edms/-/blob/master/docker/docker-compose.yml
- https://github.com/openkm/document-management-system
- https://www.openkm.com/blog/openkm-community-edition-v70-a-new-stage-for-open-source-document-management.html
- https://hub.docker.com/r/openkm/openkm-ce
- https://www.openkm.com/en/comparison-of-versions.html
- https://github.com/logicaldoc/community
- https://sourceforge.net/projects/logicaldoc/files/distribution/
- https://www.logicaldoc.com/features/comparison-matrix
- https://hub.docker.com/r/alfresco/alfresco-content-repository-community
- https://github.com/Alfresco/docs-alfresco (permissions.md, manage.md, tasks.md, audit.md)
- https://github.com/Alfresco/acs-deployment
- https://github.com/nextcloud/server/releases
- https://github.com/nextcloud/documentation (file_versioning.rst, logging_configuration.rst, system_requirements.rst)
- https://github.com/nextcloud/approval
- https://github.com/nextcloud/files_retention
- https://github.com/R0Wi-DEV/workflow_ocr
- https://nextcloud.com/pricing/
- https://github.com/haiwen/seafile-server/blob/master/LICENSE.txt
- https://github.com/haiwen/seahub
- https://github.com/haiwen/seafile-admin-docs (auditing.md, multi_tenancy.md, seafile-ai.md, setup_pro_by_docker.md)
- https://www.seafile.com/en/pricing/
- https://github.com/tesseract-ocr/tessdata
- https://github.com/tesseract-ocr/tessdoc/blob/main/Data-Files-in-different-versions.md
- https://packages.debian.org/trixie/tesseract-ocr-fil

**Commercial / general**
- https://start.docuware.com/faq/docuware-pricing
- https://start.docuware.com/en-gb/cloud-on-premises-hybrid-deployment
- https://start.docuware.com/blog/product-news/tamper-proof-audit-trails-enhanced-integrity-protection-for-your-documents
- https://1doc.ph/
- https://1doc.ph/pricing/
- https://www.m-files.com/pricing/
- https://www.m-files.com/supplemental/document-management-solution/
- https://www.laserfiche.com/pricing/
- https://www.laserfiche.com/contact/
- https://www.zoho.com/workdrive/pricing.html
- https://www.zoho.com/sites/zweb/json/pricing/workdrive-pricing-val.json
- https://www.zoho.com/workdrive/developer/docs/api/v1/getting-started-multi-dc-support.html
- https://gmanetwork.com/news/money/companies/921148/zoho-expanding-presence-in-philippines/story

**Legal**
- https://web.archive.org/web/20261005012636/https://www.clio.com/pricing/ (snapshot dated 2026-10-05)
- https://help.clio.com/hc/en-us/articles/14983859054619-Search-and-Download-Documents
- https://support.clio.com/hc/en-us/articles/360006217913-Does-Clio-Support-Automatic-OCR-
- https://help.clio.com/hc/en-au/articles/53337803186715-Understand-Clio-Work-Data-Residency
- https://www.netdocuments.com/
- https://support.netdocuments.com/s/article/205217500
- https://www.netdocuments.com/press-releases/ndflexstore
- https://www.netdocuments.com/who-we-serve/asia
- https://imanage.com/
- https://docs.imanage.com/work-help/10.10.1/en/Accessing_document_versions.html
- https://imanage.com/resources/resource-center/blog/the-imanage-cloud-delivering-consistent-productivity-everywhere/
- https://www.practicepanther.com/pricing/
- https://practicepanther.com/document-management
- https://lawsys.intelliseven.com.ph/
- https://www.mylegalwhiz.com/
- https://www.beveron.com/case-management-software/case-management-software-Philippine.html

**Accounting**
- https://www.smartvault.com/pricing/
- https://karbonhq.com/pricing/
- https://help.karbonhq.com/en/articles/10239224-esignature-overview-in-karbon
- https://job-boards.greenhouse.io/karbon/jobs/5675724004
- https://apps.xero.com/ph/app/karbon
- https://help.taxdome.com/article/1800-taxdome-pricing-plans-us-ca
- https://fr.help.taxdome.com/article/926-eu-br-taxdome-pricing-faq
- https://taxdome.com/document/docs-basic-view-history-audit-trail
- https://help.taxdome.com/article/287-knowledge-based-authentication-kba
- https://www.suralink.com/pricing

**Construction**
- https://www.procore.com/pricing
- https://v2.support.procore.com/en-au/process-guides/document-management-getting-started-guide/document-approvals-with-workflows-overview
- https://support.procore.com/products/online/user-guide/project-level/transmittals/tutorials/create-a-transmittal
- https://v2.support.procore.com/en-au/product-manuals/drawings-project/tutorials/upload-drawings
- https://v2.support.procore.com/en-au/faq-where-and-how-does-procore-store-customer-information
- https://www.procore.com/en-sg/casestudies/makati-development-corporation
- https://www.cimdata.com/en/industry-summary-articles/item/21270-procore-launches-digital-transformation-program-with-the-philippine-constructors-association-to-upskill-local-construction-workforce
- https://construction.autodesk.com/go/acc-to-forma-info/
- https://construction.autodesk.com/pricing/
- https://assets.applytosupply.digitalmarketplace.service.gov.uk/g-cloud-15/documents/720416/174842602650469-pricing-document-2026-01-28-1750.odt
- https://www.autodesk.com/learn/ondemand/module/review-and-approve-documents-in-the-cloud-using-autodesk-docs
- https://www.autodesk.com/blogs/construction/your-questions-answered-autodesk-introduces-regional-data-storage-option-in-australia/
- https://www.autodesk.com/support/partners/cim-technologies-inc/1455
- https://context.ph/2025/09/28/first-balfour-pioneers-ai-in-ph-construction/
- https://www.oracle.com/news/announcement/new-oracle-aconex-capabilities-improve-project-transparency-and-control-2026-04-13/
- https://help.aconex.com/high-compliance-environments/what-is-aconex/
- https://help.aconex.com/faqs/what-are-the-links-for-each-instance/
- https://www.oracle.com/customers/acciona/
- https://www.fieldwire.com/pricing/
- https://help.fieldwire.com/hc/en-us/articles/360040931972

**Cloud incumbents**
- https://workspace.google.com/intl/en_ph/pricing.html
- https://web.z.com/ph/google-workspace/
- https://support.google.com/drive/answer/2409045
- https://knowledge.workspace.google.com/admin/drive/manage-approvals
- https://knowledge.workspace.google.com/vault/getting-started/vault-overview
- https://knowledge.workspace.google.com/admin/reports/drive-log-events
- https://support.google.com/drive/answer/176692
- https://knowledge.workspace.google.com/admin/compliance/data-covered-by-data-regions
- https://learn.microsoft.com/en-us/sharepoint/document-library-version-history-limits
- https://support.microsoft.com/office/enable-and-configure-versioning-for-a-list-or-library-1555d642-23ee-446a-990a-bcab618c7a37
- https://learn.microsoft.com/en-us/microsoftteams/approval-admin
- https://www.microsoft.com/licensing/guidance/Microsoft-Purview
- https://learn.microsoft.com/en-us/purview/audit-solutions-overview
- https://learn.microsoft.com/en-us/microsoft-365/syntex/syntex-pay-as-you-go-services
- https://learn.microsoft.com/en-us/microsoft-365/enterprise/m365-dr-overview
- https://www.dropbox.com/plans
- https://help.dropbox.com/security/data-transfers-asia-pacific

**PH local / regulatory**
- https://archive-one.net/document-management-software-philippines-archive-one
- https://infobuilder.ph/
- https://ubix.com.ph/document-management/
- https://copylandia.com/dokmee/
- https://edamame-jp.com/kintone-philippines/en/blog/document-management-system-philippines/
- https://docs.cloud.google.com/vision/docs/languages
- https://learn.microsoft.com/en-us/azure/ai-services/document-intelligence/language-support/ocr
- https://resourcehub.bakermckenzie.com/en/resources/global-data-and-cyber-handbook/asia-pacific/philippines/topics/data-localization-and-regulation-of-non-personal-data
- https://privacy.gov.ph/wp-content/uploads/2024/06/Published-NPC-Advisory-No.-2024-01-Contractual-Clauses-for-Cross-Border-Transfers_30May24.pdf
- https://newsbytes.ph/2026/07/08/marcos-jr-meets-aws-execs-over-proposed-5-b-cloud-investment-in-ph/
- https://www.grantthornton.com.ph/insights/articles-and-updates1/tax-notes/eopt-is-here-updates-on-the-preservation-of-book-of-accounts-and-changes-in-taxpayer-registration/
- https://kpmg.com/ph/en/insights/2024/01/special-intax-january-2024-issue-1-volume-3.html
- https://dev.lawyerly.ph/laws/78650
- https://bdplaw.com.ph/news/amending-revenue-regulations-no-17-2013-dealing-%E2%80%9Cpreservation-books-accounts-and-other
- https://www.grantthornton.com.ph/technical-alerts/tax-alert/2026/bir-issues-guidelines-on-electronic-invoicing-retains-31-dec-2026-deadline/
- https://lawphil.net/courts/supreme/am/am_01-7-01_2001.html
- https://batasnatin.com/laws/cpra-c3-section-56-accounting-and-turn-over-upon-termination
- https://accralaw.com/2025/03/26/from-paper-to-pdf-the-digital-shift-in-philippine-notarization/

**Anecdotal and third-party estimates (complaints and price estimates only)**
- https://capterra.com/p/220466/DocuWare/reviews/
- https://www.capterra.com/p/122215/M-Files/reviews
- https://lexworkplace.com/netdocuments-pricing/
- https://www.g2.com/compare/netdocuments-vs-imanage-drive
- https://www.capterra.com/p/186749/TaxDome/reviews?page=4
- https://www.g2.com/products/procore
- https://costbench.com/software/construction-management/procore
- https://redresscompliance.com/oracle-aconex-per-project-vs-enterprise-licensing
- https://www.makios.com/articles/microsoft-365-pricing-changes-business-owners
- https://leadsmonky.com/best-google-workspace-reseller-in-philippines/
- https://www.dropboxforum.com/t5/Plans-and-Subscriptions/Dropbox-Standard-Plan-Activity-Log/m-p/267479
