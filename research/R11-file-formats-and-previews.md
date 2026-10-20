# R11 — File formats, previews and text extraction

- **Status:** First pass, complete. Research only. No code.
- **Researched:** 2026-10-20. **All sources accessed 2026-10-20.**
- **Question:** Which file formats should the Officetul Documents app accept, and how should it preview, extract text from, OCR and secure each one? The answers must fit the agreed constraints: originals kept byte-for-byte with a SHA-256 hash; derived files stored separately; signed PDFs never rewritten; files of 100 MB or more accepted; self-hosted open source run from Laravel queue workers on Linux.
- **Labels:** **Verified fact** (read in the primary source), **Vendor claim** (the tool's own description of itself), **Secondary** (aggregator, news or search excerpt only), **Estimate** (my arithmetic or judgement), **Open question**, **Recommendation** (proposed, not approved). `[S#]` points to the source list in §(e).

**Access caveats.** gitlab.freedesktop.org (Poppler) returned an anti-bot page, so the Poppler licence comes from a mirror of its COPYING file and from packaging metadata. Three pages could not be fetched: libreoffice.org's licence page (404), pdfa.org (403) and bamsoftware.com's zip-bomb page (403). Facts that would have come from them are taken from other sources or marked accordingly.

---

## 1. Key findings

| # | Finding | Label |
|---|---|---|
| 1 | OCRmyPDF detects signed PDFs and **will not modify them** unless `--invalidate-digital-signatures` is passed [S3]. The guard was added in v14.4.0 [S6]. | **Verified fact** |
| 2 | OCRmyPDF "does not read encrypted PDFs" (exit code 8) and cannot open certificate-encrypted PDFs [S1][S3]. | **Verified fact** |
| 3 | OCRmyPDF 17 made Ghostscript (AGPL) "no longer strictly required"; pypdfium2 can rasterise and produce PDF/A [S2]. The GitHub README still says Ghostscript is required [S4]. | **Verified fact** (the two sources conflict) |
| 4 | Ghostscript and MuPDF are AGPL-3.0 or commercial. Artifex states that you cannot "deploy our open-source as part of a server-based application or service" without disclosing your own source under AGPL [S26][S27][S28]. | **Verified fact** (Artifex's own reading) |
| 5 | pdf.js CVE-2024-4367 allowed arbitrary script in the host origin through a crafted font. It affected versions up to 4.1.392 and was fixed in 4.2.67. The workaround is `isEvalSupported: false` [S7]. | **Verified fact** |
| 6 | LibreOffice published **about 20 advisories in 2026 alone**. They include heap overflows in PPT, OOXML, DXF, WMF and PDF import, and SSRF, local-file-read and RCE through `calcext:data-mappings` (fixed in 26.2.5 / 26.8.0) [S15]. | **Verified fact** |
| 7 | One Gotenberg instance runs **one LibreOffice conversion at a time**, restarts LibreOffice after 10 conversions by default, and offers allow/deny lists for LibreOffice's outbound URLs [S12]. | **Verified fact** |
| 8 | Apache Tika says it is "not … a security boundary" and should run sandboxed with resource limits. Critical XXE CVE-2025-54988 / CVE-2025-66516 affected 1.13–3.2.1 and was fixed in 3.2.2 [S32][S35][S36]. | **Verified fact** / **Secondary** for the CVE details |
| 9 | Only Safari decodes HEIC natively. Chrome, Edge and Firefox do not [S42]. iOS Safari can **transcode** a photo before upload depending on the `accept` attribute [S43]. The bytes the server receives may therefore not be the camera's bytes. | **Verified fact** / **Secondary** |
| 10 | OCR peak memory is about **15 bytes per pixel of the largest page, per concurrent job**. That is about 500 MB for a letter page at 600 dpi [S5]. | **Verified fact** |

---

## 2. Preview strategy

### 2.1 PDF: server-rendered images vs in-browser pdf.js

| Option | Pros | Cons | Label |
|---|---|---|---|
| **Server-side render** of page images (`pdftoppm -f N -l N -scale-to PX -jpeg`) [S22] | The browser never parses the PDF, so a hostile PDF only meets the sandboxed worker. Works for every browser. Easy to cache as derived files. | CPU and storage per page. A 300-sheet drawing set means 300 renders. Text is not selectable. | Verified fact (options) / Estimate (costs) |
| **pdf.js in the browser** (Apache-2.0 [S10]) | No server CPU. Selectable text and zoom. Range requests fetch only the needed bytes [S8][S9]. | Runs attacker-supplied PDF parsing in the user's browser. History includes CVE-2024-4367 (script injection in the host origin) [S7]. | Verified fact |
| **Recommendation: hybrid** | Server renders **thumbnails plus the first-page preview** (cheap, safe). pdf.js is the full viewer, pinned to a patched version with `isEvalSupported: false`. It is served from a **separate, cookieless origin** inside a sandboxed iframe with a strict CSP. | Two code paths. | **Recommendation** |

### 2.2 Large PDFs (100 MB+ drawing sets)

| Point | Detail | Label |
|---|---|---|
| Range loading | pdf.js "may automatically start using HTTP Range Requests". The vital data is usually at the end of a PDF, so pages render without the whole download [S9]. The default `rangeChunkSize` is 65,536 bytes. `disableAutoFetch` stops background fetching of data the current page does not need, but streaming must also be disabled for it to work [S8]. | **Verified fact** |
| Server needs | The object store or signed URL must honour `Range`. For cross-origin requests, CORS is needed, with the range headers exposed [S9]. | **Verified fact** (CORS) / **Estimate** (exact header list) |
| Linearization | A linearized file can display "before [it is] fully downloaded" [S25]. Linearizing **rewrites** the file (QPDFWriter), so it may only ever be a **derived copy**, never the original. pdf.js loads non-linearized files by range anyway, so a linearized derivative is optional. | **Verified fact** / **Recommendation** |
| Raster cost | Render memory is roughly width × height × 3–4 bytes. An A1 sheet at 150 dpi is about 3,510 × 4,965 px, roughly 17 MP or 52–70 MB. An A0 sheet at 150 dpi is about 35 MP, roughly 105–140 MB. Cap the size with `-scale-to` for previews. Render one page per job. | **Estimate** |
| Large-file worker rule | Workers stream the original from object storage to local temp disk and never load it into PHP memory. They render page by page with a timeout and a memory cgroup. | **Recommendation** |

### 2.3 Office → PDF for preview

| Option | Detail | Label |
|---|---|---|
| LibreOffice headless via **unoserver** (MIT) | Keeps a long-lived LibreOffice listener, which lowers CPU per conversion. The ports have "no security" and "**must not** be accessible" outside the stack. It stops, rather than restarts, if LibreOffice crashes. It has `--conversion-timeout` and `--stop-after` [S18]. | **Verified fact** |
| **Gotenberg** (MIT, Docker) | An HTTP API that wraps LibreOffice and Chromium [S11][S14]. It runs one LibreOffice operation at a time per instance; the docs say to scale out with more instances [S12]. Defaults: `--libreoffice-restart-after` 10, `--api-timeout` 30 s. It has `--chromium-disable-routes`, `--api-disable-download-from`, `--api-body-limit`, and LibreOffice allow/deny lists with `--libreoffice-deny-private-ips` [S12]. Gotenberg gives no RAM figures; its troubleshooting page just says to raise memory and CPU [S13]. | **Verified fact** |
| CVE exposure | 2023–2026 LibreOffice advisories include: remote documents loaded through an IFrame without a prompt (CVE-2023-2255); link targets and graphic on-click bindings running scripts (CVE-2023-6186, CVE-2024-3044); INI and environment-variable exfiltration through URL fetching (CVE-2024-12426, CVE-2026-63278); and many import-filter memory bugs [S15]. | **Verified fact** |
| Macros | The default macro security is **High**: unsigned macros are disabled [S17]. Since CVE-2024-6472 (fixed in 24.2.5), High also disables macros whose certificate check fails [S15]. | **Secondary** (bug tracker) / **Verified fact** |
| Not a signature validator | CVE-2025-2866 was a PDF signature forgery in LibreOffice, and CVE-2024-7788 said signatures in repair mode were trusted [S15]. Never use LibreOffice to judge signatures. | **Verified fact** |
| **Recommendation** | Use **Gotenberg** in its own container, with **no egress** (network-level deny plus `--libreoffice-deny-public-ips` and `--libreoffice-deny-private-ips`), Chromium and download-from routes disabled, a memory limit, and a non-root read-only filesystem. Laravel workers call it only on the internal network. Track LibreOffice releases monthly. This avoids writing process supervision, which matters for a team of one or two. | **Recommendation** |

### 2.4 Images

Generate derived JPEG or WebP thumbnails and a mid-size preview with **libvips**. Set `VIPS_BLOCK_UNTRUSTED` to block loaders flagged untrusted [S40]. Strip EXIF and GPS from derivatives only. ImageMagick is a fallback only, under its hardened `policy.xml`; the documented example caps memory at 256 MiB, area at 32 MP and width/height at 4 KP, and allows only GIF, JPEG, PNG and WEBP [S37]. Never let ImageMagick hand PDFs to Ghostscript. **Recommendation.**

---

## 3. Signed and e-notarized PDFs

### 3.1 How detection works

| Step | Detail | Label |
|---|---|---|
| Structure | A signature is a `/Type /Sig` dictionary. `/Contents` holds a hex PKCS#7/CMS blob and `/ByteRange` lists the hashed spans; the `/Contents` gap itself is excluded. Signing appends an **incremental update**. `/Perms` and `/DocMDP` declare which later changes are permitted [S52]. | **Secondary** (explainer of ISO 32000) |
| Cheap pre-check | Scan the bytes for `/ByteRange` and `/Sig`. This is a heuristic only: compressed object streams or odd encodings can hide the tokens. | **Estimate** |
| Authoritative check | Parse the AcroForm signature fields. `pdfsig` (Poppler) lists each signature with its signer, time, hash algorithm, signed ranges and whether "the whole document" is signed, and can check certificates against NSS with OCSP [S21]. **pyHanko** (MIT) adds integrity checks, X.509 chain validation, difference analysis for changes after signing, and AdES validation [S49]. | **Verified fact** |
| Why validation is hard | The USF, ISA, SWA and Shadow attack classes showed that most viewers and online validators could be fooled. Viewers said the document "has not been modified" while the visible content had changed [S51]. | **Verified fact** |
| **Recommendation** | At ingest: (1) do the byte pre-check; (2) parse with pyHanko or pdfsig; (3) if **any** signature or a DocMDP certification exists, set `is_signed = true`. That flag routes the file away from every tool that writes PDFs. In MVP, show "Contains N digital signatures (not validated)" with the signer names. Full validation (trust lists, LTV) comes later. | **Recommendation** |

### 3.2 Keeping OCR away from the original

| Rule | Detail | Label |
|---|---|---|
| The storage layer is the real guarantee | The original object is write-once (immutable bucket or object lock). Every derived job reads a temp copy and writes to a different key. The SHA-256 is re-checked after each job. A tool flag is **not** the guarantee. | **Recommendation** |
| Signed PDF with a text layer | Extract with `pdftotext`. No OCR. | **Recommendation** |
| Signed scanned PDF | Rasterise the pages (`pdftoppm`), run Tesseract to plain text or hOCR, and store a **text sidecar** for search. Do not produce a "searchable signed PDF": any output would be a new, unsigned document. A derived searchable PDF built from rasters is possible later, labelled "unsigned derived copy". | **Recommendation** |
| OCRmyPDF on unsigned scans | OCRmyPDF grafts an OCR layer while keeping the PDF "minimally altered" [S2]. With `--output-type pdf`, a derived searchable PDF is written; `--sidecar` writes text; `--output-type none` writes only the sidecar [S1]. Leave the signature guard on and **never** pass `--invalidate-digital-signatures` [S3]. Use `--mode skip` so pages that already have text are not re-OCRed [S1]. | **Verified fact** / **Recommendation** |
| PDF/A | PDF/A forbids encryption, JavaScript and audio/video. Its conformance is declared in XMP as `pdfaid:part` and `pdfaid:conformance`. Any migration breaks embedded signatures [S53]. Never "convert to PDF/A" an original. OCRmyPDF's default `auto` output is best-effort PDF/A [S1], which is fine for derivatives. | **Verified fact** |

---

## 4. Licensing

| Tool | Licence | Closed-source SaaS implication | Label |
|---|---|---|---|
| Poppler (pdftotext, pdftoppm, pdfsig, pdfinfo; 26.10.0 current [S19]) | GPL-2.0-or-later (COPYING mirror, packagers' metadata). Some parts are under other licences [S20] | Invoked as a separate CLI and not distributed, so GPL copyleft is not triggered by SaaS use. Do not link it into code you ship to customers. | Secondary (licence) / Estimate (implication) |
| qpdf | Apache-2.0 (pre-v7 releases were Artistic-2.0) [S24] | Permissive. | Verified fact |
| **Ghostscript** | **AGPL-3.0** or commercial (Artifex) [S28][S27] | Artifex says server use requires disclosing your full source under AGPL, or buying a licence [S27]. Security history includes CVE-2024-29510, a `-dSAFER` sandbox bypass fixed in 10.03.1 [S29]. **Avoid in MVP.** | Verified fact (Artifex claim) |
| **MuPDF / PyMuPDF** | **AGPL** or commercial [S26] | Same as Ghostscript. **Avoid.** | Verified fact |
| LibreOffice | MPL-2.0, with LGPLv3+ as the dual option; bundled parts vary [S16] | File-level copyleft. Using it unmodified as a converter is fine. | Secondary |
| Gotenberg | MIT [S14] (bundles LibreOffice, Chromium and PDF engines under their own licences) | Fine. Check the bundled `pdftk` licence if you enable it. | Verified fact / Open question |
| unoserver | MIT [S18] | Fine. | Verified fact |
| Apache Tika | Apache-2.0; subcomponents have separate licences [S34] | Fine. | Verified fact |
| Tesseract (+ Leptonica) | Apache-2.0 (Leptonica is BSD-2-like) [S47] | Fine. | Verified fact |
| OCRmyPDF | MPL-2.0 [S4] | Fine unmodified. Its AGPL exposure comes only through Ghostscript (see §1 #3). | Verified fact |
| pypdfium2 / PDFium | Apache-2.0 or BSD-3; PDFium is BSD-style [S50] | Fine. | Verified fact |
| pdf.js | Apache-2.0 [S10] | Fine. | Verified fact |
| pyHanko | MIT [S49] | Fine. | Verified fact |
| libvips | LGPL-2.1-or-later [S39] | Fine as a dynamically linked library or CLI. | Verified fact |
| libheif | LGPL (library); MIT (sample apps). The x265 encoder is GPL [S41]. HEVC carries **patents** that sit partly outside the pools [S44]. | Decoding only (libde265) is fine for copyright. Patent exposure is an **Open question**. | Verified fact / Open question |
| ImageMagick | ImageMagick License (permissive, attribution) [S38] | Fine. | Verified fact |
| ExifTool | "Same terms as Perl itself" (Artistic or GPL) [S45] | Fine as a CLI. | Verified fact |
| LibreDWG | GPL-3.0-or-later [S57] | CLI use in SaaS is not distribution, but there is no MVP need. | Verified fact |
| UnRAR | UnRAR licence: cannot be used "to develop a RAR (WinRAR) compatible archiver"; Fedora will not ship it [S56] | Avoid. | Secondary |

**AGPL in a nutshell.** AGPL-3.0 §13 attaches source-offer duties to *modified* versions that users reach over a network. Artifex's licensing page reads the duty more broadly, to any server-based use [S27]. Whether running an *unmodified* `gs` binary as a subprocess triggers obligations is a question for a lawyer, **Open question**. The practical answer is to design Ghostscript and MuPDF out. **Recommendation.**

---

## 5. Resource needs (per job)

| Task | Rough need | Label |
|---|---|---|
| OCR (OCRmyPDF and Tesseract) | About 15 bytes per pixel of the largest page per job, about 500 MB at 600 dpi letter [S5]. At 300 dpi letter (about 8.4 MP), about 125 MB. `--jobs` multiplies the peak [S5]. | Verified fact / Estimate (300 dpi) |
| OCR time | About 2–10 s per page per core at 300 dpi. | Estimate |
| Page render (pdftoppm) | 150 dpi A4 is about 9 MB of raster, and an A0 sheet up to about 140 MB (§2.2). Under 1 s per typical page; several seconds for vector-heavy drawings. | Estimate |
| LibreOffice conversion | One conversion per instance at a time [S12]. About 300–700 MB RSS per instance. Seconds per typical DOCX; minutes for large XLSX. | Verified fact / Estimate |
| Tika server (JVM) | 1–2 GB heap. Run in its own container, with tika-pipes (4.x) for per-document timeouts and memory limits [S32]. | Estimate / Verified fact |
| Sizing for 1–2 developers | One worker box with 4 vCPU and 8 GB, with separate queues: `preview` (fast, high concurrency), `ocr` (concurrency 1–2) and `office` (feeds Gotenberg, one instance). | Estimate / Recommendation |

---

## (a) Format matrix

Abbreviations: **Prev** = preview method; **Text** = text extraction; **Meta** = metadata extraction; **Conv** = conversion needed for preview. Every row keeps the original byte-for-byte with SHA-256, and all outputs are derived files. Cost, Meta and Conv details come in Table 2.

**Table 1: handling**

| Format | Accept? | Prev | Text | OCR? |
|---|---|---|---|---|
| PDF (born-digital) | Yes | Server thumbnail and page 1 (`pdftoppm`); pdf.js full view by range | `pdftotext` | Only for pages with no text (`--mode skip`) |
| Scanned PDF | Yes | As PDF | OCR output | **Yes**: OCRmyPDF makes a derived searchable PDF and a sidecar |
| PDF/A | Yes | As PDF | `pdftotext` | If image-only |
| Signed / e-notarized PDF | Yes | As PDF (the original is served unmodified) | `pdftotext` | Raster → Tesseract → **text sidecar only**; never rewritten |
| Encrypted PDF | Yes (store) | None when a user password is set. Owner-password-only files may render [S22] | None | No (OCRmyPDF refuses [S1]) |
| DOCX / DOC | Yes | Gotenberg (LibreOffice) → derived PDF → as PDF | Tika | No (embedded images: later) |
| XLSX / XLS | Yes | Gotenberg → PDF (paging may be poor); first-N-rows HTML table later | Tika | No |
| CSV | Yes | Escaped HTML table of the first N rows | Read directly (charset detection) | No |
| PPTX / PPT | Yes | Gotenberg → PDF | Tika | No |
| TXT | Yes | Escaped text, first N KB | Direct | No |
| RTF | Yes | Gotenberg → PDF | Tika | No |
| JPEG | Yes | libvips derivatives (EXIF orientation applied, EXIF stripped) | OCR | Optional (receipts, ID photos) |
| PNG | Yes | libvips | OCR | Optional |
| TIFF (multi-page) | Yes | libvips per page → derived PDF → as PDF | OCR | **Yes** (scanner output) |
| HEIC / HEIF | Yes | libheif/libvips → JPEG derivative | OCR | Optional |
| WEBP | Yes | libvips derivative | OCR | Optional |
| EML | Yes (scan-to-email source) | Sanitised headers and text body; remote content blocked | Tika RFC822Parser [S33] | Attachments get their own processing |
| MSG | Later | As EML | Tika OutlookExtractor [S33] | Same |
| ZIP / 7z | **Store-only** | Listing of entries only (later) | None | No |
| RAR | **Store-only** | None | None | No |
| DWG | **Store-only** | None (no open renderer of quality) | None | No |
| DXF | **Store-only** | None in MVP | None | No |
| Audio / video | **Store-only** | None in MVP | None | No |

**Table 2: security, cost, priority**

| Format | Meta | Conv | Security concerns | Size / cost | MVP priority |
|---|---|---|---|---|---|
| PDF | `pdfinfo` / ExifTool | No | Embedded JS, launch actions, embedded files, parser CVEs (e.g. Poppler CVE-2025-32365 JBIG2 [S23]; LibreOffice PDF import [S15]), polyglots | Low; 100 MB+ handled by range loading | **MVP** |
| Scanned PDF | Same | No | Same | **High** CPU and RAM (§5) | **MVP** |
| PDF/A | XMP `pdfaid` [S53] | No | Same | Low | **MVP** |
| Signed PDF | pyHanko / `pdfsig` | No | Signature-spoofing attacks [S51]; never let any tool rewrite it | Low | **MVP** (detect); validation later |
| Encrypted PDF | Flag only | n/a | Cannot be scanned for content; certificate encryption is unsupported [S3] | Nil | **MVP store-only + badge** |
| DOCX/DOC | Tika | **Yes** | Macros (`.docm`), remote templates and IFrames (CVE-2023-2255), OLE objects, many import CVEs [S15] | Medium; one LibreOffice job at a time | **MVP** (`.docm`/`.dotm`: store-only) |
| XLSX/XLS | Tika | **Yes** | Macros (`.xlsm`); data-mapping SSRF and RCE (2026 CVEs) [S15]; zip-bomb XML | Medium to high for big sheets | **MVP** (`.xlsm`: store-only) |
| CSV | None | No | Formula injection when a user opens the downloaded original in Excel: cells starting `= + - @` [S58b]. The original cannot be altered, so warn on download. | Low | **MVP** |
| PPTX/PPT | Tika | **Yes** | PPT import overflow (CVE-2026-8356) [S15]; macros (`.pptm`) | Medium | **MVP** (preview best-effort) |
| TXT | None | No | XSS if served as HTML; serve `text/plain` with `nosniff` | Low | **MVP** |
| RTF | Tika | **Yes** | Embedded OLE objects | Low | **MVP** |
| JPEG | ExifTool (GPS is personal data) | Derivative | Decoder CVEs; dimension bombs (cap the pixel count) | Low | **MVP** |
| PNG | ExifTool | Derivative | Decompression bombs (cap the pixel count) | Low | **MVP** |
| TIFF | ExifTool | **Yes** (browsers lack TIFF support, **Open question**) | libtiff CVE history; per-page IFDs [S54]; BigTIFF beyond 4 GB [S54] | Medium | **MVP** |
| HEIC | ExifTool | **Yes** (only Safari decodes it [S42]) | HEVC decoder CVEs; patents [S44] | Medium | **MVP** (phone capture) |
| WEBP | ExifTool | Derivative | CVE-2023-4863 libwebp heap overflow, exploited in the wild [S59] | Low | **MVP** |
| EML | Tika | Text render | HTML bodies (XSS, tracking pixels), nested attachments, MIME bombs | Low | **MVP** for the scan-to-email pipeline; user upload later |
| MSG | Tika | Text render | CFB container with embedded messages [S55] | Low | **Later** |
| ZIP/7z | None | n/a | Zip bombs, path traversal ("zip slip"), any content inside. OWASP advises against accepting ZIPs [S58] | Storage only | **Store-only** |
| RAR | None | n/a | As ZIP, plus UnRAR licence limits [S56] | Storage only | **Store-only** |
| DWG | Tika (simple metadata only [S33]) | n/a | LibreDWG has an incomplete reader and is GPL-3 [S57] | Storage only | **Store-only** |
| DXF | None | n/a | LibreOffice DXF import overflow CVE-2026-6039 [S15] | Storage only | **Store-only** |
| Audio/video | ExifTool / Tika (format metadata [S33]) | n/a | Codec CVEs if transcoded | Large; no transcode | **Store-only** |

**Applies to all formats (Recommendation; OWASP basis [S58]):**
- Use an extension allowlist plus magic-byte detection, and reject on mismatch. This catches polyglots and spoofed `Content-Type`.
- Store under generated names, outside the webroot.
- Serve originals as `Content-Disposition: attachment` with `X-Content-Type-Options: nosniff` from a separate origin.
- Run antivirus (ClamAV) before preview jobs.
- Derived thumbnails and search text inherit the original's ACL. Search snippets come only from documents the user may open. Tika output is untrusted, so escape it before rendering [S32].

---

## (b) Recommended MVP toolchain (Recommendation)

| Job | Tool | Licence | Notes |
|---|---|---|---|
| Type detection | PHP `finfo` (libmagic) plus an extension allowlist | PHP / BSD-style | Runs in the upload request. |
| Hash | PHP `hash_file('sha256')` while streaming | PHP | Computed before any processing. |
| PDF info, text, thumbnails | Poppler `pdfinfo`, `pdftotext`, `pdftoppm` | GPL-2.0+ (CLI) | Separate processes with a timeout and memory limit. |
| Signature detection | pyHanko (`pdfsig` as a lightweight alternative) | MIT / GPL | Sets `is_signed`. |
| OCR (unsigned scans) | OCRmyPDF 17.x + Tesseract (`eng`, plus `fil` [S48]) + pypdfium2 renderer, **Ghostscript not installed** | MPL-2.0 / Apache-2.0 / BSD | Output to a derived key. Use `--mode skip`, `--sidecar` and `--jobs 1`. Confirm the install works without `gs` (§d). |
| OCR (signed scans) | `pdftoppm` → Tesseract → text or hOCR sidecar | GPL CLI / Apache | Never writes a PDF. |
| Office → PDF | Gotenberg 8 (LibreOffice 26.x), no egress, Chromium routes off | MIT / MPL-2.0 | One instance at first; add instances to scale. |
| Office/email text and metadata | Apache Tika server ≥ 3.2.2 (4.1.x preferred [S30]), tika-pipes | Apache-2.0 | Own container, JVM heap 1–2 GB, internal network only. |
| Images and HEIC | libvips (with libheif/libde265), `VIPS_BLOCK_UNTRUSTED` | LGPL | Derivatives only; strip EXIF from derivatives. |
| Image metadata | ExifTool (file names passed after `--` [S45]; ≥ 12.24 for CVE-2021-22204 [S46]) | Artistic/GPL | Read-only use. |
| Viewer | pdf.js ≥ 4.2.67 (current release), `isEvalSupported: false`, sandboxed origin | Apache-2.0 | Range requests from signed URLs. |
| **Excluded** | Ghostscript, MuPDF/PyMuPDF (AGPL); UnRAR; LibreDWG; ImageMagick for PDF | — | — |

---

## (c) Formats to reject or store-only in MVP, and why

| Format | Decision | Why |
|---|---|---|
| ZIP, 7z | **Store-only** (accepted, hashed, AV-scanned, never extracted) | OWASP advises against accepting ZIPs [S58]. Extraction brings bombs, path traversal and unbounded nested content. Construction and legal clients still send drawing bundles as ZIPs, so rejecting them would push people to other tools. |
| RAR | **Store-only** | Same as ZIP, plus the UnRAR licence restriction [S56]. |
| DWG, DXF | **Store-only** | No permissively licensed, complete renderer: LibreDWG is GPL-3 and incomplete [S57]; ODA is proprietary. Ask users to attach the PDF plot alongside the drawing. |
| Audio/video | **Store-only** (metadata only) | Transcoding is heavy (CPU, codec patents, ffmpeg CVE surface). Court and site recordings still need safekeeping. |
| `.docm`, `.xlsm`, `.pptm` and other macro-enabled files | **Store-only** with a warning | Macro and script CVE history [S15]. |
| User-password PDFs | **Store-only** with an "Encrypted" badge | Tools cannot read them [S1]. A later feature could take a password per session, never stored. |
| Executables and scripts (`.exe`, `.js`, `.bat`, `.html`, `.svg`) | **Reject** | No business need. HTML and SVG are XSS vectors when served. |

---

## (d) Open questions

1. Can OCRmyPDF 17.x run fully without Ghostscript (`--output-type pdf` and `auto`) on the target distro? The docs say yes [S1][S2]; the README says it is required [S4]. Test on a clean container.
2. Does running unmodified AGPL binaries as subprocesses in SaaS trigger obligations? This is for a lawyer, and it only matters if Ghostscript or MuPDF is ever added.
3. HEVC patent exposure for server-side HEIC **decoding** in the Philippines.
4. Under which iOS versions does Safari transcode HEIC to JPEG on upload? The `accept` attribute changes the behaviour, and the WebKit fix version is unconfirmed [S43]. Should the app record "as received from the browser" in the custody log?
5. Which browsers render TIFF natively? The caniuse page returned no data.
6. Should signed-PDF validation (pyHanko with the Philippine CA trust roots, LTV) be in MVP, or only detection?
7. Philippine e-notarization formats: are they PAdES PDFs, or detached signatures and seals? This needs R09 follow-up.
8. Measured RAM and CPU for Gotenberg/LibreOffice and Tika on real client files. All current figures are Estimates.
9. Does Gotenberg's bundled `pdftk` licence matter if the PDF-engine routes are disabled?
10. Tesseract `fil` traineddata exists only in the 4.0 tessdata sets [S48]. How accurate is it on Filipino legal text?

---

## (e) Sources (all accessed 2026-10-20)

| ID | Source | URL | Type |
|---|---|---|---|
| S1 | OCRmyPDF docs, Advanced | https://ocrmypdf.readthedocs.io/en/latest/advanced.html | Primary |
| S2 | OCRmyPDF docs, Introduction | https://ocrmypdf.readthedocs.io/en/latest/introduction.html | Primary |
| S3 | OCRmyPDF docs, PDF security issues | https://ocrmypdf.readthedocs.io/en/latest/pdfsecurity.html | Primary |
| S4 | OCRmyPDF GitHub README | https://github.com/ocrmypdf/OCRmyPDF | Primary |
| S5 | OCRmyPDF docs, Performance | https://ocrmypdf.readthedocs.io/en/latest/performance.html | Primary |
| S6 | OCRmyPDF v14 release notes (search excerpt) | https://ocrmypdf.readthedocs.io/en/latest/releasenotes/version14.html | Primary (excerpt) |
| S7 | GitHub Advisory GHSA-wgrm-67xf-hhpq (CVE-2024-4367) | https://github.com/advisories/GHSA-wgrm-67xf-hhpq | Primary |
| S8 | pdf.js API, DocumentInitParameters | https://mozilla.github.io/pdf.js/api/draft/module-pdfjsLib.html | Primary |
| S9 | pdf.js FAQ | https://github.com/mozilla/pdf.js/wiki/Frequently-Asked-Questions | Primary |
| S10 | pdf.js repository | https://github.com/mozilla/pdf.js | Primary |
| S11 | Gotenberg introduction | https://gotenberg.dev/docs/getting-started/introduction | Primary |
| S12 | Gotenberg configuration | https://gotenberg.dev/docs/configuration | Primary |
| S13 | Gotenberg troubleshooting | https://gotenberg.dev/docs/troubleshooting | Primary |
| S14 | Gotenberg repository | https://github.com/gotenberg/gotenberg | Primary |
| S15 | LibreOffice security advisories | https://www.libreoffice.org/about-us/security/advisories/ | Primary |
| S16 | Ask LibreOffice, licence thread (MPL-2.0/LGPLv3+) | https://ask.libreoffice.org/t/what-is-the-right-license-lgpl-or-mpl/45115 | Secondary |
| S17 | LibreOffice bug 157588 (macro security default) | https://bugs.libreoffice.org/show_bug.cgi?id=157588 | Secondary (excerpt) |
| S18 | unoserver repository | https://github.com/unoconv/unoserver | Primary |
| S19 | Poppler home (26.10.0, 2026-10-04) | https://poppler.freedesktop.org/ | Primary |
| S20 | Poppler COPYING (mirror; GPL-2.0+) | https://gitlab.freedesktop.org/chpe/poppler/blob/master/COPYING | Secondary (excerpt) |
| S21 | pdfsig(1) man page | https://manpages.debian.org/testing/poppler-utils/pdfsig.1.en.html | Primary |
| S22 | pdftoppm(1) man page | https://manpages.debian.org/testing/poppler-utils/pdftoppm.1.en.html | Primary |
| S23 | CVE-2025-32365 (Poppler JBIG2) | https://www.sentinelone.com/vulnerability-database/cve-2025-32365/ | Secondary |
| S24 | qpdf repository | https://github.com/qpdf/qpdf | Primary |
| S25 | qpdf docs, Linearization | https://qpdf.readthedocs.io/en/stable/linearization.html | Primary |
| S26 | MuPDF licence | https://mupdf.readthedocs.io/en/1.27.0/license.html | Primary |
| S27 | Artifex licensing | https://artifex.com/licensing | Primary (vendor) |
| S28 | Ghostscript licensing | https://ghostscript.com/licensing | Primary |
| S29 | CVE-2024-29510 (Ghostscript SAFER bypass, fixed 10.03.1) | https://cveawg.mitre.org/api/cve/CVE-2024-29510 | Primary (excerpt) |
| S30 | Apache Tika home (4.1.0, 2026-09-29) | https://tika.apache.org/ | Primary |
| S32 | Apache Tika security model | https://tika.apache.org/security-model.html | Primary |
| S33 | Apache Tika supported formats | https://tika.apache.org/3.2.3/formats.html | Primary |
| S34 | Apache Tika repository | https://github.com/apache/tika | Primary |
| S35 | oss-sec, CVE-2025-54988 | https://seclists.org/oss-sec/2025/q3/130 | Primary (excerpt) |
| S36 | The Hacker News, CVE-2025-66516 | https://thehackernews.com/2025/12/critical-xxe-bug-cve-2025-66516-cvss.html | Secondary |
| S37 | ImageMagick security policy | https://imagemagick.org/security-policy/ | Primary |
| S38 | ImageMagick licence | https://imagemagick.org/license/ | Primary |
| S39 | libvips API | https://www.libvips.org/API/current/ | Primary |
| S40 | libvips `block_untrusted_set` | https://www.libvips.org/API/current/func.block_untrusted_set.html | Primary |
| S41 | libheif repository | https://github.com/strukturag/libheif | Primary |
| S42 | Can I use: HEIF | https://caniuse.com/heif | Secondary (reference) |
| S43 | WebKit bugs 292350 / 303803 (HEIC transcoding) | https://bugs.webkit.org/show_bug.cgi?id=292350 | Secondary (excerpt) |
| S44 | Library of Congress, HEIF format description | https://www.loc.gov/preservation/digital/formats/fdd/fdd000525.shtml | Primary (reference) |
| S45 | ExifTool home (licence, security notes; 13.59) | https://exiftool.org/ | Primary |
| S46 | oss-security, CVE-2021-22204 | https://openwall.com/lists/oss-security/2021/05/09/1 | Primary (excerpt) |
| S47 | Tesseract repository | https://github.com/tesseract-ocr/tesseract | Primary |
| S48 | Tesseract data files by version | https://tesseract-ocr.github.io/tessdoc/Data-Files-in-different-versions.html | Primary |
| S49 | pyHanko repository | https://github.com/MatthiasValvekens/pyHanko | Primary |
| S50 | pypdfium2 repository | https://github.com/pypdfium2-team/pypdfium2 | Primary |
| S51 | PDF-Insecurity.org | https://pdf-insecurity.org/ | Primary (research) |
| S52 | PDF signatures report 2018 (ByteRange explanation) | https://pdf-insecurity.org/download/report-pdf-signatures-2018.pdf | Primary (excerpt) |
| S53 | Library of Congress, PDF/A-2 | https://www.loc.gov/preservation/digital/formats/fdd/fdd000318.shtml | Primary (reference) |
| S54 | Library of Congress, TIFF | https://www.loc.gov/preservation/digital/formats/fdd/fdd000022.shtml | Primary (reference) |
| S55 | Microsoft [MS-OXMSG] | https://learn.microsoft.com/en-us/openspecs/exchange_server_protocols/ms-oxmsg/b046868c-9fbf-41ae-9ffb-8de2bd4eec82 | Primary |
| S56 | SPDX, UnRAR licence | https://spdx.org/licenses/UnRAR.html | Primary (excerpt) |
| S57 | LibreDWG repository | https://github.com/LibreDWG/libredwg | Primary |
| S58 | OWASP File Upload Cheat Sheet | https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html | Primary |
| S58b | OWASP CSV Injection | https://community.owasp.org/attacks/CSV_Injection | Primary |
| S59 | Help Net Security, CVE-2023-4863 | https://helpnetsecurity.com/2023/09/12/cve-2023-4863 | Secondary |
