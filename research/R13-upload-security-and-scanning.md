# R13 — Secure, resource-aware upload and processing pipeline

- **Status:** First pass, design-level. Research only, no code. Defensive security research for Officetul Documents.
- **Researched:** 2026-10-20. **All sources accessed 2026-10-20.**
- **Question:** How should Officetul accept large uploads (≥100 MB), email and camera captures, validate and scan them, process them safely, and serve them without leaking across tenants?
- **Labels:** **Verified fact** (read in the primary source), **Vendor claim**, **Secondary** (news or search excerpt only), **Estimate**, **Open question**, **Recommendation** (proposed, not approved). Source IDs `[S#]` point to §13.

**Access caveats.** NVD pages render with JavaScript and returned nothing, so CVE text was read from the CVE.org record API, and exploitation status from the CISA KEV JSON feed (catalogue version 2026.10.16). Laravel 13.x is current; the filesystem page was read on 12.x, and the 12.x page itself says the 13.x page exists. Ookla's site blocked fetching, so Philippine network speeds are **Secondary**.

---

## 1. Large uploads

### 1.1 Options

| Option | How it works | Fit for Officetul | Label |
|---|---|---|---|
| **A. Presigned multipart direct to object storage** | Browser (Uppy `@uppy/aws-s3`) asks Laravel to sign CreateMultipartUpload, UploadPart, ListParts, Complete and Abort; bytes go straight to S3. Uppy uses multipart for files over 100 MiB by default. Default part size is file size ÷ 10,000, clamped to S3's 5 MiB–5 GiB range. `ExposeHeaders` must include `ETag`. An interrupted multipart upload "can resume where it stopped." | Web and PHP workers never carry the bytes. Best for ≥100 MB. | **Verified fact** [S17][S11] |
| **B. tus through a tus server** | Core protocol: HEAD returns `Upload-Offset`; PATCH resumes from it. Checksum extension rejects a bad chunk with `460`. The server MUST support `sha1`. Expiration extension lets the server delete unfinished uploads. | tusd (Go) can store on disk, GCS or S3-compatible storage. tus-php supports checksum (SHA-256 by default), expiration and concatenation. Under nginx it needs `fastcgi_request_buffering` off. One more service to run. | **Verified fact** [S16][S18][S19] |
| **C. Chunked upload through Laravel** (pionl/laravel-chunk-upload) | JS library (resumable.js, flow.js, Dropzone) sends chunks to a Laravel route; PHP assembles them. | Supports Laravel 9–13. The README does not mention resume. PHP-FPM workers and local disk carry every byte. | **Verified fact** [S20] |

**S3 limits:** 10,000 parts; parts of 5 MiB–5 GiB (the last part may be smaller); objects up to 48.8 TiB. AWS suggests multipart "when your object size reaches 100 MB". **Verified fact** [S11]

**Recommendation:** use **A** (Uppy plus Laravel signing endpoints) on any S3-compatible store. Keep a plain single-PUT path for small phone-camera images. **B** is the fallback if the chosen local-DC storage lacks solid multipart support (**Open question**, see §12).

### 1.2 Philippine mobile networks

| Point | Detail | Label |
|---|---|---|
| Speeds | August 2026: Philippine median mobile download was 66.33 Mbps, against a global 109.05 Mbps. No 2026 upload median was found. Older Ookla figures quoted in BusinessWorld put mobile upload at about 5.4–5.9 Mbps (year not shown in the excerpt). | **Secondary** [S52] |
| Time to upload 100 MB | At 5 Mbps, about 160 s. At 2 Mbps, about 7 min. A dropped connection with no resume means starting again. | **Estimate** |
| Design consequence | Resumable parts, per-part retry, and Uppy's 6-file concurrency default, which should be lowered on mobile. | **Recommendation** [S17] |

### 1.3 Server limits (only needed if bytes pass through Laravel)

| Setting | Default | What it does | Label |
|---|---|---|---|
| nginx `client_max_body_size` | 1m | Larger requests get `413`; `0` disables the check | **Verified fact** [S10] |
| nginx `client_body_timeout` | 60s | Counts only between two reads, not the whole body | **Verified fact** [S10] |
| PHP `upload_max_filesize` / `post_max_size` | 2M / 8M | `post_max_size` must exceed `upload_max_filesize`. Over the limit, `$_POST` and `$_FILES` arrive empty. | **Verified fact** [S9] |
| PHP `memory_limit` | 128M | "Generally speaking", should be larger than `post_max_size` | **Verified fact** [S9] |

**Recommendation:** with option A, keep the app's body limits small (for example 25 MB, enough for camera images) and never raise them to 100 MB+. Big uploads should never touch PHP-FPM.

### 1.4 Integrity (SHA-256 of the original)

