# R12 — OCR engines for search-grade full text, and a benchmark design

- **Status:** First pass, complete. Research only. No code. Nothing here is approved; every **Recommendation** needs the owner's decision.
- **Researched:** 2026-10-20. **All sources accessed 2026-10-20** unless marked otherwise.
- **Question:** Which OCR engine should make scanned paper searchable by full text in the Officetul Documents app (English plus some Filipino/Tagalog and Cebuano; MFP scans and phone photos), and how should a Phase 7 benchmark test that choice?
- **Scope:** Search-grade full-text OCR only. AI field extraction (supplier, amount) is a later, metered feature and is out of scope.
- **Fixed constraints (agreed earlier):** OCR output is a **derived file**, the original is never modified, and digitally signed PDFs are never rewritten.
- **Labels:** **VF** = Verified fact (read in the primary source). **VC** = Vendor claim (accuracy or speed stated by the maker). **IB** = Independent benchmark (peer-reviewed or third-party). **Est** = Estimate. **OQ** = Open question. **Sec** = Secondary (forum, mirror or search excerpt only). Source IDs `[S#]` point to §11.

**Access caveats.** The OpenCV docs and paperless-ngx docs returned HTTP 403. The Supreme Court E-Library failed TLS verification. The BIR forms page loaded, but its form tables were empty when fetched. Licence and release data for GitHub projects were read through the GitHub REST API on 2026-10-20.

---

## 1. Short answer

1. **MVP: OCRmyPDF + Tesseract 5 on CPU queue workers.** Only this option combines Filipino and Cebuano models [S3], refusal to modify signed PDFs by default [S9], searchable PDF plus sidecar text as separate outputs [S9], and a plain CLI for a Laravel job [S43]. **Recommendation**
2. **Tesseract is weakest on noisy scans.** In a peer-reviewed test, Document AI and Textract beat Tesseract 4.1.1, and the gap grew with noise; Tesseract kept up only on clean English pages [S42]. **IB.** Phone photos and thermal receipts are the likely weak spot. **Est**
3. **Fallback candidates:** Azure Read (Filipino and Cebuano printed) [S37]; Google Vision (Filipino and Tagalog supported, Cebuano experimental) [S25]. **AWS Textract is excluded**: English, Spanish, German, Italian, French and Portuguese only [S32][S33]. Surya's weights need a commercial licence above US$5M funding or revenue [S23]. **VF**
4. **No benchmark of OCR on Filipino or Cebuano business documents was found**, so Phase 7 needs Officetul's own test set. **OQ**

---

## 2. Engine-by-engine facts

### 2.1 Self-hosted engines

