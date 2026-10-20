# 12 — Security and Antivirus (file pipeline)

## 1. Purpose

Make sure **no uploaded file can harm Officetul, its users or other companies**, and decide what happens when something goes wrong. This section covers the file pipeline. Wider security (permissions, activity log protection, tenant isolation) comes in Phase 8 together with Section 09.

## 2. Status

**Owner agreed (Round 9, D-047, D-048, D-051), 2026-10-20.** Research: [R13](../research/R13-upload-security-and-scanning.md), with tool risks from [R11](../research/R11-file-formats-and-previews.md) (2026-10-20).

## 3. Executive Summary

- **Fail closed.** A file is visible to nobody (not even in the Inbox) until it has a **clean virus-scan verdict**. If the scanner is down, files wait. **Never release on a timer.** *(Recommendation; OWASP ASVS basis)*
- **ClamAV with hardened settings is the MVP scanner**, in its own container. Its default settings silently skip files over 100 MB and can report them "clean", so its limits must be raised and over-limit files flagged. It needs about **3–4 GB of RAM**, from a 2020 figure (Estimate). *(Verified fact for the defaults)*
- **Antivirus is not enough.** It misses many malicious PDFs and Office files. So the tools that read files run **isolated, with no internet, and with time and memory limits**. Downloads come from a **separate domain** as forced downloads. *(Recommendation)*
- **Never send client files to VirusTotal**: their contents can be shared with VirusTotal's paying customers. If hosted on AWS, GuardDuty's S3 malware scanning (same-region) can be a second engine later. *(Verified fact)*
- **Infected files** are blocked, the uploader and company admins are notified, and the file is kept locked for 30 days for appeals, then deleted. The fingerprint and verdict stay in the log. *(Recommendation; the 30-day period is for the lawyer to confirm)*

## 4. Business Context

Clients email and upload files from anywhere. A single malicious file that reached another user, or exposed another company's documents, would destroy trust and could be a reportable breach under the Data Privacy Act (D-032).

## 5. Verified Research Findings

| Finding | Source |
|---|---|
| ClamAV defaults: `MaxFileSize` 100 MB, `StreamMaxLength` 100 MB; over-limit files may pass unless `AlertExceedsMax` is on; hard ceiling 2 GB | R13 §3 |
| ClamAV needs about 3–4 GiB RAM, plus about 2.4 GiB briefly during daily signature reloads (2020 figure) | R13 §3 |
| ClamAV has had PDF and OLE2 parser bugs, so it runs isolated too | R13 §3 |
| VirusTotal shares submitted files with premium customers | R13 §3 |
| GuardDuty Malware Protection for S3: $0.09/GB + $0.215 per 1,000 files (US East); Singapore availability unverified | R13 §3 |
| Object Lock storage can't delete an infected file, so quarantine must be a separate bucket | R13 §4 |
| ImageMagick, Ghostscript, LibreOffice, Poppler and pdf.js all have serious CVE histories | R11, R13 §5 |

## 6. Technical Analysis

### 6.1 Pipeline *(Recommendation, R13 §9)*

1. **Sign:** check login, company quota, size (≤ 1 GB), type allowlist; issue short-lived upload URLs.
2. **Upload:** resumable pieces into **quarantine** storage.
3. **Validate and hash:** isolated worker with no network computes SHA-256, checks the real type against the extension, test-opens the file, and applies pixel and archive limits.
4. **Scan:** ClamAV (hardened). Clean → continue. Infected → block. Error → retry.
5. **Promote:** move to permanent storage under a random name; record the fingerprint.
6. **Process:** previews and OCR in isolated queues. A failure here does not block the original.
7. **Inbox:** the item appears for review (D-014), visible only to that company.
8. **Serve:** permission check every time; short-lived link on a separate domain; forced download; no content sniffing.

### 6.2 Failure handling *(Recommendation, R13 §10)*

