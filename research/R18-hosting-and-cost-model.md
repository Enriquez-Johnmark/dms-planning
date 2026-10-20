# R18 — Hosting options and cost model

- **Status:** First pass, design-level. Research only, no code. **Written without knowing where Officetul is hosted today** (ERP questionnaire Q12 is unanswered), so every cost here is the **incremental** cost of adding Documents, built on the stated assumptions in §1.
- **Researched:** 2026-10-20. **All sources accessed 2026-10-20.**
- **Question:** Where should Officetul Documents be hosted, what does it cost to run per client company, and what does that imply for pricing (D-018)?
- **Labels:** **Verified fact** (read in the primary source: an official price page, a price-list API or official documentation), **Vendor claim**, **Secondary** (news, third party or search excerpt only), **Estimate** (our arithmetic), **Open question**, **Recommendation** (proposed, not approved). Source IDs `[S#]` point to §13.
- **Exchange rate:** **USD 1 = ₱63.088**, the BSP daily rate for **20 Oct 2026** [S60] (**Verified fact**). Peso figures are rounded and marked "≈". R06 used ₱62.76; the difference is about 0.5%.
- **Not a pricing decision.** Pricing belongs to the senior partner (D-018). §5 only gives options and illustrative arithmetic.

**Access caveats.**
- AWS prices come from the public **AWS Price List API** files for `ap-southeast-1`. The S3 and RDS web pricing pages did not render the rates. Publication dates: EC2 2026-10-15, RDS 2026-10-20, S3 2026-10-15, Lightsail 2026-09-15, SES 2026-09-11, GuardDuty 2026-10-08.
- The AWSDataTransfer file was read from a copy downloaded on 2026-10-18 (publication date 2026-09-16) and cross-checked against R06.
- **Pages that failed:**
  - Every Vultr pricing page (HTTP 403), and Vultr's database and object-storage API endpoints (they need a token).
  - Hetzner's USD toggle (JavaScript), so Hetzner prices are in EUR.
  - The Alibaba RDS regions page (404) and pricing page (413).
  - Huawei's pricing calculator.
  - ePLDT and VITRO websites (JavaScript-only shells).
- No published price list was found for any Philippine data-centre cloud.
- **Seconds of CPU per OCR page:** no primary or independent figure exists for OCRmyPDF with Tesseract 5. §1.4 uses our **own micro-benchmark** on synthetic pages. It is labelled **Estimate** and must be repeated in the Phase 7 benchmark (D-050).

---

## 1. Workload model

### 1.1 Components, and what Documents adds

| Component | Shared with Officetul, or new for Documents | Resource notes | Label |
|---|---|---|---|
| Web / PHP-FPM, Inertia | **Shared.** Documents adds signing endpoints for direct-to-storage uploads (D-046), so file bytes never pass through PHP. | Negligible increment | **Estimate** |
| Queue workers (Laravel) | **New pool**, kept separate from web (D-051). Jobs: promote, hash, fixity checks, previews, OCR dispatch. | 0.5–1 GB | **Estimate** |
| **ClamAV** (`clamd` + `freshclam`) | **New** | Minimum 3 GiB, preferred 4 GiB (2020 note). About 1.2 GiB for signatures, up to about 2.4 GiB during a concurrent reload [S65]. Signature files today: main 89.1 MB + daily 23.4 MB + bytecode 0.3 MB ≈ **112.8 MB compressed** [S66]. | **Verified fact** (docs); size **measured 2026-10-20** |
| **Gotenberg** (LibreOffice) | **New** | "At least 512Mi of memory" (Kubernetes); "at least 1Gi" (Cloud Run). **One LibreOffice instance cannot run conversions in parallel**, so heavy load needs more instances [S61]. | **Vendor claim** / **Verified fact** |
| **Apache Tika** server | **New** | Since 2.x it forks a child JVM, so two JVMs run. The sample config sets `-Xmx1g` [S62]. No official minimum RAM. | **Verified fact** (sample); sizing **Open question** |
| **OCRmyPDF + Tesseract 5** | **New** | About 15 bytes per pixel of the largest page per concurrent job [S63]. A 300-dpi letter page is about 8.4 MP, so ≈ 125 MB per job. Tesseract runs up to 4 threads per page; for batch work, use one process per core [S64]. | **Verified fact** (rule); **Estimate** (per page) |
| Poppler / libvips previews | **New**, on the worker | Small | **Estimate** |
| **MySQL 8.4** | **Shared** database (D-074). Documents adds rows, FULLTEXT and the activity log. | Sizing in §3.1 | — |
| Redis / Valkey | Shared if Officetul has one. Documents needs no Redis of its own: Laravel's database queue works at this scale. | AWS cache.t4g.micro Valkey costs $0.0192/hr (≈ $14/mo) if one is needed [S18] | **Verified fact** (price); need **Estimate** |
| Object storage | **New.** Buckets for quarantine, originals (versioned, D-073), derived, exports, and digests (Object Lock, D-061) [R17 §5] | §3.1 | — |
| Inbound email (scan-to-email, D-015) | **New** | SES receiving in Singapore, or Postmark (§2.5) | — |
| Outbound email | **Shared** (notifications) | Small increment | — |
| Backups | DB backups **shared**. A **second copy of originals is new.** | §4.2 | — |
| Monitoring, alerting | **Shared** tools; new checks for the worker host, queue backlog, ClamAV and freshclam age, fixity, and digest uploads | §6 | — |

**Worker host RAM (Estimate):** ClamAV 4 + Gotenberg 1–2 + Tika 1–2 + two OCR jobs 0.5 + PHP workers 1 + OS 1 ≈ **8.5–10.5 GB**. So **16 GB** is the planning size. An 8 GB host could work for the pilot only with ClamAV `ConcurrentDatabaseReload no` (scans pause briefly during reloads) [S65].

### 1.2 Storage per Document (Estimate)

| Item | Arithmetic | Per Document |
|---|---|---|
| Original file | 1.5 MB average (given) | 1.5 MB |
| Extra Versions and "file anyway" copies | R17 §6.1: 6.3M Original files per 5M Documents = ×1.26 | 1.89 MB |
| Derived files (OCR PDF, text, page images) | 0.6 × 1.89 | 1.13 MB |
| Non-current S3 versions (30–90-day recovery window, D-073) | About 2% of originals | 0.04 MB |
| **Total in object storage** | | **≈ 3.1 MB** (3.1 GB per 1,000 Documents) |
| Database (core rows + search text + FULLTEXT) | R17 §6.1 and R16 §6: about 35 GB core + 130–215 GB search for 5M Documents | 37–50 KB |
| Activity log | Section 09: 0.18–1.5 GB per company per year (500–4,000 events a day) | per company, not per Document |
| Log digests (Object Lock) | Hashes only; kilobytes per company per day | ≈ 0 |

R17 assumed derived files at 0.42× (4 TB derived per 9.5 TB originals). This note uses the requested 0.6×, which is more conservative.

### 1.3 Three sizing tiers (Estimate)

| Tier | Companies | Documents | Object storage | Pages OCR'd per month | Inbound emails per month | Users |
|---|---|---|---|---|---|---|
| **Pilot** (D-017: own company, then one client) | 2 | 10,000 by the end of the pilot | 31 GB | 12,000 (4,000 Documents × 3 pages) | 600 | ~30 |
| **Year 1** | 20 | 100,000 (20 × 5,000) | 310 GB | 25,000 on average, **75,000 in a back-file month** | 6,000 (300 per company) | ~200 |
| **Growth** (R17 scale) | 100 | 5,000,000 (100 × 50,000) | **15.3 TB** (9.45 TB originals + 5.67 TB derived + 0.19 TB non-current) | 300,000 (1,000 Documents × 3 pages per company) | 50,000 (500 per company) | ~1,500 |

**Assumptions:**
- 3 pages per Document (as in R17).
- Every page is OCR'd. This is conservative: OCRmyPDF skips pages that already have text.
- Emails average about 1.4 MB, because base64 inflates a 1 MB attachment, so each email is 6 chunks of 256 KB for SES.

### 1.4 OCR CPU (own micro-benchmark, **Estimate**)

**Setup:**
- Image `jbarlow83/ocrmypdf-alpine` (OCRmyPDF 17.13.1, Tesseract 5.5.2), Docker on an Apple M1 (8 cores).
- 10 synthetic 300-dpi letter pages of English and Filipino business words, with noise, blur and ±1.5° skew.
- `fil` model from tessdata_fast.