| | **Tesseract 5** | **OCRmyPDF** (wrapper) | **PaddleOCR 3.x (PP-OCRv5)** | **EasyOCR** | **docTR** | **Surya** | **Kraken** |
|---|---|---|---|---|---|---|---|
| Licence | Apache-2.0. Leptonica is BSD-2 [S1]. Model repos are Apache-2.0 [S6]. **VF** | MPL-2.0. Non-core code is MIT [S7]. **VF** | Apache-2.0 [S17]. **VF** | Apache-2.0 [S19]. **VF** | Apache-2.0 [S20]. **VF** | Code is Apache-2.0. Weights use a modified OpenRAIL-M licence: free under US$5M funding or revenue, commercial licence above that [S23]. **VF** | Apache-2.0 [S24]. **VF** |
| Last release | 5.5.3, 2026-07-24 [S2]. **VF** | v17.13.0, 2026-09-28 [S8]. **VF** | v3.7.0, 2026-06-11 [S17]. **VF** | v1.7.2, **2024-09-24**. Issues auto-close after 6 months "due to limited resources" [S19]. **VF** | v1.1.0, 2026-08-21 [S20]. **VF** | v0.22.1, 2026-07-20 [S23]. **VF** | 7.1, 2026-08-05 [S24]. **VF** |
| English | Yes. **VF** | Uses Tesseract | Yes. **VF** | Yes [S18]. **VF** | Yes [S21]. **VF** | Yes. 90+ languages [S23]. **VC** | Model-dependent. Built for historical and non-Latin material [S24]. **VF** |
| Filipino / Tagalog | `fil` ("Filipino (old – Tagalog)") is in tessdata, _best and _fast. `tgl` exists only up to 4.00 [S3]. **VF** | `-l eng+fil` [S9]. **VF** | Tagalog `tl` is in `latin_PP-OCRv5_mobile_rec` [S15]. **VF** | Tagalog `tl` [S18]. **VF** | `tagalog` vocab (includes ñ and ₱) [S21]. A pretrained Tagalog model is **OQ** | Not checked for Tagalog. **OQ** | No modern PH model known. **OQ** |
| Cebuano | `ceb` in all three sets [S3]. **VF** | `-l ceb` | Not listed [S15]. **VF** | Not listed [S18]. **VF** | No vocab [S21]. **VF** | **OQ** | **OQ** |
| Handwriting | Not documented [S1]. Treat as unsupported. **Est** | — | The server rec model handles "handwriting". The English mobile model is improved for handwritten English [S16]. **VC** | Listed as "coming next", so not yet supported [S19]. **VF** | Not documented. **OQ** | A handwriting sample is shown, but no figures are given [S23]. **VC** | Trainable for handwriting. **VF** |
| Layout / reading order | Page segmentation modes (`--psm`) [S5]. No table structure documented. **VF** | Keeps Tesseract layout | Not assessed in R12. **OQ** | Text boxes only. No layout. **VF** | Optional `detect_layout` gives title, text, table and header regions [S20]. **VF** | Layout, reading order and table HTML [S23]. **VC** | Trainable layout and reading order [S24]. **VF** |
| Rotation / skew | OSD model (`osd`) [S4]. **VF** | `--rotate-pages` and `--deskew` [S9]. **VF** | Document orientation classifier (0/90/180/270). **UVDoc** unwarping for curved or photographed pages [S16]. **VF** | **OQ** | Rotated boxes and `straighten_pages` [S20]. **VF** | **OQ** | Baseline segmenter [S24]. **VF** |
| CPU / GPU | CPU (no GPU path documented [S1]). **Est** | CPU. Uses all cores (`--jobs`) [S7]. **VF** | CPU or GPU [S16]. **VF** | CPU allowed with `gpu=False` [S19]. **VF** | PyTorch. Docker images are CUDA-based [S20]. CPU speed for full pages is not published. **OQ** | GPU via vLLM. CPU or Apple Silicon via llama.cpp [S23]. **VF** | CPU or GPU. **OQ** |
| Published speed | 2 s/page English, 17 s/page Arabic (Tesseract 4.1.1, 12-core 4.3 GHz desktop) [S42]. **IB** | Default timeout 180 s/page [S10]. **VF** | CPU (Xeon Gold 6271C), per model, excluding pre/post-processing: server det 383 ms, mobile det 58 ms, server rec 31 ms, mobile rec 21 ms [S16]. **VC** | Not published | Only per-stage timings (i7-11800H) [S22]. **VC** | 5.35 pages/s on an RTX 5090. **0.108 pages/s** on Apple Silicon via llama.cpp [S23]. **VC** | Not published |
| Memory | **OQ** (measure in Phase 7) | About 15 bytes per pixel of the largest page, per job. A 300-dpi letter page is about 8.4 MP [S10][S11]. **VF** | **OQ** | **OQ** | **OQ** | Not stated. Benchmark used a 32 GB GPU [S23]. **VF** | **OQ** |
| Model size | English: fast 4.1 MB, best 15.4 MB. Filipino: fast 1.8 MB, best 9.0 MB. Cebuano: best 3.5 MB. `script/Latin` best 101 MB [S6]. **VF** | — | Server det 84.3 MB, mobile det 4.7 MB, server rec 81 MB, mobile rec 16 MB [S16]. **VF** | **OQ** | **OQ** | 0.65B parameters [S23]. **VC** | **OQ** |
| Output | txt, hOCR, PDF, text-only PDF, TSV, **ALTO**, PAGE [S1]. **VF** | PDF/A with text layer, `--sidecar` txt [S7][S9]. hOCR renderer is deprecated [S10]. **VF** | JSON with `rec_scores` [S16]. **VF** | Box, text and confidence [S19]. **VF** | JSON export [S20]. **VF** | Table HTML, LaTeX math [S23]. **VF** | ALTO, PageXML, hOCR [S24]. **VF** |
| Laravel path | CLI through a queued job (Laravel `Process`, default timeout 60 s) [S43]. MIT PHP wrapper available [S52]. **VF** | Same, CLI per document. **Recommendation** | Python microservice. The HTTP `/ocr` service is built in [S16]. **VF** | Python microservice, or an OCRmyPDF plugin [S53]. **VF** | FastAPI template [S20]. **VF** | Docker plus vLLM or llama.cpp server [S23]. **VF** | Python CLI |

**Reading (Est):** PaddleOCR is the credible self-hosted challenger (Apache-2.0, active, Tagalog, orientation and unwarping models for photos; no Cebuano). EasyOCR is stale. docTR's Tagalog model is unconfirmed. Kraken targets historical material. Surya fails on licence and CPU speed for an "included, no per-page fee" product without GPUs.

