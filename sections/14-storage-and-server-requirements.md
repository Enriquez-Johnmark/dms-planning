# 14 — Storage and Server Requirements

## 1. Purpose

Decide **where Officetul Documents runs**, on what servers, and how much storage it needs as it grows. Costs are in [Section 18](18-cost-estimates-and-operating-model.md); backups are in [Section 15](15-backup-retention-and-disaster-recovery.md).

## 2. Status

**Owner agreed (Round 13, D-076 to D-078), 2026-10-21.** Research: [R18](../research/R18-hosting-and-cost-model.md), with sizing from [R16](../research/R16-search-architecture.md) and [R17](../research/R17-data-model-and-erp-integration.md). Where Officetul runs today is still unknown ([ERP questionnaire](../questionnaires/erp-technical-questionnaire.md) Q12).

## 3. Executive Summary

- **Singapore, with Officetul's provider.** If that provider can't keep old file versions, apply lifecycle rules and lock files, Documents' files go to **AWS Singapore**. Linode Singapore is the cheaper option for the pilot and year 1, if its file lock is confirmed. *(D-076)*
- **No Philippine hosting for the first version.**
  - AWS's Manila zone offers servers only, with no file storage and no managed database.
  - No local provider publishes the prices and features we need.
  - We revisit only if the lawyer or a paying client requires it. *(Verified fact for AWS; D-076)*
- **One dedicated 16 GB processing server** for ClamAV, LibreOffice, Tika and OCR, separate from Officetul's web servers. *(D-077)*
- **Managed MySQL 8.4.** If Officetul is on Amazon's MySQL 8.0, upgrade now, because old-version support already costs about ₱22,100 a month extra. *(D-078)*
- **Inbound email through Amazon SES Singapore** with any host. Postmark is US-hosted. *(Recommendation, R18 §7)*

## 4. Business Context

Clients care that their files are safe, fast to open from Manila, and handled under a clear data processing agreement. For a 1–2 person team, every server we run is work: patching, alerts, restores. So the design uses managed services where they remove ops work, and keeps the one server we must run ourselves (the processing server) isolated and simple.

## 5. Verified Research Findings

| Finding | Source |
|---|---|
| AWS has no Region in the Philippines; the Manila Local Zone offers compute only, with no S3 or RDS | R18 §2, R19 §3 |
| AWS S3 Singapore supports versioning, lifecycle rules and compliance-mode Object Lock; SES inbound email is available in Singapore | R18 §2, R14 §2.3 |
| Akamai/Linode, DigitalOcean and Vultr have Singapore locations; Linode's compliance-mode lock in Singapore is unconfirmed; DigitalOcean Spaces' lifecycle support for old versions is unclear | R18 §2, R14 §2.3 |
| Amazon RDS MySQL 8.0 Extended Support costs $0.12 per vCPU-hour (since 1 August 2026) | R18 §6.2 |
| ClamAV needs about 3–4 GB of RAM, plus a reload spike | R13 §3 |
| OCR takes about 9 CPU-seconds per page (laptop test of 10 generated pages; to be replaced by the D-050 benchmark) | R18 §1 |

## 6. Technical Analysis

### 6.1 Components *(R18 §1)*

| Component | Where | Shared with Officetul? |
|---|---|---|
| Web servers and light queue jobs (Laravel) | Officetul's app servers | Yes |
| File-pipeline workers (validate, scan, convert, OCR, fixity), with their own settings, DB user and storage role (D-099) | Processing server, outside the tool containers | No |
| Processing server (ClamAV, Gotenberg/LibreOffice, Tika, OCRmyPDF/Tesseract, libvips), each in its own container with no network (D-051) | One dedicated 16 GB server | No |
| MySQL 8.4 (managed) | Officetul's database | Yes (Documents adds tables) |
| Object storage: quarantine, originals (versioned), derived, exports, digests (locked) (D-075) | AWS S3 Singapore or the provider's equivalent | No |
| Inbound email (scan-to-email) | Amazon SES Singapore | No |
| Backups | Separate backup account, second region (Section 15) | — |