| Run | Wall time | CPU seconds per page |
|---|---|---|
| `eng`, 1 job, plain PDF output | 56.8 s | **5.7** |
| `eng+fil`, 1 job, `--deskew --rotate-pages`, PDF/A, sidecar text | 91.7 s | **9.2** |
| `eng+fil`, 8 jobs, same options | 30.8 s (3.1 s per page) | 14.0 (efficiency cores and contention) |
| Peak memory of a single process | — | 248 MB |

**Planning figure: 10 CPU-seconds per page** (range 6–15).
- Synthetic pages are cleaner than phone photos, and a cloud vCPU may be slower than an M1 performance core.
- Measure on the real VM class in Phase 7 (D-050).

| Tier | CPU-hours per month | Share of one 4-vCPU worker (2,920 vCPU-hours/month) | Label |
|---|---|---|---|
| Pilot | 12,000 × 10 s = **33** | ~1% | **Estimate** |
| Year 1, back-file month | 75,000 × 10 s = **208** | ~7% | **Estimate** |
| Growth | 300,000 × 10 s = **833**. Spread over ~176 office hours, that is about **4.7 vCPU busy on average**, and about 10 at peaks. | Needs a dedicated 8-vCPU OCR host | **Estimate** |

**Unit cost:**
- On a c7g.2xlarge ($0.3332/hr for 8 vCPU [S1]), 1,000 pages ≈ 2.78 vCPU-hours ≈ **$0.12 ≈ ₱7**.
- Cloud OCR costs $1.50 per 1,000 pages (R06 §4b), so self-hosted OCR stays more than 10× cheaper per page.
- **But the host is a fixed cost**, paid whether or not pages arrive (**Estimate**).

---

## 2. Hosting options

### 2.1 (a) AWS Singapore, `ap-southeast-1`

| Item | Price (USD) | Label |
|---|---|---|
| EC2 on-demand Linux, per hour (×730 = month) | t4g.xlarge (4 vCPU/16 GiB, burstable) 0.1696 ≈ **$124**. **m7g.xlarge** (4/16) 0.204 ≈ **$149**. m7i.xlarge (x86, 4/16) 0.252 ≈ $184. **c7g.2xlarge** (8/16) 0.3332 ≈ **$243**. c7i.2xlarge 0.4116 ≈ $300. r7g.xlarge (4/32) 0.2584 ≈ $189. r7g.2xlarge (8/64) 0.5168 ≈ $377. | **Verified fact** [S1] |
| EBS gp3 | $0.096/GB-month; snapshots $0.05/GB-month | **Verified fact** [S2] |
| Public IPv4 | $0.005/hr ≈ $3.65/month per address. NAT Gateway $0.059/hr ≈ $43/month (avoid for a small setup). | **Verified fact** [S11][S2] |
| **RDS for MySQL** per hour, Single-AZ / Multi-AZ | db.t4g.medium 0.102 / 0.203. **db.m7g.large (2/8)** 0.234 / 0.468 ≈ $171 / **$342**. db.m7g.xlarge 0.468 / 0.937. **db.r7g.xlarge (4/32)** 0.574 / 1.148 ≈ $419 / **$838** | **Verified fact** [S3] |
| RDS storage gp3 | $0.138 (Single-AZ) / $0.276 (Multi-AZ) per GB-month. Backup storage beyond the free allocation $0.095/GB-month. The size of the free allocation was not readable on the page. | **Verified fact** [S3]; allocation **Open question** [S5] |
| **RDS MySQL 8.4** | Supported (8.4.11 as of 21 Aug 2026). **Standard support ends 31 July 2029.** MySQL 8.0 is now **only under paid Extended Support** (standard support ended 31 July 2026). | **Verified fact** [S4] |
| RDS Extended Support (MySQL 8) | **$0.12 per vCPU-hour** in Singapore, years 1–2 (charged on every Multi-AZ instance) | **Verified fact** [S3][S5] |
| Lightsail (bundled VM, 744-hour month) | 16 GB/4 vCPU **$84** (6 TB transfer, **40% CPU baseline**, burstable); 8 GB $44; 32 GB $164. Lightsail managed DB offers **MySQL 5.7 and 8.0 only**, which conflicts with D-074. Object storage bundles $1 / $3 / $5 for 5 / 100 / 250 GB. | **Verified fact** [S6][S7][S8] |
| **S3 Standard** | $0.025/GB-month (first 50 TB). Standard-IA $0.0138. Glacier Instant Retrieval $0.005 (90-day minimum, $0.03/GB retrieval, $0.02 per 1,000 PUTs). | **Verified fact** [S9] |
| **S3 Intelligent-Tiering** | Frequent $0.025, Infrequent $0.0138, Archive Instant $0.005 per GB-month; monitoring $0.0025 per 1,000 objects per month | **Verified fact** [S9] |
| S3 requests | PUT/COPY/LIST $0.005 per 1,000; GET $0.0004 per 1,000. Batch Operations $0.25 per job + $1 per million objects. | **Verified fact** [S9] |
| Versioning | "Normal Amazon S3 rates apply for every version of an object stored or requested." No separate fee. | **Verified fact** [S10] |
| Object Lock | Available (R14 §2.3). Price page and FAQ excerpt read show **no separate fee**, but the Object Lock FAQ answer was not reached. | **Open question** (assume storage-only cost) |
| Internet egress | First 100 GB/month free, **aggregated across the account**, so it is shared with Officetul. Then $0.12/GB (first 10 TB). Inter-region from Singapore $0.09/GB. | **Verified fact** [S12] |
| KMS | $1 per customer-managed key per month; $0.03 per 10,000 requests. D-059 uses provider keys (SSE-S3), which have no KMS charge. | **Verified fact** [S13] |
| **SES receiving in Singapore** | `inbound-smtp.ap-southeast-1.amazonaws.com` exists. **$0.10 per 1,000 emails + $0.09 per 1,000 chunks of 256 KB.** | **Verified fact** [S15][S14] |
| SES sending | À la carte **$0.10 per 1,000** + $0.12/GB of attachments. **New since 2026:** new accounts default to the "Essentials" plan at **$0.16 per 1,000** (from 21 Jul 2026), and can switch to à la carte at any time. | **Verified fact** [S14] |
| **GuardDuty Malware Protection for S3, Singapore** | **$0.135/GB** scanned + **$0.323 per 1,000 objects** (first 1 GB and 1,000 objects a month free). This also answers R13 Open question 4: it is priced in Singapore, so it is offered there. | **Verified fact** [S16] |
| AWS Backup (S3) | Warm $0.06/GB-month; warm tiered $0.042 | **Verified fact** [S17] |
| CloudWatch | Custom metric $0.30/month; logs ingested $0.70/GB (Standard) or $0.35 (IA); logs stored $0.03/GB-month; alarm $0.10. Free tier: 5 GB logs, 10 metrics, 10 alarms. | **Verified fact** / **Vendor claim** [S19] |

**Fit with confirmed decisions:**
- **Object Lock compliance mode** (D-061), **versioning and lifecycle including `NoncurrentVersionExpiration`** (D-073), multipart uploads (D-046), SES inbound in the same region (D-015), RDS MySQL 8.4 (D-074), and GuardDuty as an optional second engine (R13 §3.3).
- Everything is in one region and one account.

**ARM caveat:** the cheapest prices above are Graviton (ARM). Whether the ClamAV, Gotenberg and Tika images we pick ship `arm64` builds is an **Open question**. The x86 equivalents cost about **+24%** (m7i.xlarge $184 vs m7g.xlarge $149) [S1].

### 2.2 (b) Cheaper VPS route, Singapore