### 2.2 Cloud engines (for the fallback slot)

| | **Google Cloud Vision** (`DOCUMENT_TEXT_DETECTION`) | **Google Document AI** Enterprise OCR | **AWS Textract** `DetectDocumentText` | **Azure Document Intelligence** Read (`prebuilt-read`, v4.0) |
|---|---|---|---|---|
| Filipino / Tagalog | `fil` and `tl` **supported** [S25]. **VF** | Not stated on the OCR page [S29]. **OQ** | **Not supported.** EN/ES/DE/IT/FR/PT only [S32][S33]. **VF** | `fil` printed text supported [S37]. **VF** |
| Cebuano | `ceb` **experimental** [S25]. **VF** | **OQ** | Not supported. **VF** | `ceb` printed text supported. Hiligaynon, Iloko, Pampanga, Pangasinan and Bikol are also listed [S37]. **VF** |
| Handwriting | Latin script supported [S25][S27]. **VF** | Detected by default [S29]. **VF** | English only [S33]. **VF** | English among the 12 handwriting languages. **Filipino is not one of them** [S37]. **VF** |
| Skew / quality | Not documented | Deskew, rotation correction, page image-quality score with defect types [S29]. **VF** | Recommends at least 150 DPI [S32]. **VF** | **OQ** |
| Confidence | Word, symbol and page structure in JSON [S26]. **VF** | Document JSON [S29]. **VF** | 0–100 per block [S32]. **VF** | JSON with confidence. **OQ** (not read in detail) |
| Regions (APAC) | Global endpoint plus `eu` and `us` regional endpoints only. **No Singapore endpoint documented** [S26]. **VF** | `asia-southeast1` (Singapore) is a single region with "limited support" [S30]. **VF** | `ap-southeast-1` Singapore available. Outside the main regions, DetectDocumentText is limited to **1 TPS** [S34]. **VF** | Southeast Asia region reported available [S40]. **Sec / OQ** |

---

## 3. Pre-processing for phone photos and poor scans

| Problem | What fixes it | Tools | Label |
|---|---|---|---|
| Low resolution | Tesseract "works best" at **about 300 DPI or higher**, so rescale first | Tesseract docs [S5] | **VF** |
| Uneven lighting, shadows (phone photos) | Tesseract's default Otsu thresholding struggles with uneven backgrounds. **Adaptive Otsu and Sauvola** were added in 5.0 | `thresholding_method` parameter [S5]. OpenCV adaptive threshold [S51] | **VF** / OpenCV docs **Sec** (403) |
| Skew | Rotate so text lines are horizontal. Heavy skew breaks line segmentation | OCRmyPDF `--deskew` [S9][S5] | **VF** |
| Sideways or upside-down pages | Detect orientation (0/90/180/270) | OCRmyPDF `--rotate-pages` with a threshold. PaddleOCR doc-orientation model [S9][S16] | **VF** |
| Perspective (photo taken at an angle), curled pages | Four-corner perspective warp, or learned unwarping | OpenCV `getPerspectiveTransform`/`warpPerspective` (**Sec**, docs 403). PaddleOCR **UVDoc** [S16] (**VF**). OCRmyPDF has **no** perspective correction (**Est**, none documented in [S9][S10]) | mixed |
| Dark scanner edges, speckle | Remove borders and noise before OCR | `--clean` (unpaper) changes only the OCR input. `--clean-final` also changes the output image [S9]. unpaper is GPL-2.0 and was last released in 2022 [S14] | **VF** |
| Noisy grey or colour backgrounds | Background removal | `--remove-background`. "Should not be used on documents that contain color photos" [S9] | **VF** |

**Consequences.** Use `--clean` only: `--clean-final` and `--remove-background` alter the derived PDF's image, and an exhibit copy should look as received. **Recommendation.** Phone photos need a perspective-correction step (OpenCV or a capture app that crops and flattens) before OCR; the capture method is an R04 question. **Est.** All engines struggled with **blur**, and Tesseract was "much more sensitive to salt & pepper noise" [S42] (**IB**), so capture should reject blurry photos. **Est**

---

## 4. Cloud privacy and the Data Privacy Act

