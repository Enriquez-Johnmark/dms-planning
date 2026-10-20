# 18 — Cost Estimates and Operating Model

## 1. Purpose

Estimate **what it costs to run** Officetul Documents and **who runs it**, and give the senior partner the numbers needed to decide pricing (D-018). This section **does not set a price**.

## 2. Status

**Owner agreed (Round 13, D-086 to D-088), 2026-10-21.** Research: [R18](../research/R18-hosting-and-cost-model.md), with backup costs from [R19](../research/R19-backup-retention-and-disaster-recovery.md). **Pricing: senior partner decision pending** ([meeting guide 18](../meeting-guides/18-costs-and-pricing.md)). All figures are **Estimates** at ₱63.088 per US dollar (BSP, 20 October 2026). They exclude 12% VAT on foreign cloud bills unless stated.

## 3. Executive Summary

- **The earlier "about ₱48 per company" figure was too low.** It counted only storage, OCR and email. The **fixed processing server and ops time dominate** until there are about 50 companies. *(Estimate; replaces Section 02 §3)*
- **Monthly cost of adding Documents** (Officetul's own hosting excluded):

  | Size | AWS | Linode | Per company |
  |---|---|---|---|
  | Pilot (2 companies) | ≈ ₱8,500 | ≈ ₱6,400 | — |
  | Year 1 (20 companies) | ≈ ₱13,800 + ≈ ₱2,200 backups | ≈ ₱8,500 + backups | ≈ ₱430–800 |
  | Growth (100 companies) | ≈ ₱93,300 | ≈ ₱99,700 | ≈ ₱930–1,000 |

- **Including ops time,** one company costs about **₱900–1,270 a month**, using a placeholder ops rate of ₱600 an hour.
- **Fixed floor:** about **₱13,000–21,000 a month**, whether there are 2 companies or 20.
- **Plans limit storage and OCR pages, not users.** Users cost almost nothing; storage grows forever. *(D-086)*
- **Ops: 12–16 hours a month in year 1,** with a named on-call person and a named backup. *(D-087)*

## 4. Business Context

The senior partner sells Officetul as a one-stop ERP (D-002). Documents must either fit inside the subscription margin or be priced as an add-on without hurting the pitch. Competitors range from "included" (Xero, Zoho storage) to about ₱565–707 per user (Odoo Standard) and ₱1,590 (Dext Business). Juan sells its whole accounting product for ₱2,000 per organisation.

## 5. Verified Research Findings

| Finding | Source |
|---|---|
| AWS Singapore prices from the AWS Price List API (S3, EC2, RDS, SES, data transfer) | R18 §3, sources |
| Akamai/Linode published prices for compute, object storage and managed MySQL (up to 8 GB RAM) | R18 §3.2 |
| RDS MySQL 8.0 Extended Support: $0.12 per vCPU-hour | R18 §6.2 |
| 12% VAT applies to foreign cloud services; whether it is creditable is unconfirmed | R18 §3, R06 §5 |
| Competitor anchors: Odoo Standard US$8.95–11.20 per user; Dext Business US$25.21; Juan Essentials ₱2,000 | R18 §5.1, R06 |

## 6. Technical Analysis

### 6.1 Cost drivers *(R18 §4)*

| Driver | Unit cost (Estimate) | Matters when |
|---|---|---|
| Fixed processing server, monitoring | ≈ ₱8,300 (pilot) to ₱11,600 (year 1) a month (AWS) | Always; dominates small scale |
| Ops time | 12–16 h a month (year 1) at a rate to be set | Always |
| Stored Documents (storage + backup + rows) | ≈ ₱6.5 per 1,000 Documents a month; ≈ ₱99 per extra 50 GB incl. VAT | Grows every month |
| Database step at growth | ≈ ₱38,000 a month (AWS, to a 32 GB server) | Growth |
| OCR pages | ≈ ₱7 per 1,000 pages | Minor; bursts need capacity, not money |
| Inbound email | ≈ ₱40 per 1,000 emails | Minor |
| Users | ≈ nothing | — |

### 6.2 What plans should limit *(D-086)*

| Limit | Example (to be set by the senior partner) |
|---|---|
| Storage per company, with paid top-up blocks | e.g. 25 GB included (≈ 8,000 Documents) |
| OCR pages per month, fair use | e.g. 3,000 pages |
| Back-file scanning | Quoted as a separate project, processed overnight |
| Inbound email | About 35 MB per message (D-045); monthly fair use |

### 6.3 Illustrative margins *(Estimate; not a price proposal; R18 §5.2)*

Full cost per company is about ₱900–1,270 a month, including VAT and ops time at ₱600 an hour.

| Flat add-on per company | Companies needed to break even (AWS, year-1 shape) |
|---|---|
| ₱500 | ≈ 57 |
| ₱1,000 | ≈ 24 |
| ₱1,500 | ≈ 16 |
| ₱2,000 | ≈ 11 |

**Pricing models:**
- **Included with a storage quota:** keeps the pitch, but the cost must fit Officetul's margin.
- **Flat add-on:** needs ₱1,500–2,000 to cover costs at small scale.
- **Per user:** mismatched with costs, because a 3-user firm with 100 GB would be under-priced.
- **Hybrid:** an included base, plus storage blocks and back-file packs.

### 6.4 Operating model *(D-087; R18 §6)*

| Task | Hours a month (year 1 / growth) |
|---|---|
| Patching the processing server and tool images; urgent fixes | 3 / 5 |
| ClamAV signatures, held files, false positives | 1.5 / 4 |
| Alerts: queues, fixity, digests, disk, errors | 3 / 6 |
| Restore drills (D-085) | 1.5 / 2 |
| Security log review (D-057, D-060) | 2 / 3 |
| Incident duty and tabletop drills (D-032) | 1 / 2 |
| Capacity, cost review, upgrades | 1 / 3 |
| **Total** | **≈ 12–16 / 20–30** |

Managed services reduce this. Managed MySQL handles backups and patching, and SES handles inbound email. Laravel Forge or Ploi (about $10–39 a month) can handle server setup and deploys.

## 7. Alternatives and Trade-offs

| Question | Options | Choice |
|---|---|---|
| What to limit | Users / **storage and OCR pages** | Storage and pages: they match the costs |
| Provider for cost | AWS / Linode | Linode saves about ₱5,300 a month at year 1; the two converge at growth (D-076 decides by storage features first) |
| Ops | Self-run / **managed where it removes work** | Managed database, managed email; one self-run processing server |

## 8. Recommendation

Bring this cost model and the pricing options to the senior partner (D-088). Until they decide, use these figures for planning, and do not quote prices to clients.

## 9. Cost and Operational Implications

This section is the cost model. The key number for the senior partner is the **fixed floor of about ₱13,000–21,000 a month**, which any pricing must cover.

## 10. Risks and Limitations

| ID | Risk | Mitigation |
|---|---|---|
| RK-64 | **Prices set below the fixed floor**, so Documents loses money until about 24+ companies | Senior partner sees break-even figures (meeting guide 18) |
| RK-65 | **Storage tail:** no automatic purge (D-041), so heavy clients grow costs forever | Storage quotas and top-up blocks (D-086) |
| RK-66 | **Estimates are wrong** (OCR speed, ARM images, discounts, VAT credit) | Re-run the model after the pilot and the OCR benchmark |
| RK-67 | **Ops time is underestimated**, or the real hourly rate is higher | Track actual hours from the pilot |

## 11–12. Questions and Decisions

Grilling Round 13, Q11–Q13. **Pricing:** senior partner (OQ-020).

## 13. Confirmed Decisions

| ID | Decision |
|---|---|
| D-086 | Plans limit storage and OCR pages, not users |
| D-087 | Ops budget and named on-call people |
| D-088 | Cost model and options go to the senior partner |

## 14. Open Questions

- **OQ-020:** pricing model and level, Officetul's current price and target margin (senior partner).
- The real ops hourly rate (owner).
- Whether the 12% VAT on cloud bills is creditable (tax adviser).
- Savings Plan or reserved prices; ARM compatibility; Linode database prices above 8 GB.

## 15. Dependencies

Depends on D-018, D-076 to D-085. Feeds Section 17 (MVP scope), Section 21 (roadmap) and the senior partner meeting.

## 16. Sources

R18 (accessed 2026-10-20) and R19; competitor prices from R06.

## 17. Next Steps

- [x] Cost model agreed as the basis for pricing (D-088)
- [x] Plan limits and ops model agreed (D-086, D-087)
- [ ] Senior partner pricing decision (meeting guide 18)
- [ ] Re-run the model with pilot data