| Provider | Singapore? | VM ~4 vCPU/8 GB, ~6–8 vCPU/16 GB (USD/month, transfer) | Managed MySQL | S3-compatible storage in Singapore | Versioning / lifecycle / **Object Lock** | Label |
|---|---|---|---|---|---|---|
| **Akamai / Linode** | `sg-sin-2`, with Linodes, Object Storage and Managed Databases [S44] | $48 (5 TB). **6 vCPU/16 GB $96 (8 TB).** Dedicated 8 vCPU/16 GB $144 (6 TB). Egress $0.005/GB, pooled. | **MySQL 8.4 only.** 2 vCPU/4 GB: $63 single node, $147 for 3 nodes. 4 vCPU/8 GB: $126 / $294 [S45] | $5/month for 250 GB + 1 TB transfer; then $0.02/GB, egress $0.005/GB [S46] | Versioning **yes**; lifecycle **yes, including `NoncurrentVersionExpiration`** and abort-multipart; **Object Lock Governance and Compliance**, set at bucket creation [S47][S48][S49] | **Verified fact**. Whether Object Lock works on the `sg-sin-2` endpoint is an **Open question** (R14 §2.3). |
| **DigitalOcean** | `sgp1` [S39] | $48 (5,000 GiB). 8 vCPU/16 GB $96 (6,000 GiB). General Purpose 4/16 $126. Overage $0.01/GiB [S40][S41] | **MySQL 8.4 offered**; 8.0 clusters are force-upgraded from 30 Oct 2026. 2/4 single node $60.90; 4/8 $122.10. HA is about double (our reading of the docs). [S42] | Spaces: $5 for 250 GiB + 1 TiB; then $0.02/GiB [S43] | Versioning yes (API only). **`NoncurrentVersionExpiration` not documented. Object Lock compliance not documented.** [S43] | **Verified fact** / **Open question** |
| Vultr | `sgp`; object storage `sgp1` [S50] | 4/8 $40 (4 TB); 8/16 $96 (8 TB) [S50] | "MySQL version 8", not stated as 8.4. Pricing page blocked. | $18/month for 1,000 GB + 1,000 GB egress, then $0.018/GB [S51] | Versioning yes; lifecycle "yes" (rules not listed); lock is an on/off toggle with **modes undocumented** [S51] | **Verified fact** / **Open question** |
| Hetzner | `sin` exists (colocation; only CPX and CCX types) [S52] | **Singapore prices are higher than in the EU:** CPX32 €49.49 (2 TB); CPX42 (8/16) €93.99 (3 TB); CCX23 (dedicated 4/16) €108.99. Overage €7.40/TB. [S53] | **None found** | **No.** Object storage only in fsn1, nbg1, hel1 (EU) [S54] | — | **Verified fact** |

**Reading (Estimate):**
- **Akamai/Linode Singapore** is the only VPS route here that documents everything Documents needs: MySQL 8.4, `NoncurrentVersionExpiration` and Object Lock compliance. It is option B in §3.
- DigitalOcean would need the log digests in AWS S3 Singapore (R14 §2.3), and lacks a documented clean-up rule for old versions.
- Hetzner Singapore has no object storage or managed MySQL there, so it fails D-061, D-073 and D-074 without extra services.
- Vultr could not be assessed.
- Every VPS route still needs **an inbound mail service**. SES Singapore works from any host.

### 2.3 (c) Philippine data centres and clouds

| Option | What exists | Published prices | Fit | Label |
|---|---|---|---|---|
| **AWS Local Zone Manila** (`ap-southeast-1-mnl-1a`, parent Singapore) | GA posted 19 Jun 2023. EC2 T3, C5, R5, M5 (12 types in the price list), **EBS gp2 only**, ALB, ECS, EKS. **No S3 and no RDS.** [S20][S21] | t3.xlarge $0.2534/hr (Singapore $0.2112, **+20%**); c5.2xlarge $0.470; r5.2xlarge $0.730. gp2 **$0.222/GB-month**. Manila→Singapore transfer $0.00/GB; internet egress $0.12/GB [S22] | Compute in Manila only. **Files (S3) and database (RDS) would still be in Singapore**, unless we self-manage MySQL and files on EBS. That gives up Object Lock, versioning and lifecycle. | **Verified fact** |
| AWS full Philippines region | **None announced.** Only Saudi Arabia and Chile are listed as announced. [S23] | — | — | **Verified fact** |
| Google Cloud, Azure | **No Philippines region**, and none announced on the official lists [S24][S25] | — | — | **Verified fact** |
| **Alibaba Cloud Manila** `ap-southeast-6` (since 2021) | OSS endpoint `oss-ap-southeast-6.aliyuncs.com` [S26][S27]. Bucket WORM with versioning documented, but **not confirmed for Manila** [S29]. ApsaraDB RDS MySQL in Manila **not confirmed**. | **OSS Manila:** Standard **$0.0160/GB-month** (first 5 GB free), IA $0.00935, Archive $0.0043; egress first 100 GB free, then **$0.076/GB** [S28]. ECS prices: dynamic calculator, not captured. | Storage is cheaper than S3 Singapore. Database and compute prices are unknown. New vendor and tooling for a 1–2 person team. | **Verified fact** (OSS); **Open question** (RDS, ECS, WORM in Manila) |
| **Huawei Cloud Philippines** | Launched 3 Dec 2024, "first local 3-AZ public cloud", "over 100 services" (none listed) [S30]. OBS in Manila `ap-southeast-5` [S31]. | Not found | Unknown MySQL 8.4 and Object Lock (R14: WORM compliance-only, **Secondary**) | **Vendor claim** / **Verified fact** (OBS) |
| **Converge** (cloud; "sovereign cloud" unveiled 28 May 2026) | "We keep your data in the Philippines"; "fixed fee cloud subscription" [S32][S33] | **None published** | No S3, VM or Object Lock detail | **Vendor claim** / **Secondary** |
| **ePLDT / VITRO** (incl. VITRO Sta. Rosa, launched Apr 2025) | "Virtual compute and storage" with SLAs; GPU-as-a-Service [S34] | **None published**; websites are JavaScript-only | No S3-compatible or Object Lock product found | **Secondary** |
| STT GDC Philippines (Globe/Ayala JV), Digital Edge | **Colocation** (rack space), not cloud VMs or object storage [S35] | — | We would run our own hardware. Out of scope for 1–2 developers. | **Secondary** |

**Reading (Estimate):**
- **No Philippine option publishes all the prices we need.**
- None is confirmed to offer MySQL 8.4 as a managed service together with S3-compatible versioning, lifecycle and Object Lock.
- The AWS Local Zone moves compute, not data.

### 2.4 (d) Hybrid: application in the Philippines, digests in AWS Singapore

- **Shape:** app, database and files in a Philippine region (Alibaba or Huawei Manila, or a local provider), with only the daily **log digests** (hashes, D-061) in a compliance-mode S3 Singapore bucket.
- **Cost:** the digest bucket costs a few US cents a month (**Estimate**). R14 §2.3 already proposes this as the fallback when the main store lacks Object Lock.
- **Trade-off (Estimate):**
  - It keeps client files in the Philippines, which may help some sales conversations.
  - But it adds a second cloud vendor, an unknown database service and unpublished prices, all of which must be learned and patched by 1–2 people.
- **Viable only** once a Philippine provider confirms in writing: managed MySQL 8.4, S3 multipart, versioning with `NoncurrentVersionExpiration`, prices, and a data processing agreement.

### 2.5 Inbound email

| Service | Price | Data location | Label |
|---|---|---|---|
| **Amazon SES, Singapore** | $0.10 per 1,000 + $0.09 per 1,000 chunks of 256 KB. A 1.4 MB email ≈ $0.00064. | Singapore (same region as option A) | **Verified fact** [S14][S15] |
| Postmark | Pro $16.50/month for 10,000 emails; overage $1.30 per 1,000. **Inbound is not on Basic.** A processed inbound message counts as 1 email. | "Deft's data center (located outside of Chicago), and AWS"; no EU or Asia hosting | **Vendor claim** / **Verified fact** [S58] |
| Mailgun | Foundation $35/month for 50,000 emails, "full access to inbound routing" | Not checked | **Vendor claim** [S59] |

**Recommendation:** use SES Singapore for inbound with every hosting option.
- It is cheapest at our volumes.
- It keeps captured email in the same country as the files.
- Postmark would add a **US** sub-processor and a second cross-border location to disclose (R10 §7, D-032 sub-processor list).

### 2.6 Latency from Manila

| Route | Median of 30-ping run averages, 6–20 Oct 2026 | Label |
|---|---|---|
| Manila → Singapore | **29.0 ms** (runs 27.7–35.4 ms) | **Secondary** [S36] |
| Manila → Hong Kong | 18.5 ms | **Secondary** [S37] |

A web app adds about 30 ms per round trip from Singapore. Upload speed (R13 §1.2) matters far more for users than this latency (**Estimate**).