| Point | Google Vision | Google Document AI | AWS Textract | Azure Document Intelligence |
|---|---|---|---|---|
| Used for training? | "Google does not use the content you send to train and improve our Cloud Vision features" [S28]. **VF** | "We never use customer data to train our Document AI models" [S31]. **VF** | **By default it "may store and use document and image inputs"** to improve the service. Opt-out is an AWS Organizations AI-services opt-out policy [S33][S35]. Content kept for improvement "might" be stored in another region [S35]. **VF** | The privacy page states 24-hour deletion. It contains **no explicit training statement** [S38]. A Microsoft Q&A answer says data is not used for training [S39]. **Sec / OQ** |
| Retention | Sync: "processed in memory and not persisted to disk". Async: deleted after processing, with a TTL of "a few hours" [S28]. **VF** | Sync not persisted. Batch deleted after processing, up to 1 day TTL on failure [S31]. **VF** | Stored in-region at rest. If opted out, content is not stored in another region [S33]. **VF** | Input and results kept **24 hours**, then auto-deleted. Can be deleted earlier with the Delete Analyze Result API [S38]. **VF** |
| Sub-processor list | Published page exists [S41]. Content not read. **OQ** | Vendors are under "appropriate security and confidentiality contractual obligations" [S31]. **VF** | Published list. Updated at least 30 days before a new sub-processor, with email notice [S36]. **VF** | Not checked. **OQ** |

**DPA relevance.** Per R10 (**VF** there [S55]): Officetul is the client's **PIP**; it may not engage a sub-processor without the PIC's prior instruction (IRR s.44(b)(4)); the agreement must state the **location of processing** (s.44(a)); the PIC stays accountable for transfers (RA s.21). Therefore:

1. **Self-hosted OCR adds no sub-processor** — the simplest DPA position. **Recommendation**
2. **A cloud fallback must be listed as a sub-processor with its region and be opt-in per tenant**, so a law firm can refuse it. **Recommendation**
3. **If AWS is ever used** (e.g., later AI extraction), set the org-level AI opt-out **before** any client document is sent. **Recommendation**
4. **Google Vision has no Singapore residency option** [S26]; Document AI `asia-southeast1` ("limited support" [S30]) or Azure Southeast Asia (unconfirmed [S40]) are the APAC candidates. **OQ**

---

## 5. Output formats for search and evidence

| Output | Producer | Use in Officetul | Label |
|---|---|---|---|
| **Sidecar text** (`--sidecar`) | OCRmyPDF. Contains only the text OCRmyPDF found, so existing text and skipped pages are excluded. `--output-type none` gives text only [S9] | **The text fed to the search index.** Store it per version as a derived artefact | **VF** / **Recommendation** |
| **Searchable PDF/A** (image plus invisible text) | OCRmyPDF (PDF/A is the default) [S7] | "View with selectable text" rendition. **Derived file. The original is kept unchanged** | **VF** / **Recommendation** |
| **hOCR / ALTO** (word boxes and confidence) | Tesseract directly (`hocr`, `alto`) [S1]. OCRmyPDF no longer exposes hOCR [S10] | Search-hit highlighting on the page image, and per-word confidence. Optional for MVP | **VF** / **Est** |
| **Confidence scores** | Tesseract TSV/hOCR, PaddleOCR `rec_scores` [S16], Textract 0–100 [S32] | Flag "low-confidence OCR" documents for re-scan or fallback, using a page-level mean | **Recommendation** |

**Signed and born-digital PDFs.**

- OCRmyPDF "will refuse to modify a signed PDF regardless of other settings" unless `--invalidate-digital-signatures` is passed [S9]; signatures would then no longer validate [S13] (**Sec**). **VF.** Never pass that flag; for signed PDFs use embedded text, or OCR rendered page images to sidecar text only. **Recommendation** (whether OCRmyPDF refuses even with `--output-type none` is **OQ**).
- `--skip-text` (`--mode skip`) copies pages with text unchanged; `--redo-ocr` strips only invisible OCR text; `--force-ocr` rasterises pages and **flattens forms** [S10]. **VF.** Default to skip mode; never force. **Recommendation**

---

## 6. Benchmark design (Phase 7)

### 6.1 Sample set (target 240 pages; the owner may adjust)

