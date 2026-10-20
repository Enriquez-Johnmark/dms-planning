# 07 — OCR and Document Understanding

## 1. Purpose

Choose **how scanned documents become searchable**, at what cost and with what privacy implications, and **how to test** that it's good enough before the pilot.

## 2. Status

**Owner agreed (Round 9, D-049, D-050), 2026-10-20.** Research: [R12](../research/R12-ocr-engines-and-benchmark.md) (2026-10-20).

## 3. Executive Summary

- **OCR reads text; it does not understand documents.** In the MVP, OCR only makes scans **searchable**. Reading fields (supplier, amount) is AI extraction, a later, metered feature (D-018 / R06). OCR text is a Derived file and never replaces the original.
- **Recommended MVP engine: OCRmyPDF + Tesseract 5, self-hosted.** It is the only option that combines English, Filipino (`fil`) and Cebuano (`ceb`) models, refuses to modify signed PDFs by default, writes separate searchable outputs, and needs **no outside company to see client documents**. Its cost is server time only (≈₱1–2 per 1,000 pages, Estimate). *(Verified fact for features; Recommendation)*
- **Its weak spot is noisy pages.** In a peer-reviewed test, Tesseract kept up with Google and AWS on clean pages, but on noisy pages its word error rate was roughly double theirs (23% vs 8–12%). **Phone photos and faded thermal receipts are the likely problem.** *(Independent benchmark; old version, English only)*
- **Fallbacks, only if the test shows a need:** first **PaddleOCR** (self-hosted; straightens curved phone photos; Tagalog but no Cebuano); then **Azure Read** as an **opt-in cloud option per company** (Filipino and Cebuano; inputs deleted after 24 hours), disclosed as a sub-processor (D-032). **AWS Textract is excluded** because it doesn't read Filipino.
- **No benchmark exists for Filipino business documents**, so we test our own: a **240-page set**, scored mainly on "does search find the document". **You set the pass bar by looking at examples**; no numbers are invented in advance.

## 4. Business Context

Clients will search for a TIN, an invoice number, a supplier name or a Filipino word like "kasunduan". If OCR misses those, the app fails at its main promise. "Documents included, no per-page fee" (D-018, pending) only works if OCR is cheap, which favours self-hosting.

## 5. Verified Research Findings

| Engine | Filipino | Cebuano | Runs on our servers | Privacy | Role |
|---|---|---|---|---|---|
| Tesseract 5 + OCRmyPDF | Yes | Yes | Yes (CPU) | No sub-processor | **MVP** |
| PaddleOCR PP-OCRv5 | Tagalog | No | Yes (CPU) | No sub-processor | Fallback A (phone photos) |
| Azure Read | Yes (printed) | Yes (printed) | No (cloud) | Sub-processor; 24-hour retention; training terms still to confirm | Fallback B (opt-in) |
| Google Vision / Document AI | Yes | Experimental | No | Sub-processor; limited Singapore support | Alternative B |
| AWS Textract | **No** | **No** | No | Training opt-out needed | **Excluded** |
| Surya | ? | ? | Slow on CPU | — | **Excluded** (weights need a licence above US$5M revenue) |

Source: R12 §2, §8.

## 6. Technical Analysis

**MVP pipeline** *(Recommendation, R12 §7)*:
- **Unsigned scans:** OCRmyPDF in skip mode (only pages without text), with deskew, rotate and clean, languages `eng+fil`, plus `ceb` if the company opts in. Outputs a **sidecar text file** for search and a **searchable PDF/A** as a Derived file.
- **Signed scans:** page images → Tesseract → text only. No PDF is written.
- **Images and phone photos:** same engine. Fallback A if the test fails.
- Store per-page confidence. Show "**OCR quality low**" when confidence is poor, so users know to add details by hand.
- Runs in a dedicated **OCR queue** with time and memory limits (Section 12).

**Benchmark** *(Recommendation, R12 §6)*:

| Set | Pages | Source (lawful) |
|---|---|---|
| Thermal receipts | 30 | Own company's receipts (consent; personal data redacted); public receipt datasets |
| BIR forms, filled | 30 | Blank forms from bir.gov.ph filled with made-up data |
| Typed contracts and letters | 30 | Made-up or own company documents |
| Filipino-language letters and notices | 30 | Public government issuances |
| Cebuano | 10 | Public LGU notices (source to find) |
| Court documents | 30 | Public Supreme Court decisions, printed and re-scanned |
| Drawing title blocks | 20 | Made-up or public plans |
| Phone photos of the above | 50 | Re-photographed with 2–3 mid-range Android phones |
| Signed / digital PDFs | 10 | Self-made; checks they're left untouched |

**No client documents are used** (D-009).

**Main score:** keyword-hit rate, i.e. "does Officetul search find this document by its TIN, number, supplier or a Filipino word?". Character and word error rates, throughput, memory and cost per 1,000 pages are also measured.

**Pass bar:** set in a ~1-hour session where you look at 5–10 example pages with live search and mark "good enough to find" or not. There is a separate bar per document set.

## 7. Alternatives and Trade-offs

| Option | Cost | Privacy | Quality on noisy pages |
|---|---|---|---|
| **Self-hosted Tesseract** (MVP) | Lowest | Best (no outside party) | Weakest |
| Self-hosted PaddleOCR | Low (more CPU) | Best | Better on photos (vendor claim) |
| Cloud OCR, opt-in | ~US$1.50 per 1,000 pages | Sub-processor disclosure | Best |

## 8. Recommendation

1. **MVP OCR = OCRmyPDF + Tesseract 5, self-hosted, `eng+fil`, optional `ceb`.** No cloud OCR in the MVP.
2. **Run the 240-page benchmark before the external-client pilot.** It can run alongside the own-company pilot. You set the bar by example.
3. **Turn on fallbacks only if the benchmark says so:** PaddleOCR first, then opt-in Azure Read.
4. **Show a "low OCR quality" flag**; never hide uncertainty.

## 9. Cost and Operational Implications

- OCR is the heaviest processing job. It needs dedicated worker CPU, sized in Phase 10 from benchmark throughput.
- Benchmark effort: assembling samples and double-checked reference text for ~240 pages takes days of staff time *(Estimate)*.

## 10. Risks and Limitations

| ID | Risk | Mitigation |
|---|---|---|
| RK-14 | **OCR quality too low** on phone photos and receipts | Benchmark; fallbacks; low-quality flag |
| RK-33 | **OCR backlog** delays search after bulk scanning | Separate queue; progress status; capacity sizing in Phase 10 |
| RK-34 | **Cloud fallback leaks data** | Opt-in per company; sub-processor disclosure; no training use |

## 11–12. Questions and Decisions

Grilling Round 9, Q6–Q7.

## 13. Confirmed Decisions

| ID | Decision |
|---|---|
| D-049 | Self-hosted OCRmyPDF + Tesseract; fallbacks only if needed |
| D-050 | 240-page benchmark before the first external client; owner sets the bar |

## 14. Open Questions

R12 §10(d): no Filipino or Cebuano benchmark exists; whether `eng+fil` hurts English accuracy; whether OCRmyPDF refuses signed PDFs in text-only mode; Azure training terms and Southeast Asia availability; Tesseract throughput on our servers; whether internal use of government works is lawful (lawyer).

## 15. Dependencies

Depends on D-009, D-018, D-031, D-032. Feeds Sections 13 (search), 14 (server sizing), 18 (cost).

## 16. Sources

R12 sources (accessed 2026-10-20).

## 17. Next Steps

- [x] MVP OCR engine agreed (D-049)
- [x] Benchmark plan agreed (D-050); calibration session not yet scheduled