### 2.7 Legal note (no conclusions)

- **Data Privacy Act:**
  - It **does not prohibit** processing outside the Philippines.
  - The client (as controller) stays accountable for "comparable protection".
  - Our contract must state the **location of processing** (R10 §7; IRR s.44(a)).
- **EO 119 (2026):**
  - It sets government data classification and residency and says it "does not apply to the private sector or commercial data owned by private entities".
  - It **does cover government data handled by private contractors**: Top Secret stays in the Philippines; Confidential stays unless approved otherwise [S38].
  - Read on Lawphil, a mirror of the legal text, with inconsistent signing dates on the page. Treat as **Secondary** until read in the Official Gazette.
- **BSP Circular 1137** applies to BSP-supervised institutions (R10 §7, **Secondary**).
- **Lawyer question A12** ("any reason to prefer hosting in the Philippines over Singapore?") is **still unanswered**. Add a follow-up: does EO 119 affect a client that is a **government contractor** and stores government data in Documents? (**Open question**)

---

## 3. Cost per tier: best two options (Estimate)

**Option A:** AWS Singapore (EC2 worker hosts, S3, SES, RDS).
**Option B:** Akamai/Linode Singapore (Linode worker, Linode Object Storage, Linode Managed MySQL 8.4), plus SES Singapore for inbound and AWS S3 Singapore for the backup copy (Growth).

**What is counted:** only **Documents' incremental** monthly cost. Officetul's web servers, its base database instance, its outbound email base and shared tooling are **excluded**.

**Main assumptions (Estimate):**
- Officetul's database already runs on a managed MySQL of about 2 vCPU/8 GB with free disk (for example RDS db.m7g.large Multi-AZ).
- Officetul's existing egress uses part of AWS's 100 GB free tier.
- Prices are on-demand, without Savings Plans or reserved terms (those were not fetched).

### 3.1 Option A, AWS Singapore (USD per month)

| Line | Pilot | Year 1 | Growth | Arithmetic |
|---|---|---|---|---|
| Worker host(s) | 123.81 | 148.92 | 392.16 | Pilot t4g.xlarge (burstable is fine at ~1% OCR load). Year 1 m7g.xlarge. Growth: m7g.xlarge for intake (ClamAV, Gotenberg, Tika) + c7g.2xlarge for OCR [S1] |
| EBS scratch (gp3) | 3.84 | 5.76 | 11.52 | 40 / 60 / 120 GB × $0.096 [S2] |
| Public IPv4 (freshclam and updates) | 3.65 | 3.65 | 7.30 | 1–2 × $3.65 [S11] |
| S3 storage | 0.78 | 7.75 | **245.03** | 31 / 310 GB × $0.025. Growth uses Intelligent-Tiering on originals (see note); all-Standard would be **$382.50** [S9] |
| S3 requests (incl. fixity GETs) | 0.40 | 0.91 | 10.32 | ~15 PUTs per Document; previews and fixity GETs [S9] |
| Backup copy of originals (separate account, same region, Glacier IR) | 0.20 | 1.16 | 49.77 | Originals only: 19 / 189 / 9,450 GB × $0.005 + PUTs at $0.02 per 1,000 [S9] |
| Digest bucket (Object Lock) | 0.01 | 0.01 | 0.05 | Hashes only |
| SES inbound | 0.38 | 3.84 | 32.00 | e.g. Year 1: 6,000 × $0.0001 + 36,000 chunks × $0.00009 [S14] |
| SES outbound (notifications) | 0.20 | 2.00 | 20.00 | 2k / 20k / 200k emails × $0.10 per 1,000 (×1.6 on the Essentials plan) [S14] |
| Database increment | 1.38 | 9.66 | **606.80** | Pilot 5 GB and Year 1 35 GB × $0.276 (Multi-AZ gp3). Growth: 400 GB × $0.276 + the step from db.m7g.large to **db.r7g.xlarge Multi-AZ** ($0.68/hr × 730) for a 16–32 GB buffer pool (R16 §6) [S3] |
| Internet egress | 0 | 9.00 | 54.00 | Year 1 ~73 GB (partly free). Growth ~550 GB − 100 GB free = 450 × $0.12 [S12] |
| Monitoring / error tracking | 0 | 26.00 | 50.00 | Sentry Developer free → Team $26; CloudWatch logs beyond the free tier [S67][S19] |
| **Total, USD** | **134.64** | **218.66** | **1,478.95** | |
| **Total, PHP @ 63.088** | **≈ ₱8,494** | **≈ ₱13,795** | **≈ ₱93,304** | |
| + 12% VAT (reverse charge on foreign cloud, R06 §5) | ≈ ₱9,514 | ≈ ₱15,450 | ≈ ₱104,500 | Whether this is claimable input VAT: **Open question** (R06) |
| **Per company per month (ex VAT)** | **≈ ₱4,247** | **≈ ₱690** | **≈ ₱933** | ÷ 2 / 20 / 100 |
| Optional: GuardDuty S3 second engine | ~0.3 | 5.06 | 65.75 | Year 1: (15.75 − 1) GB × $0.135 + (10,500 − 1,000) × $0.000323 [S16] |

**Growth S3 note:** originals held as 10% Frequent, 20% Infrequent and 70% Archive Instant (an assumption: old receipts are rarely opened). That costs $82.79 for storage + $15.75 monitoring for 6.3M objects. Derived files stay in Standard at $141.75, because small objects are not auto-tiered. **Estimate**

### 3.2 Option B, Akamai/Linode Singapore (USD per month)

| Line | Pilot | Year 1 | Growth | Arithmetic |
|---|---|---|---|---|
| Worker host(s) | 96.00 | 96.00 | 240.00 | 6 vCPU/16 GB shared $96. Growth adds a dedicated 8 vCPU/16 GB at $144 [S44] |
| Object storage | 5.00 | 6.20 | 306.00 | $5 includes 250 GB; then $0.02/GB. Growth: 5 + 15,050 × 0.02 [S46] |
| Backup copy of originals | 0.26 | 1.16 | 50.72 | Second bucket or AWS Glacier IR Singapore at $0.005/GB + Linode egress $0.005/GB |
| Digests (Object Lock) | 0.01 | 0.01 | 0.05 | Linode lock if confirmed in `sg-sin-2`; otherwise an AWS S3 Singapore bucket |
| SES inbound + outbound | 0.58 | 5.84 | 52.00 | As option A |
| Database increment | 0 | 0 | **882.00** | Year 1 assumes the existing plan's disk absorbs 35 GB; otherwise +$147 for one 3-node plan step. Growth: 3-node plan step from 8 GB to 32 GB RAM, **extrapolated linearly** from the published 4 GB and 8 GB prices ($294 → ~$1,176). **Open question.** [S45] |
| Egress | 0 | 0 | 0 | Inside the pooled transfer allowances |
| Monitoring | 0 | 26.00 | 50.00 | As option A |
| **Total, USD** | **101.85** | **135.21** | **1,580.76** | |
| **Total, PHP @ 63.088** | **≈ ₱6,426** | **≈ ₱8,530** | **≈ ₱99,727** | |
| + 12% VAT | ≈ ₱7,197 | ≈ ₱9,554 | ≈ ₱111,695 | |
| **Per company per month (ex VAT)** | **≈ ₱3,213** | **≈ ₱427** | **≈ ₱997** | |

### 3.3 Reading the tiers (Estimate)

- **Pilot and Year 1:** cost is dominated by **one fixed worker host** (about 70–90% of the total). Option B saves about **₱5,300 a month** at Year 1, mainly because the Linode 16 GB VM costs $96 against $149 for m7g.xlarge.
- **Growth:** the cost moves to the **database step** (41% of A) and **storage** (17–25%).
  - Option A gets cheaper storage through Intelligent-Tiering and Glacier IR.
  - Option B's managed MySQL is priced linearly per GB of RAM.
  - The two options roughly meet at about ₱1,000 per company.
- **D-064 will likely fire at Growth.** Searchable text of about 85 GB is above the 20 GB trigger. A self-hosted Meilisearch host (e.g. EC2 r7g.2xlarge, 64 GB, ≈ $377 [S1]) would then take the search text out of MySQL. That changes, but does not remove, the database step.
- **This replaces the earlier "about ₱48 per tenant" figure** (R06 §4d, Section 02 §3). That figure counted only variable storage, OCR and email costs. **The fixed worker host, the database step, monitoring and ops time dominate** until there are about 50 or more companies.
- **Cost cuts not priced here:**
  - Graviton Savings Plans or reserved terms (**Open question**).
  - An 8 GB pilot worker (t4g.large ≈ $62) with `ConcurrentDatabaseReload no`.
  - A Lightsail 16 GB worker at $84, with a 40% CPU baseline [S8].