| Stratum | Pages | Why | Lawful source (suggested) |
|---|---|---|---|
| Thermal-paper receipts (incl. faded) | 30 | Highest risk: low contrast, narrow fonts | Staff's **own** receipts with consent, PII redacted. Public receipt sets: **CORD** (Indonesian receipts, CC-BY-4.0) [S46]. **SROIE** (ICDAR 2019; licence **OQ**) [S47] |
| BIR forms (2307, 2550Q, 1701, 0605 types), filled | 30 | Boxes, small print, mixed typed and handwritten entries | Blank forms from **bir.gov.ph** [S45], filled with **synthetic** data and printed and scanned in-house |
| Typed contracts and letters (English) | 30 | Baseline: should be easy | Synthetic contracts. Owner-supplied samples with client consent |
| **Filipino-language letters and notices** | 30 | Language model coverage, Taglish | Government issuances in Filipino, e.g. DILG memorandum circulars on "Paggamit ng Wikang Filipino" [S56]. Government works carry no copyright, but profit-making exploitation needs agency approval (RA 8293 s.176.1) [S44]. **VF.** Internal test use is likely fine (**OQ**, ask a lawyer) |
| Cebuano text | 10 | Edge language | Public LGU notices in Cebuano (**OQ**, source not yet found) |
| Court documents (decisions, pleadings) | 30 | Long text, footnotes, stamps | Supreme Court E-Library decisions [S57] (site TLS failed on access; **OQ**). Printed and re-scanned to simulate paper |
| Construction drawing title blocks | 20 | Small text, rotated labels, large formats | Synthetic title blocks. Public LGU or DPWH plan sheets (**OQ**) |
| **Phone photos** of the above (angled, shadowed, curled) | 50 | Real capture path | Re-photograph a subset of the strata above with 2–3 mid-range Android phones |
| Already-digital and signed PDFs | 10 | Must skip or refuse correctly. Not scored for accuracy | Self-signed test PDFs |

Each document should carry metadata: dpi, capture device, skew angle, language, and paper type. **Recommendation**

### 6.2 Ground truth

1. **Seed** with the best engine's output; **two people correct independently**; adjudicate (gives inter-annotator agreement). **Recommendation**
2. **Normalise** before scoring: Unicode NFC, collapsed whitespace, one rule for quotes and for ₱/PHP/P; human reading order. **Recommendation**
3. **Store** page-level plain text, plus ALTO or PAGE where layout matters; dinglehopper (Apache-2.0) reads all three [S48]. **VF**

### 6.3 Metrics

| Metric | Why | Tool |
|---|---|---|
| **CER** (character error rate) | Standard edit-distance metric. Penalises every wrong character | dinglehopper [S48], ocreval (ISRI, used by [S42]) [S49] |
| **WER** / word accuracy | Closer to searchability. ISRI word accuracy ignores stopwords and case [S42] | ocreval, jiwer (Apache-2.0) [S50] |
| **Word recall for search** | Share of ground-truth content words (unique, non-stopword, at least 3 characters) found exactly in the OCR text. Search cares about misses more than extras | Small script (Phase 7) |
| **Keyword-hit rate** | For a fixed **query list** per stratum, the share of documents returned by the real Officetul search index (TIN, OR number, supplier, case number, Filipino words like "kasunduan" and "bayad"). This is the user-facing metric | The search stack itself |
| **Garbage ratio** | Extra text beyond the ground truth length. Tesseract emitted excess text on noise, which gave negative ISRI accuracy on 4.6% of measurements [S42]. **IB** | Same script |
| Throughput | Pages/minute and p95 seconds/page per worker, end to end (rasterise, preprocess, OCR, PDF/A, sidecar) | Wall-clock on the stated server |
| Peak RSS memory per job | Sizes queue workers. Compare with the OCRmyPDF rule of thumb [S11] | `/usr/bin/time -v` |
| **Cost per 1,000 pages** | (server hourly price ÷ pages/hour) × 1,000 for self-hosted. List price for cloud | Self-hosted ≈₱1–2 and cloud ≈US$1.50 per 1,000, both from prior pricing research, not re-verified here. **Est** |

**Server class:** fix and state it (the production queue-worker VM type: vCPU, RAM, CPU model, no GPU); run with `--jobs` = vCPUs and with 1 job. **Configurations (~8):** Tesseract `eng` fast, `eng` best, `eng+fil` best, `script/Latin` best; OCRmyPDF with and without `--clean --deskew --rotate-pages`; PaddleOCR PP-OCRv5 mobile and server on CPU; Azure Read and Google Vision/Document AI **on synthetic and public pages only** until a DPA position is agreed. **Recommendation**

### 6.4 Pass/fail criteria: how to set them (no numbers proposed)

All **Recommendations**:

1. **Baseline first.** Report CER, word recall and keyword-hit rate per stratum with confidence intervals (wide at ~30 pages, acceptable for go/no-go).
2. **Calibrate by eye.** Show the owner 5–10 pages at different error levels with a live search on each; the owner marks "good enough to find this document" or not. Thresholds come from that session.
3. **Per-stratum bars** (receipts and phone photos get their own, not the average).
4. **Comparative rule for the fallback:** enable cloud for a stratum only if its keyword-hit gain exceeds a margin the owner sets, at the agreed cost.
5. **Non-accuracy gates:** signed PDFs untouched and originals byte-identical (100% by design); p95 seconds/page and peak memory within limits the owner agrees.

