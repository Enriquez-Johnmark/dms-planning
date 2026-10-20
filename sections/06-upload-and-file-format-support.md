# 06 — Upload and File-Format Support

## 1. Purpose

Decide **which files the Documents app accepts, previews and makes searchable**, how large they may be, and how big files get uploaded reliably.

## 2. Status

**Owner agreed (Round 9, D-044 to D-046), 2026-10-20.** Research: [R11](../research/R11-file-formats-and-previews.md) (formats, tools, licences) and [R13](../research/R13-upload-security-and-scanning.md) §1–2 (large uploads, type checks), both 2026-10-20. Matrix: [appendices/file-format-matrix.md](../appendices/file-format-matrix.md).

## 3. Executive Summary

- **Most business formats are fully supported:** PDF, scans, Word, Excel, PowerPoint, images including iPhone HEIC, multi-page TIFF, and emails. **Store-only:** archives, CAD drawings, audio and video, macro-enabled Office files, password-protected PDFs. **Rejected:** programs, scripts, HTML and SVG. *(Recommendation)*
- **Big uploads go straight from the browser to storage in resumable pieces** (Uppy multipart with signed URLs). They never pass through the web server, and a dropped mobile connection resumes instead of restarting. *(Verified fact for the technique; Recommendation)*
- **Our server computes the fingerprint (SHA-256) after the upload completes.** Storage services can't check a whole-file SHA-256 on multipart uploads, and hashing in the browser loads the whole file into memory. *(Verified fact, R13 §1)*
- **Proposed maximum: 1 GB per file**, above the agreed 100 MB floor and below the virus scanner's 2 GB hard limit. Email intake is capped at about 35–40 MB per message by the email services. *(Verified fact for the limits; Recommendation for 1 GB)*
- **Previews are page images made on our server** in the MVP, including for 100 MB drawings. An in-browser PDF viewer on a separate, sandboxed domain comes later. *(Recommendation)*
- **No AGPL tools.** Ghostscript and MuPDF would oblige us to publish our own code or buy a licence, so the toolchain avoids them. *(Verified fact for the licences, R11 §4)*

## 4. Business Context

Clients scan receipts, contracts and permits (small files), law firms handle court filings (up to 100 MB per file), and contractors handle drawing sets (often larger). Philippine mobile connections drop, so uploads must resume rather than restart.

## 5. Verified Research Findings

| Finding | Label | Source |
|---|---|---|
| Ghostscript and MuPDF are AGPL or commercial; server use requires source disclosure or a licence | Verified fact | R11 §4 |
| LibreOffice had about 20 security advisories in 2026 (SSRF, local file read, RCE) | Verified fact | R11 §1 |
| pdf.js CVE-2024-4367 (fixed 4.2.67) argues for a sandboxed origin for any in-browser viewer | Verified fact | R11 §2 |
| Apache Tika says it is "not a security boundary"; a critical XXE was fixed in 3.2.2 | Verified fact | R11 §1 |
| SES inbound accepts 40 MB per message; Postmark 35 MB of attachments | Verified fact | R13 §7 |
| S3 multipart uploads can't verify a whole-file SHA-256 (only per-part or CRC checks) | Verified fact | R13 §1 |
| Browser rendering of TIFF is limited; HEIC renders only in Safari | Verified fact / Open question | R11 §(a) |

## 6. Technical Analysis

**Upload flow** (detail in [Section 12](12-security-and-antivirus.md)):
1. The browser asks Officetul for permission to upload; Officetul checks login, quota, file type and size.
2. The file uploads in pieces straight to a **quarantine** area of storage.
3. Our worker computes the fingerprint and checks the file's real type.
4. Virus scan.
5. The file is promoted to permanent storage.
6. Previews and OCR are made.
7. The item appears in the Inbox.

**Toolchain** *(Recommendation, R11 §(b))*:

| Job | Tool | Licence |
|---|---|---|
| Type check | PHP finfo plus an extension allowlist | PHP |
| PDF text, page images, info | Poppler (`pdftotext`, `pdftoppm`, `pdfinfo`) | GPL-2.0 (CLI use) |
| Signed-PDF detection | pyHanko or `pdfsig` | MIT / GPL |
| Office → PDF for preview | Gotenberg (LibreOffice) in a no-internet container | MIT / MPL |
| Office and email text | Apache Tika ≥ 3.2.2 in its own container | Apache-2.0 |
| Images, HEIC | libvips with libheif | LGPL |
| Image metadata | ExifTool (read-only) | Artistic/GPL |
| **Excluded** | Ghostscript, MuPDF, ImageMagick for PDFs, UnRAR, LibreDWG | — |

**Assumes S3-compatible object storage.** Hosting is decided in Phases 9–10. A local data centre must support multipart upload and versioning, or the design needs a fallback (R13 §12).

## 7. Alternatives and Trade-offs

| Question | Options | Trade-off |
|---|---|---|
| Large uploads | Through Laravel in chunks / **direct to storage** | Direct is faster and keeps web servers light, but needs object storage |
| Preview | **Server page images** / in-browser PDF viewer | Images are safer and work for huge files; the viewer gives smoother zoom (later) |
| Archives | Reject / **store-only** / unpack | Rejecting pushes contractors to other tools; unpacking brings bombs and traversal attacks |
| Max size | 100 MB / **1 GB** / 2 GB+ | 1 GB covers drawing sets; above 2 GB can't be virus-scanned |

## 8. Recommendation

1. Adopt the [file-format matrix](../appendices/file-format-matrix.md).
2. Maximum **1 GB per file**; email intake about 35 MB per message.
3. Resumable direct-to-storage uploads; server-side fingerprint.
4. Server page-image previews in the MVP; sandboxed in-browser viewer later.
5. The toolchain above, with no AGPL components.

## 9. Cost and Operational Implications

- Conversion and preview containers add CPU and RAM to the server (Gotenberg about 1 GB, Tika 1–2 GB *(Estimate, R11 §5)*). Sized in Phase 10.
- Storage grows with originals plus derived files. Previews are small compared with originals.

## 10. Risks and Limitations

| ID | Risk | Mitigation |
|---|---|---|
| RK-30 | **Converter vulnerabilities** (LibreOffice, Poppler, image libraries) | Isolated containers, no network, limits, prompt patching (Section 12) |
| RK-31 | **Users need previews of store-only formats** (CAD, ZIP) | Ask for the PDF plot; revisit after the pilot |
| RK-32 | **Local hosting lacks multipart storage** | Phase 9–10 hosting check; resumable-upload fallback |

## 11–12. Questions and Decisions

Grilling Round 9, Q1–Q3.

## 13. Confirmed Decisions

| ID | Decision |
|---|---|
| D-044 | File-format matrix |
| D-045 | 1 GB per file; ~35 MB per email |
| D-046 | Resumable direct-to-storage uploads; server-side SHA-256 |

## 14. Open Questions

Whether OCRmyPDF runs fully without Ghostscript (docs conflict, R11 §(d)); Poppler licence confirmation; browser TIFF support.

## 15. Dependencies

Depends on D-015, D-026, D-031, D-042. Feeds Sections 12, 14 (server sizing), 18 (cost).

## 16. Sources

R11 and R13 source lists (accessed 2026-10-20).

## 17. Next Steps

- [x] Format matrix agreed (D-044)
- [x] Size limits agreed (D-045)
- [x] Upload method agreed (D-046)