---

## 4. Cost per company and cost drivers

### 4.1 Unit costs (Option A prices; Estimate unless marked)

| Driver | Unit cost | Per company in the Year 1 shape (5,000 Documents, 1,250 pages and 300 emails a month) | Per company in the Growth shape (50,000 Documents) |
|---|---|---|---|
| **Stored Documents** (object storage + backup copy + database rows) | **≈ $0.103 ≈ ₱6.5 per 1,000 Documents per month** (≈ ₱7.3 with VAT) | ≈ ₱33/month | ≈ ₱325/month, and it keeps growing every month |
| Per GB, 50 GB block (S3 Standard + backup share) | ≈ ₱99 per 50 GB per month incl. VAT (≈ ₱2/GB) | — | — |
| OCR pages (self-hosted, marginal) | ≈ $0.12 ≈ ₱7 per 1,000 pages | ≈ ₱9 | ≈ ₱21 |
| Inbound email (SES) | ≈ $0.64 ≈ ₱40 per 1,000 emails of 1.4 MB | ≈ ₱12 | ≈ ₱20 |
| Optional GuardDuty | $0.135/GB + $0.323 per 1,000 files [S16] | ≈ ₱16 | ≈ ₱41 |
| **Fixed share** (worker host, database step, monitoring) | — | ≈ ₱610 (A) / ₱385 (B) | ≈ ₱670 (A) |
| **Ops time** (§6) | 12–16 h/month (Year 1), 20–30 h (Growth) | ≈ ₱420 at an assumed ₱600 per hour (**replace with the real rate**) | ≈ ₱150 |

**Main drivers, in order:**
1. **Fixed hosts and ops time** at small scale.
2. **Accumulated storage**, together with the database size it drags along, at scale.
3. **Back-file scanning bursts.** These need temporary OCR capacity, not money per page.

Email and per-page OCR costs are minor. **Users cost almost nothing** directly, because they add requests, not storage.

### 4.2 What to meter or limit in plans (**Recommendation**)

| Meter | Why | Example |
|---|---|---|
| **Storage quota per company (GB)**, with paid top-up blocks | Tracks the one cost that grows every month for ever (D-041: no automatic purge) | 25 GB included (≈ 8,000 Documents at 3.1 MB). Extra 50 GB blocks cost us ≈ ₱99/month. |
| **OCR pages per month (fair-use)**, plus a one-off **back-file OCR pack** | Protects the shared worker queue from one company's 40,000-page back-file month; it is not a cost-per-page issue | 3,000 pages/month included; a back-file project is quoted separately and queued off-hours |
| Inbound email size and volume | Already capped at about 35 MB per message (D-045); a monthly fair-use number stops abuse | 1,000 emails/month |
| AI extraction (later release) | Real per-page cost ($10 per 1,000 pages, R06) | Quota plus packs (R06 §9.4) |
| **Users: do not meter for cost reasons** | Users add almost no cost. Per-user pricing is a market choice, not a cost one. | — |

---

## 5. Pricing implications for D-018 (options only; the decision belongs to the senior partner)

### 5.1 Competitor anchors (R06, converted at ₱63.088)

| Anchor | ≈ PHP per month | Basis |
|---|---|---|
| Odoo Standard, per user (Documents included) | ₱565–707 per user | US$8.95–11.20 |
| Zoho Books autoscan add-on | ₱505–631 per 50 scans | US$8–10 |
| Hubdoc standalone; included in Xero PH plans | ₱757; ₱0 extra in Xero | US$12 |
| Dext Business (250 docs, 5 users) | ₱1,590 | US$25.21 |
| Juan Essentials (whole accounting product) | ₱2,000 per organisation | Published PHP price |

### 5.2 Illustrative margins (**Estimate, not a price proposal**)

**Cost basis:**
- Section 3 cloud cost including 12% VAT.
- Plus ops time at an assumed ₱600 per hour: 14 h/month in Year 1, 25 h at Growth.
- **Excludes** development cost, support desk time, sales commission and Officetul's shared infrastructure.
- Revenue is VAT-exclusive.

| Scenario | Full cost per company | Add-on ₱500 | ₱1,000 | ₱1,500 | ₱2,000 | Per user ₱150 (10 users Y1 / 15 at Growth) |
|---|---|---|---|---|---|---|
| Year 1, option A | ≈ ₱1,192 | −138% | −19% | 20% | 40% | 20% |
| Year 1, option B | ≈ ₱898 | −80% | 10% | 40% | 55% | 40% |
| Growth, option A | ≈ ₱1,195 | −139% | −20% | 20% | 40% | 47% |
| Growth, option B | ≈ ₱1,267 | −153% | −27% | 16% | 37% | 44% |

**Break-even (option A, Year 1 shape):**
- Fixed cost ≈ ₱21,400/month: worker, disk, IP and monitoring with VAT (≈ ₱13,000) + 14 ops hours (≈ ₱8,400).
- Variable cost ≈ ₱121 per company.
- Companies needed at each flat add-on price:

| Flat add-on per company | Companies needed |
|---|---|
| ₱500 | ≈ **57** |
| ₱1,000 | ≈ **24** |
| ₱1,500 | ≈ **16** |
| ₱2,000 | ≈ **11** |

- Without counting ops time: about 34 / 15 / 9 / 7 companies.

### 5.3 Pricing models against these costs (Estimate)

| Model | What the cost model says |
|---|---|
| **Included in Officetul, with a storage quota** (R06 and Section 02 recommendation) | Each company costs about **₱430–1,000/month** in cloud cost alone, or **₱900–1,270** including ops time. That must fit inside Officetul's subscription margin. Officetul's current price is unknown (Section 02 Open question). The quota stops the Growth-shape storage tail. |
| Flat add-on per company | Below about ₱1,000 it loses money at 20 companies. ₱1,500–2,000 covers cost, but sits at or above Dext and Juan's whole accounting product. |
| Per user | Matches Odoo's model. Revenue scales with seats, but cost scales with **documents**, so a 3-user firm with 100 GB would be under-priced. |
| Storage-tier add-ons | A 50 GB block costs about ₱99/month incl. VAT. Even a ₱300 block has a large margin, but R06 warns that small blocks "look petty". |
| Hybrid: included base + storage blocks + back-file OCR packs | Covers the fixed cost through the subscription, and the long-tail costs through what actually grows |

**For the senior partner:** the fixed cost (one worker host plus ops time) is roughly **₱13,000–21,000 a month** whether there are 2 or 20 companies. Pricing has to cover that floor, not just the per-document cost.

---

## 6. Operating model

### 6.1 Ops tasks and time (Estimate)

| Task | Frequency | Hours per month (Year 1 / Growth) | Notes |
|---|---|---|---|
| OS and container patching (worker hosts; ClamAV, LibreOffice, Tika images); urgent CVEs | Monthly + ad hoc | 3 / 5 | clamd itself has had code-execution CVEs (R13 §3.1) |
| ClamAV signatures | Automatic `freshclam` (12 checks a day default); alert when signatures are older than 24 h | 0.5 / 1 | Cloud IPs "will most likely be rate limited" (R13 §3.1) |
| False positives and held files (D-047, D-048) | Ad hoc | 1 / 3 | |
| Alerts: queue backlog, fixity failures, digest upload, disk, errors | Daily glance + on-call | 3 / 6 | |
| **Restore tests** (DB point-in-time restore, plus an object restore from the backup copy) | Quarterly | 1.5 / 2 | 4–6 h per drill ÷ 3 |
| Security log review (support access, MFA, refused access; D-057, D-060) | Weekly | 2 / 3 | |
| Incident duty (D-032: written procedure, 24-hour breach notice to clients) | On-call rota + 2 tabletop drills a year | 1 / 2 | Needs a **named** primary and backup person |
| Capacity and cost review, upgrades (MySQL minor versions, PHP) | Monthly | 1 / 3 | |
| **Total** | | **≈ 12–16 h / 20–30 h** | |