---

## 7. Staged recommendation

| Stage | What runs | Trigger to move on |
|---|---|---|
| **MVP** | OCRmyPDF (skip mode, `--deskew --rotate-pages --clean`, `-l eng+fil`, `--sidecar`, PDF/A derived) on Tesseract 5 `tessdata_best`. Laravel queue job calls the CLI through `Process` with a long timeout. Tenant-selectable language set adds `ceb`. Never `--invalidate-digital-signatures` or force mode. Store sidecar text and per-page mean confidence. paperless-ngx (GPL-3.0; reference only, not reusable code) uses the same OCRmyPDF pattern [S54] (**Sec**). **Recommendation** | Phase 7 shows a stratum below the agreed bar |
| **Fallback A (self-hosted)** | PaddleOCR PP-OCRv5 as a Python microservice for the **phone-photo** path (orientation and UVDoc unwarping). Possibly an OCRmyPDF engine plugin later [S12]. No new sub-processor. **Recommendation** | Fallback A also fails the phone-photo or receipt bar |
| **Fallback B (cloud, opt-in)** | **Azure Read** (Filipino and Cebuano printed) or **Google Vision/Document AI** (Filipino supported, Cebuano experimental). Per-tenant opt-in, disclosed as a sub-processor, used only for low-confidence pages. **Not AWS Textract.** **Recommendation** | — |
| **Revisit** | Re-run yearly; on a Tesseract major release; if a CPU-viable, permissively licensed VLM OCR appears; and when metered AI extraction ships (it calls a cloud model anyway). **Recommendation** | — |

---

## 8. (a) Comparison table

| Engine | Licence fit | Filipino | Cebuano | CPU-only viable | Signed-PDF safe pipeline | DPA burden | Maintenance | R12 role |
|---|---|---|---|---|---|---|---|---|
| Tesseract 5 + OCRmyPDF | Good (Apache-2.0, MPL-2.0) | Yes (`fil`) | Yes | Yes | Yes, refuses by default | None | Active (2026) | **MVP** |
| PaddleOCR PP-OCRv5 | Good (Apache-2.0) | Tagalog `tl` | No | Yes (VC timings) | Wrap yourself | None | Active (2026) | **Fallback A** |
| EasyOCR | Good | Tagalog `tl` | No | Yes | Wrap yourself | None | **Stale** (2024) | Not recommended |
| docTR | Good | Vocab only, model OQ | No | OQ | Wrap yourself | None | Active (2026) | Watch |
| Surya | **Weights restricted** above US$5M | OQ | OQ | **Slow** (0.1 p/s, VC) | Wrap yourself | None | Active | Not recommended |
| Kraken | Good | No model | No | OQ | Wrap yourself | None | Active | Not suited |
| Google Vision | Pay-per-use | Supported | Experimental | n/a | Yes, if only images are sent | Sub-processor; US/EU/global only | n/a | Fallback B candidate |
| Google Document AI OCR | Pay-per-use | OQ | OQ | n/a | Same | Sub-processor; Singapore "limited" | n/a | Fallback B candidate |
| AWS Textract | Pay-per-use | **No** | **No** | n/a | Same | Training **opt-out needed** | n/a | **Excluded** |
| Azure Read | Pay-per-use | Yes | Yes | n/a | Same | Sub-processor; 24 h retention; training statement OQ | n/a | **Fallback B (preferred)** |

**Independent accuracy evidence:** only [S42] (2021 runs; Tesseract 4.1.1; no preprocessing; English book scans with synthetic noise). English word error rates annotated in its Fig. 4 (Document AI / Textract / Tesseract): **no added noise 1.3 / 1.8 / 2.4%; one noise layer 8.2 / 12 / 23.3%; two layers 17.5 / 22.5 / 41.6%**. **IB.** Clean pages are close; noisy pages are not. The data are too old and too unlike Officetul's documents to pick a winner. Vendor tables (docTR's FUNSD/CORD comparison [S22], Surya's olmOCR-bench score [S23]) are **VC** and are not used for ranking.

## 9. (b) Recommendation

**Recommendation — MVP:** OCRmyPDF + Tesseract 5 `tessdata_best` (`eng+fil`, optional `ceb`) on CPU via Laravel queue: sidecar text for search, PDF/A derived file, originals and signed PDFs untouched, no sub-processor. **Fallback:** PaddleOCR (self-hosted) for phone photos first; then Azure Read as an opt-in, disclosed cloud fallback for low-confidence pages. Both pending Phase 7 and the owner's thresholds.

## 10. (c) Benchmark plan and (d) open questions