| Point | Detail | Label |
|---|---|---|
| S3 cannot check a full-object SHA-256 on multipart | Full-object checksums on multipart exist only for CRC64NVME, CRC32 and CRC32C. SHA-256 on multipart is **composite**: a checksum of part checksums, not the file's SHA-256. | **Verified fact** [S12] |
| S3 still checks transport | S3 checks per-part checksums and rejects a mismatch with `BadDigest`. A full-object CRC64NVME can be supplied at Complete. | **Verified fact** [S12] |
| Multipart ETag is not MD5 | For multipart objects the ETag "is not an MD5 digest" | **Verified fact** [S12] |
| Browser hashing of 100 MB+ | WebCrypto `digest()` "does not support streaming input: you must read the entire input into memory" | **Verified fact** [S54] |
| Design | The **authoritative SHA-256** is computed **server-side** by a worker that streams the assembled object once, at the same time as the scan read. A client-side hash (computed incrementally with a JS library) is an optional hint for early duplicate detection. It is never trusted. Store `sha256`, size, the S3 full-object CRC64NVME and the S3 `VersionId`. | **Recommendation** |

### 1.5 Quotas and abuse

| Control | Detail | Label |
|---|---|---|
| ASVS | 5.2.1 (L1): accept only file sizes the app can process. 5.2.4 (L3): per-user size quota and file-count limit. | **Verified fact** [S2] |
| Presigned PUT does not cap size | Anyone holding the URL can upload, and an existing key is replaced. Expiry can be up to 7 days via SDK. | **Verified fact** [S13] |
| Enforcement | At sign time, check declared size against the per-tenant quota and the per-file maximum, and sign only `ceil(size / partSize)` parts. At Complete, `HeadObject` the real size, then abort and delete if it is over. Keep a per-tenant running total in MySQL. | **Recommendation** |
| Orphans | Incomplete multipart uploads are charged until aborted. Add an `AbortIncompleteMultipartUpload` lifecycle rule (AWS's example uses 7 days). | **Verified fact** [S14] |

---

## 2. Type validation

| Point | Detail | Label |
|---|---|---|
| Client `Content-Type` | "don't trust the Content-Type header as it can be spoofed" | **Verified fact** [S1] |
| Extension | Use an allowlist "for business functionality". Reject names containing `:` (NTFS alternate data streams). | **Verified fact** [S1] |
| Magic bytes | Signature checks "should not be used on its own, as bypassing it is pretty common" | **Verified fact** [S1] |
| Laravel `mimes` | "validates the MIME type of the file by reading the file's contents". It does **not** check that the content matches the extension: a PNG named `photo.txt` passes `mimes:png`. Use `extensions` together with it. | **Verified fact** [S4] |
| Under the hood | `mimes` calls `guessExtension()`. `mimetypes` calls `getMimeType()`. Both block `.php`, `.phtml` and `.phar` client extensions unless `php` is listed. Symfony MimeTypes asks `FileinfoMimeTypeGuesser` (PHP `finfo`) first, then the `file` binary. | **Verified fact** [S5][S6] |
| Polyglots | A file can be a valid PDF and also valid HTML, JS or ZIP. libmagic reports one type, so passing a sniff does not make the file safe to render. | **Estimate** (well-known class; no single primary source fetched) |
| ASVS 5.2.2 (L1/L2) | Check that the extension matches the content, inside archives too | **Verified fact** [S2] |
| ASVS 5.2.6 (L3) | Reject images over a maximum pixel size (pixel flood) | **Verified fact** [S2] |
| Filenames | OWASP: use a random UUID name. ASVS 5.3.2: build paths from internal data, never user names. Laravel: `getClientOriginalName()` is "unsafe"; `hashName()` gives a random name. | **Verified fact** [S1][S2][S7] |

**Recommendation (allowlist v1):** PDF, JPEG, PNG, HEIC/HEIF (phone camera), TIFF (scanners), DOCX, XLSX and plain CSV. **Reject** HTML, SVG, XML, JS, executables, macro-enabled Office formats (`.docm`/`.xlsm`) and archives in v1. OWASP advises against ZIP uploads [S1]. A file must pass all three checks: extension allowlist, sniffed type matching the extension, and a parser check by the worker (the file opens with the expected library). Key every object as `tenants/{tenant_id}/{uuid}`. Keep the original filename only as display metadata, normalised to Unicode NFC, stripped of control characters and path separators, and length-capped.

---

## 3. Malware scanning

### 3.1 ClamAV facts

| Topic | Detail | Label |
|---|---|---|
| RAM | Minimum **3 GiB**, preferred **4 GiB**. Signatures alone take "upwards of 1.2 GiB". Reloads take about **2.4 GiB** briefly because clamd holds two engines. The note is dated 2020-09-20, so the database has likely grown. | **Verified fact** [S21]; current size **Open question** |
| Reload trade-off | `ConcurrentDatabaseReload` (default yes) keeps scanning during a reload. Set it to `no` to block scans briefly in exchange for half the peak RAM. | **Verified fact** [S22][S23] |
| Size limits (defaults) | `MaxFileSize` **100M** (larger files "won't be scanned"). `MaxScanSize` 400M. `StreamMaxLength` **100M** (INSTREAM connection closed). `MaxRecursion` 17. `MaxFiles` 10,000. `MaxScanTime` 120,000 ms. | **Verified fact** [S22] |
| Hard ceilings | "Technical design limitations prevent ClamAV from scanning files greater than **2 GB**". All size options are capped at 4 GB. | **Verified fact** [S23] |
| **Silent pass trap** | `AlertExceedsMax` (default **no**): only when it is yes are files over MaxFileSize, MaxScanSize or MaxRecursion flagged. With the default, an over-limit 150 MB PDF can come back "OK" without being fully scanned. | **Verified fact** (option text) [S22]; consequence **Estimate** |
| Off-by-default heuristics | `AlertOLE2Macros`, `AlertEncrypted`, `AlertEncryptedArchive` and `AlertEncryptedDoc` all default to no. `ScanPDF` and `ScanOLE2` default to yes. | **Verified fact** [S22] |
| Socket | Use a Unix `LocalSocket` with `LocalSocketMode`, or `--fdpass` when clamd runs as another user. clamd "does not currently protect or authenticate traffic" on TCP. | **Verified fact** [S28] |
| Updates | `freshclam` `Checks` default is **12/day**. HTTP 429 means rate limiting; cloud IPs (AWS, GCP, Azure, Oracle) "will most likely be rate limited". Use a private mirror (`cvdupdate`) only if needed, since freshclam with CDIFFs is "far more efficient". | **Verified fact** [S24][S25][S26] |
| False positives | Allow-list a file by hash (`.sfp` SHA-256) or ignore a signature by name (`.ign2`). Report to Talos with sigtool output. | **Verified fact** [S27] |
| clamd is itself attack surface | CVE-2025-20260: PDF scanning buffer overflow, possible code execution. CVE-2025-20128: OLE2 decryption DoS. CVE-2023-20032: HFS+ parser heap overflow. | **Verified fact** [S42] |

### 3.2 What AV does and does not catch

| Catches reasonably | Weak or misses | Label |
|---|---|---|
| Known malware families, known malicious documents, EICAR, known phishing signatures | New (zero-day) exploit documents, targeted malicious PDF JavaScript, unsigned macros unless `AlertOLE2Macros` is on, encrypted archives and PDFs unless the alert options are on, content beyond the size limits | **Estimate** (follows from signature-based design and the defaults in [S22]) |

So AV is **one layer**. OWASP also lists CDR, image rewriting and sandboxes [S1]. Officetul's main protection is **never executing or rendering originals in a privileged context** (§5–6).

### 3.3 Alternatives and privacy (Data Privacy Act)

| Service | Facts | Privacy fit | Label |
|---|---|---|---|
| **AWS GuardDuty Malware Protection for S3** | Scans each new object (PutObject, CompleteMultipartUpload, …) in the **same Region**, in an isolated VPC with no internet, and deletes its copy after the scan. Can tag the object `GuardDutyMalwareScanStatus` = NO_THREATS_FOUND / THREATS_FOUND / UNSUPPORTED / ACCESS_DENIED / FAILED. Results go to EventBridge with **at-least-once** delivery. Limits: 100 GB per object, 100,000 extracted files, depth 100, 25 buckets per account per Region. Price (us-east-1): **$0.09/GB** plus **$0.215 per 1,000 objects**; free tier 1 GB and 1,000 objects per month. | Data stays in the Region. Becomes an AWS sub-processor, already likely if S3 is used. AWS-only. | **Verified fact** [S29][S30][S31][S32] |
| GuardDuty in Singapore | Launch notes say "all Regions where GuardDuty is available" except China and GovCloud, both added later. Singapore is not named on a fetched page. | — | **Secondary** [S33]; **Open question** |
| **VirusTotal** | Results are shared with partners. "The contents of submitted files… may also be shared with premium VirusTotal customers." | **Do not upload client files.** That would disclose personal data to third parties. A hash-only lookup leaks less but is still a disclosure decision. | **Verified fact** [S34] |
| **Cloudmersive** | Says it processes "in memory and does not retain the payload". A 2025 analytics feature keeps data 90 days on Managed Instance. | Another cross-border sub-processor. Needs a DPA and a data-location check. | **Vendor claim** [S35] |

**Recommendation:** **ClamAV in its own container** as the baseline, because it works in both hosting options (Singapore cloud or local DC). If hosting is AWS Singapore, **add GuardDuty S3** as a second engine for defence in depth. Never use public multi-scanners on tenant content. Any scanning vendor goes on the sub-processor list (R10 §11) [S53].

---

## 4. Quarantine and fail-closed policy

### 4.1 File state machine (**Recommendation**)

```
upload_pending ─(complete+size ok)→ received ─→ validating ─┬→ rejected (type/size/parse)
                                                            └→ quarantined ─→ scanning ─┬→ infected (terminal, blocked)
                                                                                         ├→ scan_error ─(retry ≤N)→ scanning
                                                                                         │              └(exhausted)→ held_for_review
                                                                                         └→ clean ─→ processing ─┬→ ready (in Inbox)
                                                                                                                └→ processing_failed (original downloadable, no preview/OCR)
ready ─(user files it)→ filed
```

Rules:

1. **Only `clean` or later is ever readable by tenant users.** Everything before that lives in a `quarantine/` prefix or bucket that the web app's IAM role cannot GET. The scanner role can read it; the promote step copies to `originals/`.
2. **No timer ever releases a file.** Only a scan verdict moves a file forward.
3. Every transition is a row in `document_file_events` (who or what, when, from→to, engine, signature version).
4. Keep originals in a **separate locked bucket**. Compliance-mode Object Lock "can't be overwritten or deleted by any user, including the root user" [S15], so infected files must never enter it. The SES S3 action also does not support buckets with default Object Lock retention [S37].

### 4.2 Idempotency and duplicates

| Point | Detail | Label |
|---|---|---|
| Job keys | `ShouldBeUnique` with `uniqueId()`, or `WithoutOverlapping($key)->expireAfter()`. Key = `tenant_id:sha256:stage`. | **Verified fact** (mechanism) [S8]; key choice **Recommendation** |
| Timeouts | A job's timeout "should always be less than its 'retry after' value", or the job may run twice. Needs the PCNTL extension. Default timeout 60 s; Redis `retry_after` 90 s. | **Verified fact** [S8] |
| Retries | Default is one attempt. `#[Tries]` and `#[MaxExceptions]` set limits. | **Verified fact** [S8] |
| GuardDuty duplicates | At-least-once delivery means handlers must be idempotent | **Verified fact** [S30] |
| Duplicates within a tenant | Same `tenant_id` and `sha256` already present: skip re-processing and show "already in Documents / Inbox". The capture event is still recorded. | **Recommendation** |
| **No cross-tenant dedup** | Sharing blobs or scan results across tenants keyed by hash lets one tenant learn whether another tenant holds a given file, through timing or a "duplicate" message | **Recommendation** (side-channel reasoning; **Estimate**) |
| Verdict cache | Reuse a clean verdict only for the same tenant, same hash **and** same signature version | **Recommendation** |

---

## 5. Safe processing and sandboxing

### 5.1 Parser risk (why converters get sandboxed)

| Component | Example CVE | What it allowed | KEV? | Label |
|---|---|---|---|---|
| ImageMagick | CVE-2016-3714 "ImageTragick" | Code execution via MVG, MSL, HTTPS and other coders | Yes (added 2024-09-09) | **Verified fact** [S42][S43] |
| Ghostscript | CVE-2024-29510 (<10.03.1) | Format-string bug, **SAFER sandbox bypass** | No | **Verified fact** [S42][S43] |
| Ghostscript | CVE-2023-36664 (<10.01.2) | Pipe-device permission bypass | No; CVE-2017-8291 is in KEV | **Verified fact** [S42][S43] |
| LibreOffice | CVE-2024-12426 (24.8 <24.8.4) | Document URLs exfiltrate **environment variables** and INI values | — | **Verified fact** [S42] |
| LibreOffice | CVE-2024-12425 | Path traversal, arbitrary `.ttf` write | — | **Verified fact** [S42] |
| LibreOffice | CVE-2023-2255 | External frames loaded without prompt | — | **Verified fact** [S42] |
| poppler | CVE-2025-52886 (<25.06.0) | Use-after-free | — | **Verified fact** [S42] |
| pdf.js | CVE-2024-4367 (pdfjs-dist ≤4.1.392) | **Arbitrary JS** via fonts. Fixed in 4.2.67; workaround `isEvalSupported: false`. | — | **Verified fact** [S42][S44] |
| libwebp / FreeType | CVE-2023-4863, CVE-2025-27363 | Heap overflow / out-of-bounds write | Both yes | **Verified fact** [S42][S43] |

### 5.2 Controls

| Control | Detail | Label |
|---|---|---|
| Separate worker pool | Queues `scan`, `convert` and `ocr` run on worker containers or hosts separate from web, and never with web's `.env`. Laravel workers can take queues by priority (`--queue=high,default`). | **Verified fact** (mechanism) [S8]; split **Recommendation** |
| No network | `--network none` leaves "only the loopback device". This blocks CVE-2024-12426-style exfiltration and ImageTragick's HTTPS fetches. | **Verified fact** [S49]; mapping **Recommendation** |
| Memory and CPU caps | `--memory` is a hard limit and the kernel kills processes on OOM. Set `--memory-swap` equal to `--memory` for no swap. Use `--cpus`. | **Verified fact** [S48] |
| Wall-clock limit | `timeout -k 10s 120s <cmd>`. Exit code 124 means timed out. | **Verified fact** [S50] |
| Unprivileged | Non-root user, read-only root filesystem, a tmpfs scratch directory wiped after each job, no secrets mounted. The worker passes bytes in and gets derivatives out (a PHP sidecar holds the S3 credentials). | **Recommendation** |
| ImageMagick | Use the hardened policy example: memory 256MiB, map 512MiB, **area 32MP**, disk 1GiB, time 60 s, width/height 4KP, list-length 16. Deny all delegates and modules except the formats needed. "Run ImageMagick within restricted environments." | **Verified fact** [S40] |
| Ghostscript | SAFER is the default since 9.50. Pre-9.50 SAFER "could be overridden". Never pass `-dNOSAFER`. Track patch levels, because SAFER has been bypassed (CVE-2024-29510). | **Verified fact** [S41][S42] |
| LibreOffice | Disabling macros in headless conversion: only bug-tracker and forum material found (an Expert setting that disables all macro execution). Treat macros as possibly live and rely on the container (no network, no secrets). Reject `.docm`/`.xlsm` in v1. | **Secondary** [S51]; **Open question** |
| Bombs | ASVS 5.2.3 (L2): check uncompressed size and file count **before** decompressing. 5.2.5 and 5.3.3 (L3): no symlinks; ignore archive path info (zip slip). DOCX/XLSX are ZIPs, so limits apply to them too. | **Verified fact** [S2] |
| Derivatives | Previews are re-encoded raster images (PNG/WebP), so the browser never parses the original. OCR = poppler `pdftoppm` → Tesseract inside the same sandbox. | **Recommendation** |
| Patching | Rebuild images weekly, plus on any KEV entry for these components. Watch KEV for Ghostscript, ImageMagick, LibreOffice, poppler, ClamAV, libwebp and FreeType. This also serves NPC 2023-06 s.32 vulnerability management. | **Recommendation** [S43][S53] |

---

## 6. Serving files safely

| Control | Detail | Label |
|---|---|---|
| Authorise every request | Every download, preview, thumbnail, OCR text or search snippet goes through a Laravel controller that applies a tenant-scoped policy before building any URL. Never put raw storage keys in the client. OWASP: "use a handler that gets mapped to filenames inside the application". | **Verified fact** [S1]; design **Recommendation** |
| Short-lived URL | `Storage::temporaryUrl($path, now()->plus(minutes: 5), ['ResponseContentType'=>…, 'ResponseContentDisposition'=>'attachment; …'])`. Local disk supports temporary URLs via `serve => true`. | **Verified fact** [S7] |
| Content-Disposition | `attachment` asks the browser to save. Send `filename*` (RFC 5987) with an ASCII `filename` fallback. Strip control characters and paths. ASVS 5.4.1–5.4.2: set and encode the filename. | **Verified fact** [S47][S2] |
| nosniff | `X-Content-Type-Options: nosniff` makes the browser use the declared type, so `text/plain` "won't be interpreted as HTML" | **Verified fact** [S45] |
| CSP sandbox | `Content-Security-Policy: sandbox` blocks scripts and popups and applies a unique origin. It works only as an HTTP header, not in `<meta>`. | **Verified fact** [S46] |
| Separate origin | OWASP: store files "on a different server". Serve previews and any inline originals from a separate registrable domain (e.g. `officetul-usercontent.com`), never a subdomain that shares cookies. | **Verified fact** [S1]; domain **Recommendation** |
| PDF viewer | If an in-browser PDF view is offered: pdfjs-dist ≥4.2.67 with `isEvalSupported: false`, on the separate origin. Default to raster previews. | **Verified fact** [S44]; **Recommendation** |
| IDOR | UUID identifiers, a global tenant scope on every Eloquent query, and policy tests that try tenant A's IDs against tenant B | **Recommendation** |
| Search | Filter by `tenant_id` and ACL **inside** the search query, not after it. Build snippets only from documents that passed the same check. Never index files that are not `clean`. | **Recommendation** |
| Treat URLs as secrets | Presigned URLs are bearer tokens. Don't log full URLs, and keep expiry short (minutes, not the 7-day maximum [S13]). | **Recommendation** |

---

## 7. Email intake

| Topic | Detail | Label |
|---|---|---|
| SES verdicts | Receipt includes `spfVerdict`, `dkimVerdict`, `dmarcVerdict` (+ `dmarcPolicy` on fail), `spamVerdict` and `virusVerdict`, each PASS / FAIL / GRAY / PROCESSING_FAILED | **Verified fact** [S36] |
| SES size | S3 action: maximum email **40 MB including headers**. Raw MIME is stored, and is encrypted client-side if KMS is chosen. | **Verified fact** [S37] |
| SES region | Inbound endpoint exists in **ap-southeast-1 (Singapore)** | **Verified fact** [S38] |
| Postmark | Attachments total **35 MB** maximum, sent as base64 in JSON. SpamAssassin headers. `Received-SPF` (pass, neutral, softfail, fail). DKIM/DMARC verdicts are not documented on that page. | **Verified fact** [S39]; DKIM/DMARC **Open question** |
| Effect of base64 | Attachments grow about 4/3 in MIME, so the real attachment ceiling is about 30 MB on SES and 35 MB raw on Postmark. **The ≥100 MB requirement cannot be met by email**; the bounce or notice should point to browser upload. | **Estimate** |

**Recommendation (intake rules):**

1. Accept only if the sender is on the tenant's allowlist **and** DMARC = PASS, or SPF or DKIM PASS aligned with the From domain. Put GRAY or FAIL in a "held" Inbox state visible only to tenant admins. Never auto-file it.
2. Treat the SES `virusVerdict` as advisory. Still run each extracted attachment through the full pipeline (§4).
3. Parse MIME in the sandboxed worker, not in the web tier. Each attachment becomes its own file record linked to the message ID.
4. Archives in email are rejected in v1, same as browser uploads.
5. Rate limits per tenant inbox, for example messages per hour and bytes per day, plus a global circuit breaker. Over the limit: hold and alert, don't drop silently.
6. NPC Circular 2023-06 s.23 requires "secure transmission and reception of email messages, including attachments" [S53].

---

## 8. Logging and monitoring

| Event | Fields | Alert? | Label |
|---|---|---|---|
| Upload signed / completed / aborted | tenant, user, declared vs actual size, uuid | Quota breach | **Recommendation** |
| Type rejected | ext, sniffed type, reason | Spike per tenant | OWASP: log "input validation failures" **Verified fact** [S3] |
| Scan result | engine, signature version, verdict, sha256, duration | **Every infected file**; any `scan_error` backlog over 15 min | **Recommendation** |
| Scanner health | clamd PING, signature age, freshclam 429s | Signature age over 24 h, or clamd down | **Recommendation** [S25] |
| Sandbox kills | OOM, timeout 124, non-zero exit | Repeated kills on the same hash | **Recommendation** [S48][S50] |
| Authorisation failures | cross-tenant ID attempts, expired-URL reuse | Any cross-tenant attempt | OWASP: authz failures **Verified fact** [S3] |
| Email verdicts | SPF/DKIM/DMARC, sender allowlisted?, held | Burst from a non-allowlisted sender | **Recommendation** |
| Never log | presigned URLs, file contents, OCR text, session tokens | — | **Verified fact** (exclusions) [S3] |

---

## 9. Recommended pipeline (**Recommendation**)

1. **Sign.** Laravel authenticates the user, checks the tenant quota and the per-file maximum (proposed v1 maximum: **1 GB**, under ClamAV's 2 GB ceiling), checks the extension allowlist, then creates `quarantine/{tenant}/{uuid}` and signs a bounded number of parts with 5-minute URLs.
2. **Upload.** Uppy multipart to the quarantine bucket. Resumable, with the S3 per-part checksum.
3. **Complete.** Laravel completes the upload, runs `HeadObject` to compare actual and declared size, sets the state to `received`, and dispatches `ValidateAndHash` (unique key = uuid).
4. **Validate and hash** (sandbox, no network). Stream once: SHA-256, magic bytes, extension match, parser open test, archive and pixel limits. Fail: `rejected`. Pass: `quarantined`.
5. **Scan.** clamd (Unix socket, own container, `AlertExceedsMax yes`, `MaxFileSize`/`MaxScanSize`/`StreamMaxLength` ≥ app maximum, `AlertOLE2Macros yes`, `AlertEncrypted yes`), plus GuardDuty if on AWS. Infected: block and notify. Error: retry. Clean: promote.
6. **Promote.** Copy to `originals/{tenant}/{uuid}` (versioned, Object Lock as R09/R10 decide), record `VersionId`, delete from quarantine.
7. **Process.** `convert` and `ocr` queues, sandboxed: raster previews, OCR text, thumbnails. Failure gives `processing_failed`, but the original stays available.
8. **Inbox.** The item appears in the tenant's review Inbox and is indexed under tenant and ACL filters.
9. **Serve.** Authorised controller → short-lived URL on the user-content domain with `attachment`, `nosniff` and CSP `sandbox`.

---

## 10. Failure handling

| Scenario | Policy | Rationale |
|---|---|---|
| clamd down / unreachable | Stay in `quarantined`. Back off and retry. Alert at 15 min. **Never release.** | Fail closed (ASVS 5.4.3 [S2]) |
| Scan timeout (`MaxScanTime` or job timeout) | `scan_error`, retry ≤3, then `held_for_review` by Officetul ops. The tenant sees "security check pending". | A timeout may mean a bomb |
| File above scanner limits | Reject at sign time. If one slips through, `AlertExceedsMax` makes it a finding, not a pass. | Default config silently skips [S22] |
| Infected | `infected`: never served. Notify the uploader and tenant admins. Keep 30 days in a locked-down quarantine for false-positive appeals, then delete. Keep hash and verdict in the audit log permanently. | Supports appeals and breach analysis without keeping malware |
| False positive claimed | Officetul ops review. Release only with a per-hash `.sfp` entry for that tenant and an audit record. | ClamAV allow-list mechanism [S27] |
| Encrypted PDF/ZIP (cannot scan) | `held_for_review`. The tenant admin may accept the risk, which is logged. Never preview it. | AV cannot see inside |
| Corrupt or unparseable file | `rejected` with a reason. The original is not kept. | It cannot be a usable record |
| Converter crash / OOM / timeout | `processing_failed` after 2 tries. The original stays downloadable with no preview. Alert if the same hash repeats. | Availability without a risky retry loop |
| Duplicate upload, same tenant | Link to the existing file and show a notice | Saves storage |
| Same hash, different tenant | Process independently | Avoids the side channel (§4.2) |
| Client abandons multipart | Lifecycle abort after 1–7 days, then quota released | [S14] |
| Worker dies mid-job | Unique-lock expiry plus `retry_after` > timeout. Jobs are idempotent per stage. | [S8] |
| Temp files | tmpfs per job, wiped on exit. Nightly sweep of any local temp directory older than 24 h. | No leftovers across tenants |
| Signature updates failing (429) | Alert at 24 h staleness. Keep scanning with the last good database and record its version on each verdict. | [S25] |
| Email from unknown sender / DMARC fail | Hold, visible to admins only, not processed until released | Spoofing defence |

---

## 11. MVP vs later

| MVP | Later |
|---|---|
| Uppy multipart to a quarantine bucket; server-side SHA-256 | tus fallback for local-DC storage if needed |
| Extension allowlist + finfo + parser check; random keys | CDR (rebuild PDF/Office) for high-risk tenants |
| ClamAV container with hardened limits; fail-closed state machine | GuardDuty S3 second engine (if AWS); rescan of the Inbox on new signatures |
| Sandboxed converters (no network, memory/CPU/time caps, ImageMagick policy) | gVisor or Firecracker isolation; seccomp profiles |
| Raster previews only; `attachment` downloads; nosniff; separate user-content domain | In-browser pdf.js viewer on the sandbox origin |
| Per-tenant quota and file-size cap; lifecycle abort | Per-user quotas (ASVS L3) and anomaly alerts |
| Email: SES verdicts + sender allowlist + full pipeline; no archives | Archive support with bomb and symlink checks |
| Security event log and alerts listed in §8 | SIEM export; documented VAPT cadence for NPC |

---

## 12. Open questions

1. Hosting: AWS Singapore (GuardDuty available, SES inbound in Region) or a local DC (S3-compatible store: does it support multipart, versioning and Object Lock equivalently)?
2. Product maximum file size: 1 GB proposed. Is there a real customer need above 1 GB (for example construction drawings or video)? ClamAV cannot scan above 2 GB.
3. Current ClamAV RAM at the 2026 database size; measure on a test box.
4. Is GuardDuty Malware Protection for S3 confirmed in ap-southeast-1 (console check), and what is the Singapore price?
5. Does the chosen inbound provider expose DKIM/DMARC verdicts (Postmark), and should Officetul run its own DMARC check?
6. How to disable LibreOffice macros reliably in headless mode (official configuration key)?
7. Infected-file retention: is 30 days acceptable to a Philippine lawyer for Data Privacy Act proportionality?
8. Should tenant admins be allowed to override `held_for_review` for encrypted files, or only Officetul ops?

---

## 13. Sources (all accessed 2026-10-20)

| ID | Source | URL | Type |
|---|---|---|---|
| S1 | OWASP File Upload Cheat Sheet | https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html | Primary |
| S2 | OWASP ASVS 5.0, V5 File Handling | https://github.com/OWASP/ASVS/blob/master/5.0/en/0x14-V5-File-Handling.md | Primary |
| S3 | OWASP Logging Cheat Sheet | https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html | Primary |
| S4 | Laravel 13.x Validation (also 12.x) | https://laravel.com/docs/13.x/validation | Primary |
| S5 | Laravel `ValidatesAttributes.php` (12.x) | https://raw.githubusercontent.com/laravel/framework/12.x/src/Illuminate/Validation/Concerns/ValidatesAttributes.php | Primary (source) |
| S6 | Symfony Mime `MimeTypes.php` (7.3) | https://raw.githubusercontent.com/symfony/mime/7.3/MimeTypes.php | Primary (source) |
| S7 | Laravel 12.x File Storage | https://laravel.com/docs/12.x/filesystem | Primary |
| S8 | Laravel 13.x Queues | https://laravel.com/docs/13.x/queues | Primary |
| S9 | PHP core ini directives | https://www.php.net/manual/en/ini.core.php | Primary |
| S10 | nginx `ngx_http_core_module` | https://nginx.org/en/docs/http/ngx_http_core_module.html | Primary |
| S11 | Amazon S3 multipart upload limits | https://docs.aws.amazon.com/AmazonS3/latest/userguide/qfacts.html | Primary |
| S12 | S3 checking object integrity (upload) | https://docs.aws.amazon.com/AmazonS3/latest/userguide/checking-object-integrity-upload.html | Primary |
| S13 | S3 presigned URL upload | https://docs.aws.amazon.com/AmazonS3/latest/userguide/PresignedUrlUploadObject.html | Primary |
| S14 | S3 abort incomplete multipart lifecycle | https://docs.aws.amazon.com/AmazonS3/latest/userguide/mpu-abort-incomplete-mpu-lifecycle-config.html | Primary |
| S15 | S3 Object Lock | https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lock.html | Primary |
| S16 | tus resumable upload protocol | https://tus.io/protocols/resumable-upload | Primary |
| S17 | Uppy AWS S3 plugin | https://uppy.io/docs/aws-s3/ | Primary |
| S18 | tus-php README | https://github.com/ankitpokhrel/tus-php | Primary (project) |
| S19 | tusd README | https://github.com/tus/tusd | Primary (project) |
| S20 | pionl/laravel-chunk-upload README | https://github.com/pionl/laravel-chunk-upload | Primary (project) |
| S21 | ClamAV Docker docs (memory) | https://docs.clamav.net/manual/Installing/Docker.html | Primary |
| S22 | ClamAV `clamd.conf.sample` (main) | https://raw.githubusercontent.com/Cisco-Talos/clamav/main/etc/clamd.conf.sample | Primary (source) |
| S23 | `clamd.conf(5)` (Debian mirror of the upstream man page) | https://manpages.debian.org/unstable/clamav-daemon/clamd.conf.5.en.html | Primary (mirror) |
| S24 | `freshclam.conf(5)` (Debian mirror) | https://manpages.debian.org/unstable/clamav-freshclam/freshclam.conf.5.en.html | Primary (mirror) |
| S25 | ClamAV freshclam FAQ | https://docs.clamav.net/faq/faq-freshclam.html | Primary |
| S26 | ClamAV private mirror (cvdupdate) | https://docs.clamav.net/appendix/CvdPrivateMirror.html | Primary (search excerpt) |
| S27 | ClamAV allow lists | https://docs.clamav.net/manual/Signatures/AllowLists.html | Primary |
| S28 | ClamAV scanning usage | https://docs.clamav.net/manual/Usage/Scanning.html | Primary |
| S29 | GuardDuty Malware Protection for S3 | https://docs.aws.amazon.com/guardduty/latest/ug/gdu-malware-protection-s3.html | Primary |
| S30 | How Malware Protection for S3 works | https://docs.aws.amazon.com/guardduty/latest/ug/how-malware-protection-for-s3-gdu-works.html | Primary |
| S31 | Malware Protection for S3 quotas | https://docs.aws.amazon.com/guardduty/latest/ug/malware-protection-s3-quotas-guardduty.html | Primary |
| S32 | GuardDuty pricing | https://aws.amazon.com/guardduty/pricing/ | Primary |
| S33 | AWS What's New, June 2024 launch (search excerpt) | https://aws.amazon.com/about-aws/whats-new/2024/06/detect-malware-object-uploads-amazon-s3-guardduty | Secondary (excerpt) |
| S34 | VirusTotal "How it works" | https://docs.virustotal.com/docs/how-it-works | Primary |
| S35 | Cloudmersive Virus API and release notes (search excerpts) | https://cloudmersive.com/virus-api ; https://cloudmersive.com/release-notes/Virus-Scan-API-Threat-Detection-Analytics | Vendor (excerpt) |
| S36 | SES receiving notification contents | https://docs.aws.amazon.com/ses/latest/dg/receiving-email-notifications-contents.html | Primary |
| S37 | SES Deliver to S3 action | https://docs.aws.amazon.com/ses/latest/dg/receiving-email-action-s3.html | Primary |
| S38 | SES endpoints and quotas | https://docs.aws.amazon.com/general/latest/gr/ses.html | Primary |
| S39 | Postmark inbound parse | https://postmarkapp.com/developer/user-guide/inbound/parse-an-email | Primary |
| S40 | ImageMagick security policy | https://imagemagick.org/security-policy/ | Primary |
| S41 | Ghostscript "Using Ghostscript" (SAFER) | https://ghostscript.readthedocs.io/en/latest/Use.html | Primary |
| S42 | CVE.org records via the CVE API, `https://cveawg.mitre.org/api/cve/<ID>`, for CVE-2016-3714, CVE-2023-36664, CVE-2024-29510, CVE-2023-2255, CVE-2024-12425, CVE-2024-12426, CVE-2025-52886, CVE-2024-4367, CVE-2023-4863, CVE-2025-27363, CVE-2025-20260, CVE-2025-20128, CVE-2023-20032 | https://www.cve.org/ | Primary |
| S43 | CISA KEV catalogue JSON (v2026.10.16) | https://www.cisa.gov/sites/default/files/feeds/known_exploited_vulnerabilities.json | Primary |
| S44 | pdf.js advisory GHSA-wgrm-67xf-hhpq | https://github.com/mozilla/pdf.js/security/advisories/GHSA-wgrm-67xf-hhpq | Primary |
| S45 | MDN `X-Content-Type-Options` | https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/X-Content-Type-Options | Primary (reference) |
| S46 | MDN CSP `sandbox` | https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Content-Security-Policy/sandbox | Primary (reference) |
| S47 | RFC 6266 Content-Disposition | https://www.rfc-editor.org/rfc/rfc6266 | Primary |
| S48 | Docker resource constraints | https://docs.docker.com/engine/containers/resource_constraints/ | Primary |
| S49 | Docker `none` network driver | https://docs.docker.com/engine/network/drivers/none/ | Primary |
| S50 | `timeout(1)` man page | https://man7.org/linux/man-pages/man1/timeout.1.html | Primary |
| S51 | LibreOffice bug 154625 and forum threads on macro settings (search excerpts) | https://www.libreoffice.org/bugzilla/show_bug.cgi?id=154625 | Secondary (excerpt) |
| S52 | Philippines Ookla ranking, Aug 2026 (news, search excerpt); BusinessWorld excerpts | https://pro.edgex.exchange/en-US/news/article/philippines-climbs-ookla-speed-rankings-august-2026 ; https://www.bworldonline.com/?p=420041 | Secondary |
| S53 | R10, Data Privacy Act duties (NPC Circular 2023-06 ss.23, 32) | research/R10-data-privacy-for-saas-dms.md | Internal |
| S54 | MDN `SubtleCrypto.digest()` | https://developer.mozilla.org/en-US/docs/Web/API/SubtleCrypto/digest | Primary (reference) |