### 6.2 Managed services that reduce ops for 1–2 people

| Choice | Saves | Costs | Label |
|---|---|---|---|
| **Managed MySQL 8.4** (RDS, Linode, DO) instead of self-managed | Backups, point-in-time restore, patching, failover | About 2× the raw VM price (e.g. RDS r7g.xlarge Single-AZ $419 vs EC2 r7g.xlarge $189) [S1][S3] | **Verified fact** (prices) / **Estimate** |
| If Officetul is on **RDS MySQL 8.0** today | — | Extended Support **$0.12/vCPU-hour**: a 2-vCPU Multi-AZ instance pays about **$350/month extra** (≈ ₱22,100) until it upgrades. This strengthens D-074. | **Verified fact** [S3][S4] / **Estimate** |
| SES inbound (vs running our own MTA) | No mail server to patch; email lands in S3 | Pennies (§2.5) | **Verified fact** |
| Laravel Forge ($12–39/month) or Ploi ($10–36) | Server provisioning, deploys, SSL | Small | **Verified fact** [S55][S57] |
| Laravel Cloud (Singapore region listed; Pro 16 GiB/4 vCPU $128/month) | Fully managed app platform | Whether it can run separate ClamAV, Gotenberg and Tika containers with no network (D-051) was not checked. Likely only for web and queue, with the tool containers elsewhere. | **Verified fact** (price, region) [S56]; fit **Open question** |
| GuardDuty S3 malware scanning | A second engine; no host to patch | ~$5/month (Year 1) to ~$66 (Growth). **Does not replace ClamAV** (D-047). | **Verified fact** (price) |
| Container services (ECS/Fargate) for the tool containers | Host patching | Not priced in this note | **Open question** |
| Uptime and error tracking | — | Better Stack free (10 monitors); UptimeRobot Solo $9–10; Sentry Team $26 [S67][S68][S69] | **Vendor claim** |

---

## 7. Recommendations summary

1. **Host Documents in Singapore, in the same provider and region as Officetul**, once Q12 is answered.
   - If Officetul is not in Singapore, or its host lacks S3-compatible storage with versioning, lifecycle and Object Lock, **default to AWS `ap-southeast-1`** (option A).
   - Option A covers D-046, D-061, D-073 and D-074 in one account, with SES inbound in the same region.
   - **Option B (Akamai/Linode Singapore)** is a credible cheaper route for Pilot and Year 1 (≈ ₱5,300 less a month at Year 1). Confirm Object Lock on `sg-sin-2` first. **Recommendation**
2. **No Philippine hosting for the MVP.**
   - No Philippine provider publishes the prices and features we need, and the AWS Manila Local Zone has no S3 or RDS.
   - Revisit only if the lawyer (A12) or a paying client requires it, e.g. a government contractor under EO 119. **Recommendation**
3. **One dedicated 16 GB worker host for the tool containers**, separate from web servers (D-051).
   - Pilot ≈ ₱7,800–8,500/month (AWS) or ≈ ₱6,100 (Linode), before VAT.
   - Add an 8-vCPU OCR host at Growth. **Recommendation**
4. **Protect originals with a second copy in a separate account in the same region**, in Glacier Instant Retrieval: ≈ ₱73/month at Year 1.
   - Turn on Intelligent-Tiering for originals above about 1 TB stored.
   - Versioning alone (D-073) does not protect against account compromise or operator error. **Recommendation**
5. **Use managed MySQL 8.4.** If Officetul is on RDS 8.0, upgrade now: Extended Support is already being charged. **Recommendation**
6. **Use SES Singapore for inbound email** with any host. Do not use Postmark, which is US-hosted. **Recommendation**
7. **Meter storage (GB) and OCR pages per month (fair-use), not users;** sell back-file OCR as a project. **Recommendation**
8. **Budget 12–16 ops hours a month in Year 1, and name an on-call primary and backup** (D-032). **Recommendation**
9. **Replace the "₱48 per tenant" figure** in Section 02 with this note's figures: about ₱430–1,000 per company per month in cloud cost, or ₱900–1,270 including ops time, at Year 1 and Growth scale. **Recommendation**

## 8. Open questions

1. **Q12:** where Officetul runs today, on how many servers, and whether its database is RDS or self-managed, MySQL 8.0 or 8.4. This decides whether option A or B is "the same place as Officetul".
2. Does Akamai `sg-sin-2` support Object Lock **compliance mode** on its endpoint type? Does DigitalOcean Spaces support `NoncurrentVersionExpiration`?
3. Do our chosen ClamAV, Gotenberg and Tika images run on **arm64** (Graviton)? If not, add about 24% to the worker cost.
4. What are the real OCR CPU seconds per page on the production VM class, with real phone photos and scans (D-050)?
5. Linode Managed MySQL prices above 8 GB RAM, and Vultr MySQL 8.4 availability and prices.
6. What are Savings Plan or reserved prices for EC2/RDS in Singapore (1-year, no upfront)?
7. Is S3 Object Lock really free of any fee beyond storage? (The FAQ answer was not reached.) How big is the RDS free backup allocation?
8. Is same-region replication free of data-transfer charges (request charges only)?
9. Does SES's new Essentials plan change **inbound** pricing, or only sending?
10. Is the 12% reverse-charge VAT on foreign cloud bills creditable input VAT for Officetul's selling entity? (R06)
11. A12 follow-up: does EO 119 affect clients that are government contractors?
12. Alibaba Manila: ApsaraDB RDS MySQL availability and version, ECS prices, and OSS WORM in Manila. Huawei Manila: ECS, RDS and prices.
13. Officetul's current subscription price and the senior partner's target margin, needed to judge "included vs add-on".

---

## 9. Decisions for the owner

Each is worded so that "yes" accepts the recommendation. Plain examples use the Year 1 shape (20 companies).

**Decision 1: Host Documents in Singapore with Officetul's provider; if that provider can't meet our storage needs, use AWS Singapore.**
- *Example:* the maintainer answers Q12: "Officetul runs on one DigitalOcean droplet in Singapore." Spaces can't clean up old file versions or lock the digests. So the Documents files, the worker host and inbound email go to AWS Singapore, about ₱13,800 a month at 20 companies. The app keeps running where it is, about 30 ms from Manila.
- **Recommended answer: yes.**

**Decision 2: Don't build for Philippine hosting in the MVP; revisit only if the lawyer (A12) or a paying client requires it.**
- *Example:* a construction client that is a DPWH contractor asks whether files stay in the Philippines. We say they are in Singapore under our data processing agreement, and pass the question to the lawyer. We don't move hosting for one client before the lawyer answers.
- **Recommended answer: yes.**

**Decision 3: Run ClamAV, LibreOffice, Tika and OCR on one dedicated 16 GB worker server, separate from Officetul's web servers.**
- *Example:* during the pilot this costs about ₱7,800 a month on AWS (or about ₱6,100 on Linode). When the client scans 40,000 pages of old files in one month, only the worker queue slows down. Officetul's Accounting screens don't.
- **Recommended answer: yes.**

**Decision 4: Keep a second copy of every original file in a separate backup account in the same Singapore region.**
- *Example:* at Year 1 this costs about ₱73 a month. If someone deletes the main storage, or a leaked key is used to wipe it, the clients' 189 GB of originals can still be restored from the backup account.
- **Recommended answer: yes.**

**Decision 5: Use managed MySQL 8.4, and if Officetul is on Amazon RDS MySQL 8.0, upgrade it now.**
- *Example:* a 2-vCPU Multi-AZ RDS database on 8.0 pays about ₱22,100 a month extra in Extended Support since 1 August 2026. Upgrading (D-074) stops that charge.
- **Recommended answer: yes.**

**Decision 6: Plans limit storage and monthly OCR pages, not users; large back-file scanning is quoted as a separate project.**
- *Example:* a law firm with 4 users gets 25 GB and 3,000 OCR pages a month. Each extra 50 GB costs us about ₱99 a month. Scanning 30 years of case files (60,000 pages) is quoted as a back-file project and processed overnight.
- **Recommended answer: yes.** The actual quota sizes and prices go to the senior partner (Decision 8).

**Decision 7: Budget about 12–16 hours a month of ops in Year 1, with a named on-call person and a named backup.**
- *Example:* the developer patches the worker host on the first Monday of each month and runs a restore test each quarter. Whenever ClamAV signatures are more than a day old, they get an alert. If they are on leave, the backup person carries the 24-hour breach-notice duty (D-032).
- **Recommended answer: yes.**