**(c) Plan** (detail in §6): build the 240-page lawful set → double-key ground truth → run ~8 configurations on the fixed worker class → score CER, WER, word recall, keyword-hit rate, garbage ratio, throughput, memory, ₱/1,000 pages → owner calibration session sets per-stratum bars → confirm MVP or switch on a fallback.

**(d) Open questions**

1. What accuracy does any engine reach on Filipino/Tagalog and Cebuano business text? No benchmark was found.
2. Does `eng+fil` hurt English accuracy compared with `eng` alone? How does `script/Latin` compare? Measure both in Phase 7.
3. Does OCRmyPDF refuse a signed PDF even with `--output-type none --sidecar`? If so, signed PDFs need a separate path: render the pages, then run Tesseract.
4. Azure: is there an explicit contractual statement that Document Intelligence does not use inputs for training? Is Read v4.0 GA in Southeast Asia?
5. Does Google Document AI OCR support Filipino and Cebuano, and which OCR versions run in `asia-southeast1`?
6. What is the SROIE dataset licence, and is there a public Cebuano or construction-drawing source?
7. What is Tesseract's peak memory and real throughput on Officetul's worker VM? No published figure exists for Tesseract 5.
8. Does the chosen search engine tolerate OCR typos, for example with fuzzy matching? This affects how much CER matters. It is an R-search topic.
9. Can Officetul use RA 8293 s.176 government works for an internal benchmark only? A lawyer should confirm.

---

## 11. (e) Sources (all accessed 2026-10-20)