| Scenario | Policy |
|---|---|
| Scanner down | Files stay in quarantine; retry; alert ops at 15 min; **never auto-release** |
| Scan timeout | Retry up to 3 times, then held for review by our ops; the company sees "security check pending" |
| Infected | Blocked, never shown; uploader and company admins notified; locked 30 days for appeal, then deleted; fingerprint and verdict kept in the log |
| False positive claimed | Our ops review; release with a per-fingerprint exception and a log entry |
| Password-protected file (can't be scanned inside) | Held; a **company admin can accept the risk** (logged); stored with an "Encrypted" badge; no preview |
| Corrupt or unreadable | Rejected with a reason shown to the uploader; not filed |
| Converter crash | Original stays downloadable without a preview; alert if the same file keeps failing |
| Same file, same company | Duplicate warning (D-042) |
| Same file, different company | Processed separately, so no one can learn what another company holds |
| Abandoned upload | Cleaned up after 1–7 days; quota released |
| Virus signatures out of date | Alert at 24 h; keep scanning with the last good version and record the version on each verdict |
| Email from an unapproved sender or failing DMARC | Held, visible to company admins only, not processed until released |

### 6.3 Isolation of file-reading tools

Each tool runs in its own container, as an unprivileged user, with **no network access** and with CPU, memory and time limits:
- Gotenberg/LibreOffice: one job at a time, outbound URLs blocked.
- Tika: internal network only, version ≥ 3.2.2.
- Poppler, Tesseract, libvips: pixel limits, untrusted-input mode.
- ClamAV: its own container.

The worker pool is separate from the web servers.

## 7. Alternatives and Trade-offs

| Question | Options | Recommendation |
|---|---|---|
| Scanner | ClamAV / GuardDuty (AWS only) / commercial API | **ClamAV now**; GuardDuty as a second engine later if on AWS; no third-party API that shares files |
| When the scanner is down | Release after a timeout / **hold** | Hold: availability matters less than safety here |
| Encrypted files | Reject / **hold, admin may accept** / ops only | Admin acceptance: bank statements are commonly password-protected |
| Infected retention | Delete at once / **30 days locked** / forever | 30 days for appeals; lawyer to confirm proportionality |

## 8. Recommendation

Adopt §6 as the MVP file-security design. Later: content disarm and reconstruction for high-risk companies, a second scan engine, stronger sandboxing (gVisor), rescanning the Inbox on new signatures, archive unpacking with bomb checks, SIEM export. *(Recommendation, R13 §11)*

## 9. Cost and Operational Implications

- ClamAV RAM (≈3–4 GB) plus converter containers: Phase 10 sizing.
- **Someone must watch alerts** (scanner down, held files, stale signatures). This is an ops duty for the company (with D-032's incident procedure).

## 10. Risks and Limitations

| ID | Risk | Mitigation |
|---|---|---|
| RK-35 | **Malware that AV misses** reaches a user | Isolated processing; forced downloads from a separate domain; page-image previews |
| RK-36 | **Held files pile up** when the scanner fails unnoticed | Alerts; ops duty; status shown to users |
| RK-37 | **Admins accept a malicious encrypted file** | Warning text; logged acceptance; never previewed |

## 11–12. Questions and Decisions

Grilling Round 9, Q4–Q5 and Q8.

## 13. Confirmed Decisions

| ID | Decision |
|---|---|
| D-047 | Fail-closed scanning and failure policies |
| D-048 | Password-protected files: admin may accept risk; Encrypted badge |
| D-051 | Tool isolation; page-image previews; separate download domain; no AGPL tools |

## 14. Open Questions

R13 §12: hosting (AWS Singapore vs local data centre); ClamAV RAM at today's database size; GuardDuty in Singapore; Postmark DMARC verdicts; disabling LibreOffice macros; 30-day infected retention (lawyer).

## 15. Dependencies

Depends on D-014, D-015, D-032, D-033, D-042. Feeds Section 09 (logging), 14 (sizing), 15 (backup excludes quarantine), 18 (cost).

## 16. Sources

R13 (54 sources) and R11 (accessed 2026-10-20).

## 17. Next Steps

- [x] Fail-closed pipeline agreed (D-047)
- [x] Failure policies agreed (D-047, D-048)
- [x] Isolation approach agreed (D-051)