**Decision 8: Bring this cost model and the pricing options in §5 to the senior partner for the D-018 pricing decision.**
- *Example:* the senior partner sees two numbers. Serving one company costs about ₱900–1,270 a month including ops time (₱430–1,000 in cloud cost alone). A flat ₱1,000 add-on loses money until about 24 companies, but bundling into Officetul with a 25 GB quota keeps the "documents included" pitch. They then choose.
- **Recommended answer: yes.**

---

## 10. Statement of limits

- No Officetul code or hosting details were seen. Every number is incremental to an **assumed** Officetul baseline.
- Sizes and traffic are estimates. Measure them in the own-company pilot (D-017).
- The OCR figure comes from 10 synthetic pages on an Apple M1, not from real scans or a cloud VM.
- Linode's Growth database price is a linear extrapolation.
- Ops hours are judgement, not measurement. The ₱600 hourly rate is a placeholder.
- Margins exclude development, support desk, sales and Officetul's shared infrastructure.

---

## 11. Related notes

- R06 §4–§9: unit costs, competitor prices, VAT.
- R10 §7: cross-border transfer.
- R12: OCR engine choice.
- R13 §3, §12: ClamAV and GuardDuty.
- R14 §2.3, §6, §9: Object Lock and fixity costs.
- R16 §6: search sizing.
- R17 §5–§6: storage layout and sizing.
- Section 02: pricing context.
- Questionnaire A12.

## 12. What this note changes elsewhere (for the owner to apply)

- **R13 Open question 4:** GuardDuty Malware Protection for S3 **is priced in Singapore** ($0.135/GB + $0.323 per 1,000 objects).
- **R14 §2.3:** AWS Manila Local Zone has no S3, so Object Lock is still not available in the Philippines from AWS.
- **Section 02 §3 and §9:** "≈ ₱48 per tenant" excludes fixed hosts and ops. See §3.3.
- **R06 FX:** use ₱63.088 (BSP, 20 Oct 2026) for new figures.

---

## 13. Sources (all accessed 2026-10-20)