- [S1] Tesseract README (licence, output formats, 100+ languages) — https://github.com/tesseract-ocr/tesseract
- [S2] Tesseract releases (5.5.3, 2026-07-24) — https://github.com/tesseract-ocr/tesseract/releases
- [S3] Traineddata files by version (fil, tgl, ceb) — https://tesseract-ocr.github.io/tessdoc/Data-Files-in-different-versions.html
- [S4] Traineddata files (best/fast/legacy, osd) — https://tesseract-ocr.github.io/tessdoc/Data-Files.html
- [S5] Improving the quality of the output — https://tesseract-ocr.github.io/tessdoc/ImproveQuality.html
- [S6] tessdata_best / tessdata_fast / tessdata repos (licence, file sizes via GitHub API) — https://github.com/tesseract-ocr/tessdata_best , https://github.com/tesseract-ocr/tessdata_fast , https://github.com/tesseract-ocr/tessdata
- [S7] OCRmyPDF README — https://github.com/ocrmypdf/OCRmyPDF
- [S8] OCRmyPDF releases (v17.13.0, 2026-09-28) — https://github.com/ocrmypdf/OCRmyPDF/releases
- [S9] OCRmyPDF Cookbook (sidecar, clean, deskew, rotate, signatures, languages) — https://ocrmypdf.readthedocs.io/en/latest/cookbook.html
- [S10] OCRmyPDF Advanced features (modes, renderer, timeouts, 8.4 MP note) — https://ocrmypdf.readthedocs.io/en/latest/advanced.html
- [S11] OCRmyPDF Performance (memory per job) — https://ocrmypdf.readthedocs.io/en/latest/performance.html
- [S12] OCRmyPDF Plugins — https://ocrmypdf.readthedocs.io/en/latest/plugins.html
- [S13] OCRmyPDF PDF security issues (search excerpt only, **Sec**) — https://ocrmypdf.readthedocs.io/en/latest/pdfsecurity.html
- [S14] unpaper (GPL-2.0, release 7.0.0 2022-04-20) — https://github.com/unpaper/unpaper
- [S15] PP-OCRv5 multilingual (Tagalog `tl`) — https://www.paddleocr.ai/latest/en/version3.x/algorithm/PP-OCRv5/PP-OCRv5_multi_languages.html
- [S16] PaddleOCR general OCR pipeline (model sizes, timings, UVDoc, serving) — https://www.paddleocr.ai/latest/en/version3.x/pipeline_usage/OCR.html
- [S17] PaddleOCR repo (Apache-2.0, v3.7.0) — https://github.com/PaddlePaddle/PaddleOCR
- [S18] EasyOCR supported languages — https://www.jaided.ai/easyocr/
- [S19] EasyOCR repo — https://github.com/JaidedAI/EasyOCR
- [S20] docTR repo — https://github.com/mindee/doctr
- [S21] docTR vocabs — https://mindee.github.io/doctr/latest/modules/datasets.html
- [S22] docTR model choice and benchmark (**VC**) — https://mindee.github.io/doctr/latest/using_doctr/using_models.html
- [S23] Surya repo (licence, speed, olmOCR-bench) — https://github.com/datalab-to/surya
- [S24] Kraken repo — https://github.com/mittagessen/kraken
- [S25] Cloud Vision OCR language support — https://docs.cloud.google.com/vision/docs/languages
- [S26] Cloud Vision detect text (regional endpoints) — https://docs.cloud.google.com/vision/docs/ocr
- [S27] Cloud Vision handwriting — https://docs.cloud.google.com/vision/docs/handwriting
- [S28] Cloud Vision data usage — https://docs.cloud.google.com/vision/docs/data-usage
- [S29] Document AI Enterprise Document OCR — https://docs.cloud.google.com/document-ai/docs/enterprise-document-ocr
- [S30] Document AI regions — https://docs.cloud.google.com/document-ai/docs/regions
- [S31] Document AI security — https://docs.cloud.google.com/document-ai/docs/security
- [S32] Textract best practices (languages, 150 DPI, confidence) — https://docs.aws.amazon.com/textract/latest/dg/textract-best-practices.html
- [S33] Textract FAQ (data use, opt-out, languages, handwriting) — https://aws.amazon.com/textract/faqs/
- [S34] Textract endpoints and quotas — https://docs.aws.amazon.com/general/latest/gr/textract.html
- [S35] AWS Organizations AI services opt-out policies — https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_ai-opt-out.html
- [S36] AWS sub-processors — https://aws.amazon.com/compliance/sub-processors/
- [S37] Azure Document Intelligence Read/Layout language support — https://learn.microsoft.com/en-us/azure/ai-services/document-intelligence/language-support/ocr
- [S38] Azure Document Intelligence data, privacy and security — https://learn.microsoft.com/en-us/azure/foundry/responsible-ai/document-intelligence/data-privacy-security
- [S39] Microsoft Q&A, "Will the input data in Document Intelligence be used for model training?" (**Sec**) — https://learn.microsoft.com/en-us/answers/questions/2151648/will-the-input-data-in-document-intelligence-be-us
- [S40] Microsoft Q&A on Document Intelligence region availability (**Sec**) — https://learn.microsoft.com/en-us/answers/a/1912475
- [S41] Google Cloud subprocessors (page exists, content not read) — https://cloud.google.com/terms/subprocessors
- [S42] Hegghammer, T. (2022). "OCR with Tesseract, Amazon Textract, and Google Document AI: a benchmarking experiment." *Journal of Computational Social Science* 5:861–882, CC BY 4.0, doi:10.1007/s42001-021-00149-1 (read via the Oxford ORA PDF) — https://ora.ox.ac.uk/objects/uuid:0f922084-1e7a-425a-92b0-7cbc7aba7626
- [S43] Laravel 12.x Processes — https://laravel.com/docs/12.x/processes
- [S44] RA 8293 (Intellectual Property Code) s.176 — https://lawphil.net/statutes/repacts/ra1997/ra_8293_1997.html
- [S45] BIR Forms index — https://www.bir.gov.ph/bir-forms
- [S46] CORD receipt dataset (CC-BY-4.0) — https://github.com/clovaai/cord
- [S47] ICDAR 2019 SROIE (licence **OQ**; code mirror MIT) — https://rrc.cvc.uab.es/?ch=13 , https://github.com/zzzDavid/ICDAR-2019-SROIE
- [S48] dinglehopper (Apache-2.0) — https://github.com/qurator-spk/dinglehopper
- [S49] ocreval (Apache-2.0) — https://github.com/eddieantonio/ocreval
- [S50] jiwer (Apache-2.0) — https://github.com/jitsi/jiwer
- [S51] OpenCV (Apache-2.0; docs returned 403) — https://github.com/opencv/opencv , https://docs.opencv.org/4.x/da/d6e/tutorial_py_geometric_transformations.html
- [S52] tesseract-ocr-for-php (MIT) — https://github.com/thiagoalessio/tesseract-ocr-for-php
- [S53] OCRmyPDF-EasyOCR plugin (MIT, v0.2.1 2024-08-30) — https://github.com/ocrmypdf/OCRmyPDF-EasyOCR
- [S54] paperless-ngx (GPL-3.0, reference architecture using OCRmyPDF; docs 403) — https://github.com/paperless-ngx/paperless-ngx
- [S55] Internal: R10 Data Privacy Act duties (IRR ss.43–44, RA s.21) — research/R10-data-privacy-for-saas-dms.md
- [S56] DILG MC 2024-048 "Paggamit ng Wikang Filipino…" (aggregator, **Sec**) — https://batasnatin.com/laws/mc-2024-048
- [S57] Supreme Court E-Library (TLS error on access) — https://elibrary.judiciary.gov.ph/