### 6.2 Storage growth *(Estimate; R17 §6.1, R19 §9.1)*

| Size | Companies × Documents | Originals | Derived | Database share |
|---|---|---|---|---|
| Pilot | 2 × 5,000 | ≈ 19 GB | ≈ 8 GB | ≈ 5 GB |
| Year 1 | 20 × 5,000 | ≈ 189 GB | ≈ 80 GB | ≈ 25–35 GB |
| Growth | 100 × 50,000 | ≈ 9.5 TB | ≈ 4 TB | ≈ 300–400 GB |

At growth the search text passes the 20 GB trigger in D-064, so a Meilisearch server is likely to be added then.

### 6.3 Server sizes *(Estimate; R18 §3)*

| Size | Processing server | Database |
|---|---|---|
| Pilot / Year 1 | One 16 GB server (4 vCPU) | Officetul's existing managed MySQL, plus about 5–35 GB |
| Growth | Intake server + 8-vCPU OCR server | A step up to about 32 GB of RAM, for a 16–32 GB memory cache |

## 7. Alternatives and Trade-offs

| Question | Options | Choice |
|---|---|---|
| Region | **Singapore** / Philippines / Tokyo | Singapore: about 30 ms from Manila, full services; Philippines lacks managed storage and databases |
| Provider | **Officetul's provider** / **AWS** / Linode | Officetul's provider if it meets the storage needs; else AWS; Linode if cheaper and its lock is confirmed |
| Processing | Run on the web servers / **dedicated server** / serverless | Dedicated: isolation (D-051), predictable cost |
| Database | **Managed** / self-managed | Managed: backups and patching handled |

## 8. Recommendation

Adopt §6. Once questionnaire Q12 is answered, confirm the provider. If it is not AWS, check Object Lock (compliance mode), version expiry and lifecycle rules before choosing it for files.

## 9. Cost and Operational Implications

See [Section 18](18-cost-estimates-and-operating-model.md). Roughly, the cost of adding Documents is ₱8,500 a month for the pilot and ₱13,800 a month for year 1 on AWS (ex VAT), most of it the processing server.

## 10. Risks and Limitations

| ID | Risk | Mitigation |
|---|---|---|
| RK-56 | **Officetul's provider can't meet storage needs**, so Documents spans two providers | Files and processing on AWS; app stays put; one data processing addendum lists both |
| RK-57 | **Container images don't run on ARM servers**, about 24% more cost | Check before choosing server types |
| RK-58 | **A client or the lawyer requires Philippine hosting** | Revisit (D-076); Alibaba and Huawei Manila are the candidates to research |
| RK-59 | **OCR is slower than the laptop estimate** | D-050 benchmark on the real server class |

## 11–12. Questions and Decisions

Grilling Round 13, Q1–Q3.

## 13. Confirmed Decisions

| ID | Decision |
|---|---|
| D-076 | Singapore with Officetul's provider; AWS if storage needs aren't met; no PH hosting in the MVP |
| D-077 | One dedicated 16 GB processing server |
| D-078 | Managed MySQL 8.4; upgrade now if on RDS 8.0 |

## 14. Open Questions

- Where Officetul runs today and its database version (Q12, OQ-028).
- Linode Singapore compliance lock; DigitalOcean lifecycle support.
- ARM compatibility of the tool images.
- Real OCR speed (D-050).
- Lawyer A12: any reason to prefer Philippine hosting?

## 15. Dependencies

Depends on D-045 to D-051, D-061, D-064, D-073 to D-075. Feeds Sections 15, 18 and 20.

## 16. Sources

R18 (accessed 2026-10-20), especially the AWS Price List API, the Akamai/Linode pricing pages and the Amazon RDS Extended Support pricing; R13, R14, R16, R17 and R19.

## 17. Next Steps

- [x] Region and provider rule agreed (D-076)
- [x] Processing server agreed (D-077)
- [x] Database agreed (D-078)
- [ ] Questionnaire Q12 answered and provider confirmed
- [ ] Image ARM check and OCR benchmark on the server class
