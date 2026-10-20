# File-Format Matrix (MVP)

**Status:** agreed by the project owner 2026-10-20 (D-044, D-045), in [Section 06](../sections/06-upload-and-file-format-support.md). Full detail, tools and security notes: [R11](../research/R11-file-formats-and-previews.md) §(a). Every accepted file keeps its **Original file** unchanged with a SHA-256 fingerprint. Previews and text are **Derived files** (see [GLOSSARY](../GLOSSARY.md)).

| Format | MVP decision | Preview | Searchable text | Notes |
|---|---|---|---|---|
| PDF (born-digital) | **Full** | Page images | Text layer | Pages without text are OCR'd |
| Scanned PDF | **Full** | Page images | OCR | Main use case |
| PDF/A | **Full** | Page images | Text layer / OCR | |
| Digitally signed or e-notarized PDF | **Full, read-only** | Page images | Text sidecar only | **Never rewritten** (D-031) |
| Password-protected PDF | **Store-only** + "Encrypted" badge | — | — | Common for bank statements; release rule in Section 12 |
| DOCX / DOC | **Full** | Converted to PDF → page images | Extracted | Macro versions (`.docm`) store-only |
| XLSX / XLS | **Full** | Converted → page images (wide sheets may page badly) | Extracted | `.xlsm` store-only |
| CSV / TXT | **Full** | Escaped text or table, first part | Direct | CSV: formula-injection warning on download |
| PPTX / PPT / RTF | **Full** | Converted → page images | Extracted | `.pptm` store-only |
| JPEG / PNG / WEBP | **Full** | Resized image (location data stripped from previews) | OCR | Phone capture |
| TIFF (multi-page) | **Full** | Per page | OCR | Common scanner output |
| HEIC / HEIF | **Full** | Converted to JPEG | OCR | iPhone photos |
| EML (email) | **Full** (scan-to-email) | Safe headers and plain body | Extracted | Attachments processed separately |
| MSG (Outlook) | Later | — | — | |
| ZIP / 7z / RAR | **Store-only** | — | — | Never unpacked in the MVP |
| DWG / DXF (CAD) | **Store-only** | — | — | Ask for the PDF plot too |
| Audio / video | **Store-only** | — | — | No conversion |
| Macro-enabled Office files | **Store-only** + warning | — | — | |
| EXE, scripts, HTML, SVG | **Reject** | — | — | No business need; attack risk |

**Size limits** (proposed): up to **1 GB per file** by browser upload (agreed floor ≥ 100 MB, D-026). Email intake is limited to about **35–40 MB per message** by the email services (R13 §7).