| ID | Source | URL | Type |
|---|---|---|---|
| S1 | AWS EC2 on-demand Linux prices, Asia Pacific (Singapore) (price-list JSON, published 2026-10-15) | https://b0.p.awsstatic.com/pricing/2.0/meteredUnitMaps/ec2/USD/current/ec2-ondemand-without-sec-sel/Asia%20Pacific%20(Singapore)/Linux/index.json | Primary (price list) |
| S2 | AWS Price List API, AmazonEC2 ap-southeast-1 (EBS gp3, snapshots, NAT) | https://pricing.us-east-1.amazonaws.com/offers/v1.0/aws/AmazonEC2/current/ap-southeast-1/index.json | Primary (price list) |
| S3 | AWS Price List API, AmazonRDS ap-southeast-1 (published 2026-10-20) | https://pricing.us-east-1.amazonaws.com/offers/v1.0/aws/AmazonRDS/current/ap-southeast-1/index.json | Primary (price list) |
| S4 | Amazon RDS, "MySQL on Amazon RDS versions" | https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/MySQL.Concepts.VersionMgmt.html | Primary |
| S5 | Amazon RDS for MySQL pricing page (backup and Extended Support text) | https://aws.amazon.com/rds/mysql/pricing/ | Primary |
| S6 | AWS Price List API, AmazonLightsail ap-southeast-1 | https://pricing.us-east-1.amazonaws.com/offers/v1.0/aws/AmazonLightsail/current/ap-southeast-1/index.json | Primary (price list) |
| S7 | Lightsail, choosing a database engine | https://docs.aws.amazon.com/lightsail/latest/userguide/amazon-lightsail-choosing-a-database.html | Primary |
| S8 | Lightsail baseline CPU performance | https://docs.aws.amazon.com/lightsail/latest/userguide/baseline-cpu-performance.html | Primary |
| S9 | AWS Price List API, AmazonS3 ap-southeast-1 (published 2026-10-15) | https://pricing.us-east-1.amazonaws.com/offers/v1.0/aws/AmazonS3/current/ap-southeast-1/index.json | Primary (price list) |
| S10 | Amazon S3 FAQs (versioning billing) | https://aws.amazon.com/s3/faqs/ | Primary |
| S11 | AWS Price List API, AmazonVPC ap-southeast-1 (public IPv4) | https://pricing.us-east-1.amazonaws.com/offers/v1.0/aws/AmazonVPC/current/ap-southeast-1/index.json | Primary (price list) |
| S12 | AWS Price List API, AWSDataTransfer ap-southeast-1 (published 2026-09-16; copy from 2026-10-18) | https://pricing.us-east-1.amazonaws.com/offers/v1.0/aws/AWSDataTransfer/current/ap-southeast-1/index.json | Primary (price list) |
| S13 | AWS Price List API, awskms ap-southeast-1 | https://pricing.us-east-1.amazonaws.com/offers/v1.0/aws/awskms/current/ap-southeast-1/index.json | Primary (price list) |
| S14 | Amazon SES pricing page; AmazonSES ap-southeast-1 price list | https://aws.amazon.com/ses/pricing/ ; https://pricing.us-east-1.amazonaws.com/offers/v1.0/aws/AmazonSES/current/ap-southeast-1/index.json | Primary |
| S15 | AWS General Reference, SES endpoints (email receiving) | https://docs.aws.amazon.com/general/latest/gr/ses.html | Primary |
| S16 | AWS Price List API, AmazonGuardDuty ap-southeast-1 | https://pricing.us-east-1.amazonaws.com/offers/v1.0/aws/AmazonGuardDuty/current/ap-southeast-1/index.json | Primary (price list) |
| S17 | AWS Price List API, AWSBackup ap-southeast-1 | https://pricing.us-east-1.amazonaws.com/offers/v1.0/aws/AWSBackup/current/ap-southeast-1/index.json | Primary (price list) |
| S18 | AWS Price List API, AmazonElastiCache ap-southeast-1 | https://pricing.us-east-1.amazonaws.com/offers/v1.0/aws/AmazonElastiCache/current/ap-southeast-1/index.json | Primary (price list) |
| S19 | AWS Price List API, AmazonCloudWatch ap-southeast-1; CloudWatch pricing page (free tier) | https://pricing.us-east-1.amazonaws.com/offers/v1.0/aws/AmazonCloudWatch/current/ap-southeast-1/index.json ; https://aws.amazon.com/cloudwatch/pricing/ | Primary |
| S20 | AWS What's New, Local Zones Manila GA (posted 19 Jun 2023) | https://aws.amazon.com/about-aws/whats-new/2023/06/general-availability-aws-local-zones-manila | Primary |
| S21 | AWS Local Zones features by location | https://aws.amazon.com/about-aws/global-infrastructure/localzones/features/ | Primary |
| S22 | AWS price lists, Manila Local Zone (EC2 CSV, data transfer CSV) | https://pricing.us-east-1.amazonaws.com/offers/v1.0/aws/AmazonEC2/current/ap-southeast-1-mnl-1/index.csv ; https://pricing.us-east-1.amazonaws.com/offers/v1.0/aws/AWSDataTransfer/current/ap-southeast-1-mnl-1/index.csv | Primary (price list) |
| S23 | AWS Regions and Availability Zones | https://aws.amazon.com/about-aws/global-infrastructure/regions_az/ | Primary |
| S24 | Google Cloud regions and zones | https://docs.cloud.google.com/compute/docs/regions-zones | Primary |
| S25 | Azure regions list (updated 2026-10-13) | https://learn.microsoft.com/en-us/azure/reliability/regions-list | Primary |
| S26 | Alibaba Cloud regions and zones; global locations | https://www.alibabacloud.com/help/en/cloud-migration-guide-for-beginners/latest/regions-and-zones ; https://www.alibabacloud.com/en/global-locations | Primary |
| S27 | Alibaba OSS regions and endpoints | https://www.alibabacloud.com/help/en/oss/user-guide/regions-and-endpoints | Primary |
| S28 | Alibaba OSS pricing (Manila tables) | https://www.alibabacloud.com/en/product/oss/pricing | Primary |
| S29 | Alibaba OSS retention policies (WORM) | https://www.alibabacloud.com/help/en/oss/user-guide/oss-retention-policies | Primary |
| S30 | Huawei Cloud Philippines region launch (3 Dec 2024) | https://huaweicloud.com/intl/en-us/news/20241203162826823.html | Vendor |
| S31 | Huawei OBS regions and endpoints (updated 2026-10-10) | https://support.huaweicloud.com/usermanual-obs/obs_41_0003.html | Primary |
| S32 | Converge cloud page | https://globalbusiness.convergeict.com/our-solution/cloud | Vendor |
| S33 | Philstar, Converge sovereign cloud (18 Jun 2026, sponsored) | https://www.philstar.com/business/motoring/2026/06/18/2535897/beyond-connectivity-converge-unveils-country-sovereign-cloud-stack-built-whats-next | Secondary |
| S34 | Philstar, VITRO Sta. Rosa launch (24 Apr 2025); VITRO IPO (23 Jun 2026) | https://www.philstar.com/nation/2025/04/24/2437911/marcos-leads-launch-philippines-first-ai-ready-data-center ; https://www.philstar.com/other-sections/education-and-home/2026/06/23/2537068/pldt-taking-data-center-unit-public-files-p24-billion-ipo | Secondary |
| S35 | W.Media, STT GDC Philippines | https://w.media/stt-gdc-announces-plans-to-build-sustainable-carrier-neutral-data-center-in-the-philippines/ | Secondary |
| S36 | WonderNetwork pings Manila ↔ Singapore | https://wondernetwork.com/pings/manila/singapore ; https://wondernetwork.com/pings/singapore/manila | Secondary (third-party measurement) |
| S37 | WonderNetwork pings Manila → Hong Kong | https://wondernetwork.com/pings/manila/hongkong | Secondary |
| S38 | EO 119 (2026), Lawphil text | https://www.lawphil.net/executive/execord/eo2026/eo_119_2026.html | Secondary (legal-text mirror) |
| S39 | DigitalOcean regional availability | https://docs.digitalocean.com/platform/regional-availability/ | Primary |
| S40 | DigitalOcean Droplet pricing | https://www.digitalocean.com/pricing/droplets | Primary |
| S41 | DigitalOcean bandwidth billing | https://docs.digitalocean.com/platform/billing/bandwidth/ | Primary |
| S42 | DigitalOcean Managed Databases pricing; MySQL docs and pricing details | https://www.digitalocean.com/pricing/managed-databases ; https://docs.digitalocean.com/products/databases/mysql/ ; https://docs.digitalocean.com/products/databases/mysql/details/pricing/ | Primary |
| S43 | DigitalOcean Spaces pricing; S3 compatibility; Spaces API | https://www.digitalocean.com/pricing/spaces-object-storage ; https://docs.digitalocean.com/products/spaces/reference/s3-compatibility/ ; https://docs.digitalocean.com/reference/api/spaces/ | Primary |
| S44 | Linode API: regions, Linode types, transfer prices, region availability | https://api.linode.com/v4/regions ; https://api.linode.com/v4/linode/types ; https://api.linode.com/v4/network-transfer/prices ; https://api.linode.com/v4/regions/availability | Primary (API) |
| S45 | Linode API: database engines and types | https://api.linode.com/v4/databases/engines ; https://api.linode.com/v4/databases/types | Primary (API) |
| S46 | Akamai Object Storage pricing | https://www.akamai.com/cloud/pricing/object-storage | Primary |
| S47 | Akamai Object Storage lifecycle policies | https://techdocs.akamai.com/cloud-computing/docs/lifecycle-policies | Primary |
| S48 | Akamai Object Storage, Object Lock | https://techdocs.akamai.com/cloud-computing/docs/protect-data-with-object-lock | Primary |
| S49 | Akamai Object Storage limits; network transfer usage | https://techdocs.akamai.com/cloud-computing/docs/object-storage-product-limits ; https://techdocs.akamai.com/cloud-computing/docs/network-transfer-usage-and-costs | Primary |
| S50 | Vultr API: regions, plans, object-storage clusters; bandwidth overage | https://api.vultr.com/v2/regions ; https://api.vultr.com/v2/plans ; https://api.vultr.com/v2/object-storage/clusters ; https://docs.vultr.com/support/platform/billing/what-is-the-bandwidth-overage-rate | Primary (API) |
| S51 | Vultr object storage billing, S3 compatibility, buckets; MySQL versions | https://docs.vultr.com/support/platform/billing/how-is-object-storage-billed ; https://docs.vultr.com/products/cloud-storage/object-storage/s3-compatibility-matrix ; https://docs.vultr.com/products/storage/object-storage/management/manage-buckets ; https://docs.vultr.com/support/products/managed-databases/what-mysql-versions-are-available-in-vultr-managed-databases | Primary |
| S52 | Hetzner Cloud locations | https://docs.hetzner.com/cloud/general/locations/ | Primary |
| S53 | Hetzner Cloud pricing (regular performance, general purpose); billing FAQ | https://www.hetzner.com/cloud/regular-performance/ ; https://www.hetzner.com/cloud/general-purpose/ ; https://docs.hetzner.com/cloud/billing/faq/ | Primary |
| S54 | Hetzner Object Storage; overview; supported actions | https://www.hetzner.com/storage/object-storage/ ; https://docs.hetzner.com/storage/object-storage/overview/ ; https://docs.hetzner.com/storage/object-storage/supported-actions/ | Primary |
| S55 | Laravel Forge pricing | https://laravel.com/forge/pricing | Primary |
| S56 | Laravel Cloud pricing (regions incl. Asia Pacific Singapore) | https://laravel.com/cloud/pricing | Primary |
| S57 | Ploi pricing | https://ploi.io/pricing | Primary |
| S58 | Postmark pricing; "How does monthly pricing work"; EU privacy page | https://postmarkapp.com/pricing ; https://postmarkapp.com/support/article/1107-how-does-monthly-pricing-work ; https://postmarkapp.com/eu-privacy | Vendor |
| S59 | Mailgun pricing | https://www.mailgun.com/pricing/ | Vendor |
| S60 | BSP daily PHP per USD rate (updated 20 Oct 2026: 63.088) | https://www.bsp.gov.ph/statistics/external/day99_data.aspx | Primary |
| S61 | Gotenberg installation; configuration | https://gotenberg.dev/docs/getting-started/installation ; https://gotenberg.dev/docs/configuration | Primary (project) |
| S62 | Apache Tika server wiki (incl. Tika 2.x forking) | https://cwiki.apache.org/confluence/display/TIKA/TikaServer ; https://cwiki.apache.org/confluence/display/TIKA/TikaServer+in+Tika+2.x | Primary (project) |
| S63 | OCRmyPDF performance; advanced | https://ocrmypdf.readthedocs.io/en/latest/performance.html ; https://ocrmypdf.readthedocs.io/en/latest/advanced.html | Primary (project) |
| S64 | Tesseract FAQ (threads, `OMP_THREAD_LIMIT`) | https://tesseract-ocr.github.io/tessdoc/FAQ.html | Primary (project) |
| S65 | ClamAV Docker docs (memory) | https://docs.clamav.net/manual/Installing/Docker.html | Primary |
| S66 | ClamAV database files, HTTP HEAD sizes measured 2026-10-20 | https://database.clamav.net/main.cvd ; https://database.clamav.net/daily.cvd ; https://database.clamav.net/bytecode.cvd | Primary (measurement) |
| S67 | Sentry pricing | https://sentry.io/pricing/ | Vendor |
| S68 | Better Stack pricing | https://betterstack.com/pricing | Vendor |
| S69 | UptimeRobot pricing | https://uptimerobot.com/pricing/ | Vendor |
| S70 | Own OCR micro-benchmark: `jbarlow83/ocrmypdf-alpine` (OCRmyPDF 17.13.1, Tesseract 5.5.2), `fil` from tessdata_fast, 10 synthetic 300-dpi pages, Apple M1, 2026-10-20 | https://hub.docker.com/r/jbarlow83/ocrmypdf-alpine ; https://raw.githubusercontent.com/tesseract-ocr/tessdata_fast/main/fil.traineddata | Internal (measurement) |
| S71 | Internal notes: R06, R10, R12, R13, R14, R16, R17; Section 02; legal and tax questionnaire A12 | research/, sections/02-market-and-competitor-research.md, questionnaires/legal-and-tax-questions.md | Internal |
