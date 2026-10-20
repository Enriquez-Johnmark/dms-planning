# R04: Getting paper from the office scanner into the cloud DMS

**Question:** What are the practical ways to get documents from a client's physical scanner (Epson, Brother, Canon, HP, Ricoh/PFU, etc.) into the cloud Laravel DMS automatically? What does each cost, and what does each need to maintain and secure? The owner's goal is "scan on the office scanner → document appears in the DMS", ideally with no vendor-maintained scripts on client PCs.
**Researched:** 2026-10-18. Every source was accessed on 2026-10-18. Citations such as [S1] point to the source list at the end.
**Labels:** **Verified fact** = official doc/spec/pricing page confirms it. **Vendor claim** = vendor marketing. **Anecdotal** = forum, reseller or third-party. **Open question** = not verifiable or sources disagree. **Inference** = my reasoning from the cited facts. **Recommendation** = proposed decision for the owner.

---

## 0. TL;DR

- **Scan-to-email is the only channel that is close to universal *and* needs nothing installed on client PCs.** Every business MFP checked (Epson WF, Brother MFC, Canon iR ADV, HP LaserJet, Ricoh IM) has it, and so do most network scanners. The exceptions are USB-only scanners (Epson DS-C490), the Brother ADS-4300N (probably PC-only), and ScanSnap (hosted relay, 300 mails/month). It always needs an outgoing SMTP server configured on the device. [§1]
- **The hard part is the client's SMTP server, not the DMS.** Microsoft 365 disables Basic-auth SMTP AUTH **by default at the end of December 2026** (admins can re-enable it; final removal date to be announced in H2 2027) [S2]. Google Workspace says devices that sign in with username and password alone have not been supported since **1 May 2025** [S4]. Older scanners that can't do OAuth will break. **Inference:** the DMS vendor should offer its *own* SMTP submission endpoint with per-tenant credentials, so scanners don't depend on the client's mail tenant.
- **Inbound email is cheap.** Amazon SES receiving costs $0.10 per 1,000 emails plus $0.09 per 1,000 256 KB chunks, takes up to 40 MB per message via S3, runs in Singapore, and hands the app SPF/DKIM/DMARC/spam/virus verdicts [S10–S13]. Postmark (Pro, from $16.50/month) has a 35 MB attachment limit and works with the maintained `beyondcode/laravel-mailbox` package [S6–S8, S21].
- **SFTP push is a strong second channel, but only some devices support it.** In the sample, only Brother business models and Canon ScanFront 400 list SFTP [S69][S70][S75]. Epson adds FTPS. Canon iR, HP and Ricoh MFPs list SMB/FTP (Canon also WebDAV). FTP sends passwords and data in clear text [S22]. A managed SFTP endpoint costs about $216/month per endpoint on AWS Transfer Family [S23].
- **Manufacturer clouds almost all target Google Drive, OneDrive, SharePoint, Dropbox and Box, not custom APIs.** Epson Connect API (scan destinations) and Ricoh CloudStream's Generic HTTPS connector are the only documented "push to your URL" routes found [S27][S40]. A portable pattern is "scan to the client's OneDrive/Google Drive folder → DMS pulls via Graph/Drive webhooks" [S42][S43].
- **A "Scan" button in the browser always needs local software** (Dynamsoft service, Asprise Scan App, VintaSoft service). Chrome 142+/Firefox 153+ now add a Local Network Access permission prompt [S47][S48]. That is exactly the per-PC maintenance the owner wants to avoid.
- **Phones:** the native iOS Notes and Google Drive scanners plus a normal upload cost nothing. Paid in-browser SDKs start at about 1,299/yr (Dynamsoft) or around $2,500/yr (Scanbot) [S55][S56].
- **NPC Circular 2023-06 §23 (Emails)** requires a PIC/PIP that transfers personal data by email to "use secure transmission and reception of email messages, including attachments" [S24]. Email ingestion must therefore enforce TLS end to end and be documented.

---

## 1. Scan-to-email on devices sold to PH SMEs

Device-side scan-to-email always sends through an **SMTP server configured in the device's web console**. The one exception found is ScanSnap, which uses a Ricoh-hosted relay. Where a source is marked "excerpt", the official page returned 403 and its content was read through search-engine excerpts.

| Model (class) | Scan-to-email (standalone) | SMTP auth / OAuth | Size handling | Output formats | Label | Src |
|---|---|---|---|---|---|---|
| **Epson WorkForce Pro WF-C5890** (A4 inkjet MFP) | Yes, "through a pre-configured email server". Other destinations: Network Folder/FTP, Computer, Memory, Cloud (Epson Connect), WSD. | Epson MFP mail settings: Off / SMTP AUTH / POP-before-SMTP, with SSL/TLS | Not found | PDF/JPEG/TIFF. **Searchable PDF via PC software** (Epson Scan 2 / Document Capture Pro). | Verified fact (FAQ) / excerpt | [S60][S61][S65] |
| **Epson DS-790WN / DS-785W / DS-1760WN** (network document scanners) | Yes (DS-790WN brochure: folder, email, USB, cloud without a PC) | **OAuth2 for Exchange Online / Outlook.com** (needs Entra enterprise-app consent). No Gmail OAuth found. | **Device setting for max attachment: 1/2/5/10/20/30 MB** | PDF/A on DS network models | Verified fact (excerpt) / Vendor claim | [S62][S64][S67] |
| Epson DS-C490 | **No.** USB-only; email via PC software. | — | — | — | Verified fact | [S66] |
| Epson EcoTank L6490 | **Not confirmed.** A PH reseller lists only Epson Connect scan-to-cloud. | — | — | — | Anecdotal / Open question | [S68] |
| **Brother MFC-L5915DW / MFC-L2885DW** (A4 laser MFP) | Yes (email server) | **SMTP-AUTH, SSL/TLS.** OAuth not stated. | Not stated | Searchable PDF, PDF/A, Office formats | Verified fact | [S70] |
| **Brother ADS-4300N** (network scanner) | **Probably not standalone.** Email appears only as a PC destination; the "Send to Email Server" help lists only ADS-4700W/4900W. | — | — | Searchable PDF, PDF/A, signed/secure PDF | Verified fact / Open question | [S69][S71] |
| **Canon imageRUNNER ADVANCE DX C3926i** (A3 MFP) | Yes (SMTP/POP3). Also SMB 3.1.1, FTP, WebDAV. | **OAuth 2.0 for Exchange Online and Google Workspace** (Remote UI) | **"Maximum Data Size for Sending" splits oversized email**; default 2 MB on several models; 0 = no limit. Canon advises setting it below the mail server's limit. | **Searchable PDF, compact PDF, PDF/A-1b** | Vendor spec (excerpt) / Verified fact (manual) | [S72][S73][S74] |
| Canon imageRUNNER 2630i | Spec sheet lists E-mail (SMTP), SMB, FTP, WebDAV; searchable PDF listed as standard on one sheet, optional on another | — | — | — | Anecdotal (sheet via search; 403) | [S87] |
| **Canon imageFORMULA ScanFront 400** (network scanner) | Yes: email, shared folder, FTP, USB. Protocols **SMB, FTP, SFTP, FTPS, SMTPS**. | SMTPS | — | PDF, PDF/A | Vendor spec (excerpt) | [S75] |
| **HP LaserJet Pro MFP 4103fdw** (A4 laser MFP) | Yes ("Scan to email") | Not found | — | PDF, JPG, TIFF. No OCR stated. Network folder/SharePoint **not confirmed** (HP's table lists them for Color LJ Pro 4301–4303, not 4101–4104). | Verified fact / Open question | [S76][S78] |
| HP LaserJet Enterprise (FutureSmart) | Yes | **Office 365 OAuth on FutureSmart ≥ 3.4** | — | SFTP not found | Verified fact (excerpt) | [S77] |
| **Ricoh IM C3010** (A3 MFP) | Yes (SMTP auth or POP-before-SMTP). Folder via SMB/FTP. | Exchange Online OAuth by firmware (from July 2025) | — | PDF/A. **Searchable PDF needs optional OCR unit.** | Vendor claim | [S79][S80] |
| **PFU/Ricoh ScanSnap iX1600/iX2500** | Email without a PC only via Ricoh's hosted **ScanSnap Mail** relay: **30 MB max, 10 addresses, 300 mails/month**. Own SMTP not supported. iX2500 gained PC-free scan to network folder (Oct 2025). | n/a | 30 MB | — | Verified fact / Vendor claim | [S81][S82] |

**Mail-provider constraints on the device's SMTP hop:**

| Provider | Device options | Size limit | Label | Src |
|---|---|---|---|---|
| **Microsoft 365** | Client SMTP submission (`smtp.office365.com:587`, TLS 1.2+, licensed mailbox, 10,000 recipients/day, 30 msgs/min). SMTP relay via connector (static IP or certificate, port 25). Direct Send (internal recipients only, so useless for sending to the DMS). High Volume Email (internal only). **Basic-auth SMTP AUTH: unchanged to Dec 2026, disabled by default for existing tenants end-Dec 2026 (re-enable possible), unavailable for new tenants after Dec 2026, final removal date announced H2 2027.** SMTP AUTH is off for orgs created after Jan 2020 and under Security Defaults. | Default **35 MB send / 36 MB receive**, admin-configurable 1–150 MB. ~33% encoding overhead when leaving Microsoft. | Verified fact | [S1][S2][S3] |
| **Google Workspace** | SMTP relay (`smtp-relay.gmail.com`, IP-authenticated, 10,000 recipients/user/day). Gmail SMTP (`smtp.gmail.com` 465/587, **app password**, 2,000 msgs/day). Restricted server (`aspmx.l.google.com:25`, no TLS/auth, Google recipients only, so it can't reach the DMS). Since **1 May 2025** username/password-only sign-in is unsupported. | **25 MB** sending (Business editions, before encoding) | Verified fact | [S4][S84] |
| Personal Gmail | — | 25 MB | Verified fact | [S5] |

**Inference:** a 30-page colour scan can exceed 25 MB. Devices that split mail (Canon "Divide Data") produce fragments the DMS would have to re-merge. **Recommendation:** tell tenants to scan B/W or greyscale at 200–300 dpi for email, and point large-batch users to SFTP or upload. Real per-page sizes should be measured during the pilot (Open question).

---

## 2. Receiving email into the Laravel app

### 2.1 Laravel side

| Item | Finding | Label | Src |
|---|---|---|---|
| `beyondcode/laravel-mailbox` | Latest **6.0.0 (2026-03-20)**, supports Laravel 10–13, repo not archived. Drivers in `src/Drivers`: **Mailgun, Postmark, SendGrid, MailCare, Log**. **No SES driver.** Parses MIME with `zbateson/mail-mime-parser`. | Verified fact | [S21] |
| SES with Laravel | No package driver, so SES → S3 → SNS (HTTPS) → Laravel controller needs custom glue: fetch the raw MIME from S3 and parse it. | Inference | [S11] |
| First-party Laravel inbound mail | None found. Laravel's mail features are outbound. | Open question (not exhaustively checked) | — |

### 2.2 Inbound-email services

| Service | Price (official) | Max inbound size | Per-tenant addressing | Auth/spam signals given to the app | Src |
|---|---|---|---|---|---|
| **Amazon SES receiving** | **$0.10 / 1,000 emails + $0.09 / 1,000 incoming 256 KB chunks.** New AWS accounts get up to $200 Free Tier credits. | **40 MB** incl. headers via S3 action; only 150 KB via SNS action | Receipt rules match domains, sub-domains or addresses; a catch-all on `scan.example.ph` is possible. Recipient = SMTP envelope `RCPT TO`. | `spfVerdict`, `dkimVerdict`, `dmarcVerdict` (+`dmarcPolicy`), `spamVerdict`, `virusVerdict` (PASS/FAIL/GRAY/PROCESSING_FAILED). Scan is on per rule (`ScanEnabled`). `TlsPolicy: Require` bounces non-TLS mail. Optional KMS encryption to S3. IP allow/block filters. | [S10][S11][S12][S14] |
| SES region | Receiving endpoint exists in **Asia Pacific (Singapore) ap-southeast-1** (`inbound-smtp.ap-southeast-1.amazonaws.com`). Not in Malaysia (ap-southeast-5). | — | — | — | [S13] |
| **Postmark Inbound** | Inbound is listed on **Pro ($16.50/mo, from 10k emails)** and **Platform ($18/mo)**, not Basic ($15). Each processed inbound message counts as 1 email toward volume. Overage $1.80/1k (Basic), $1.30/1k (Pro), $1.20/1k (Platform). | **35 MB** total attachments | Plus-addressing → `MailboxHash` field. Inbound domain forwarding by MX to `inbound.postmarkapp.com`, incl. wildcard `*.yourdomain` sub-domains. | `Received-SPF` header (pass/softfail/fail/neutral). SpamAssassin `X-Spam-Score/Status/Tests`. **No dedicated DKIM/DMARC field.** | [S6][S7][S8][S9] |
| **Mailgun Routes** | Free $0 (100/day, **1 inbound route**). Basic from $15/mo (5 routes). Foundation $35/mo (full inbound routing). Scale $90/mo. | **Inbound limit not documented.** 25 MB is the documented *sending* limit. A user report says >25 MB inbound is lost. | Routes with `forward("https://…")` to a webhook. `store()` keeps messages up to 3 days. | Webhook posts `message-headers` JSON. Dedicated SPF/DKIM fields not confirmed on the current docs page. | [S17][S18][S19][S20] |
| **SendGrid Inbound Parse** | Free **trial 60 days** (100/day). Essentials from $19.95/mo. Pro from $89.95/mo. Per-plan Inbound Parse availability not shown in extracted text. | Docs recommend ≤ **30 MB** total. Spam check only ≤ 2.5 MB. | Dedicated host/sub-domain with MX `mx.sendgrid.net`. Local part can't be abuse/postmaster/unsubscribe. | `SPF`, `dkim` fields always. `spam_score` if enabled. | [S15][S16] |

**Per-tenant address schemes (Inference):**
- `<tenant-token>@scan.<dms-domain>` (catch-all sub-domain; works on SES, Postmark wildcard, Mailgun, SendGrid). Prefer a long random token, not the tenant name, so addresses can't be guessed.
- `scan+<tenant-token>@in.<dms-domain>` (Postmark `MailboxHash`). **Open question:** do all scanner address-book fields accept `+`? Test on pilot devices.
- Allow the tenant to *rotate* the token if the address leaks.

---

## 3. Scan to network folder / FTP / SFTP

| Device | SMB | FTP | FTPS | **SFTP** | Other push | Label | Src |
|---|---|---|---|---|---|---|---|
| Epson WF/DS network models | Yes | Yes | **Yes** (DS guide: can validate server certificate) | **Not found** | Epson Connect cloud | Verified fact (excerpt) | [S60][S63] |
| Brother ADS-4300N, MFC-L5915DW, MFC-L2885DW family | Yes (CIFS) | Yes | — | **Yes ("SSH Server (SFTP)")** | SharePoint | Verified fact | [S69][S70] |
| Canon iR ADV DX | Yes (SMB 3.1.1) | Yes | — | **Not found** | **WebDAV (http/https)** | Vendor spec / Verified fact | [S36][S72] |
| Canon ScanFront 400 | Yes | Yes | Yes | **Yes** | SMTPS | Vendor spec (excerpt) | [S75] |
| HP LaserJet Pro 4103fdw | Not confirmed | — | — | Not found | — | Open question | [S76] |
| HP Enterprise | Yes | Yes (forum) | — | Not found | SharePoint/OneDrive via Entra app (FutureSmart ≥ 5.8, third-party guide) | Anecdotal | [S77] |
| Ricoh IM C3010 | Yes | Yes | — | **Not found** | — | Vendor claim | [S79] |

**Inference:** SFTP push to a cloud endpoint is possible today only for Brother business models and some Canon network scanners in the sample. Most MFPs offer only SMB, which can't reach a cloud host safely over the internet, and FTP.

**Cloud-side options for "scan to folder":**

| Pattern | What it means | Cost / burden | Label |
|---|---|---|---|
| **Per-tenant SFTP endpoint in the cloud** (device pushes straight to the DMS) | Only works for SFTP-capable devices. Per-tenant user with key or password, chroot to an S3/disk prefix, DMS ingests new files. | AWS Transfer Family: **$0.30/hour per protocol per endpoint (≈ $216/30-day month) + $0.04/GB** (US East example) [S23]. One endpoint can serve many tenant users. Self-hosted OpenSSH on an existing VM costs less but adds patching and monitoring for the vendor. | Verified fact (price); Inference (design) |
| **Cloud FTP endpoint** | Would accept the many FTP-only devices. | **Not recommended:** FTP sends passwords and all data unencrypted [S22], which conflicts with NPC 2023-06 §11 "adequately protected" [S24]. | Verified fact + Inference |
| **Local "watch folder" agent** (device → SMB share on a client PC/NAS → agent uploads over HTTPS) | Works with every SMB-capable device. | Exactly what the owner wants to avoid: an installer per site, auto-update, service crashes, Windows updates, SMB credentials on the device, antivirus quarantines, someone has to notice when it stops. **Partial alternative:** the share is a OneDrive/Google Drive-synced folder (client already maintains the sync client) → DMS pulls via Graph/Drive API (see §4). | Inference |

---

## 4. Manufacturer cloud connectors

| Vendor / service | Destinations | Push to a custom URL / public API? | Cost / availability | Label | Src |
|---|---|---|---|---|---|
| **Epson Connect "Scan to Cloud"** | Registered email addresses or cloud storage. Up to 60 destinations and 10 accounts per service. **Not** Google Shared drives, "Shared with me", Dropbox Team Space or OneDrive "Shared". Some models unsupported. | — | No price or end-of-service notice on the guide | Verified fact | [S25] |
| Epson ScanSmart | Google Drive, Dropbox, Evernote, OneDrive. It is **PC software**, so it needs an install. | — | — | Verified fact | [S26] |
| **Epson Connect API** | Web API using OAuth 2.0 and an Epson Global ID. It can "send scanned data directly to cloud storage without a driver", with endpoints to get, register, update and delete scan destinations. v1.3 is being replaced by v2.0. | Destination types reportedly `mail` and `url`. **Whether a `url` destination receives an HTTP POST, the payload format, terms and PH availability are unconfirmed** (official reference returned 403). | Unknown | Verified fact (API exists); Anecdotal (`url` type, from unofficial wrapper); Open question | [S27][S28] |
| **Brother Web Connect** | Google Drive, Dropbox, OneDrive, Box, SharePoint Online, Evernote, OneNote. Not all in every country. | — | Free with device | Verified fact | [S29] |
| Brother "Scan to SharePoint" profile | Configured in the device web UI with a site URL and optional HTTPS certificate check. **Open question:** does it work with SharePoint Online modern auth? | — | — | Verified fact / Open question | [S30] |
| **Brother BSI** (Brother Solutions Interface) | Server (cloud or on-prem) drives devices with XML. Destination can be "email, FTP, Network, SharePoint or your own web service". | **Yes**, but proprietary. Developer program needs approval and a licence agreement. | Cost and supported PH models unknown | Vendor claim | [S31] |
| **HP Smart Tasks / Shortcuts** | Email, print, or cloud "such as Google Drive or Dropbox". Needs an HP account and the HP Smart app. Users report OneDrive save failures. | No | HP+ requirement unknown | Verified fact + Anecdotal | [S32] |
| **HP Workpath** | App platform for **Enterprise** devices. Apps reach devices only through a channel/print service provider. No side-loading. | SDK via developer account | Not SME-priced | Vendor claim (page 403) | [S33] |
| **Canon uniFLOW Online** | Canon PH lists Google Drive, Box, SharePoint Online, OneDrive/OneDrive for Business, Dropbox, Evernote, Concur. SaaS subscription per device per month. **uniFLOW Online Express** is free on imageRUNNER ADVANCE (DX) ("Scan to Myself", "Scan to Google Drive"). | No public API found | Prices not published | Vendor claim | [S34][S35] |
| Canon iR-ADV **WebDAV send** | The File destination can be a WebDAV host over http:// or https://. **Open question:** does it accept any HTTPS WebDAV server (e.g. one run by the DMS)? If yes, this is a no-install push channel for Canon fleets. | Possibly (WebDAV) | — | Verified fact / Open question | [S36] |
| **PFU/Ricoh ScanSnap Cloud** | Box, Dropbox, Evernote, Google Drive, Google Photos, OneDrive (varies by region). Needs a ScanSnap account. **The Philippines is on the supported-country list.** | A 2016 SDK let providers build a "dedicated connector". Current status unknown. | Free with device | Verified fact (PH) / Vendor claim / Open question | [S37][S38] |
| Ricoh RSI connectors | OneDrive for Business, SharePoint, Google Drive, Box, Dropbox. Subscription. | No | Subscription; PH price unknown | Vendor claim | [S39] |
| **Ricoh CloudStream Generic HTTPS connector** | Sends all scan files and parameters as one multipart/form-data POST to *your* HTTPS endpoint, with optional Bearer token. Needs a publicly trusted certificate. | **Yes (documented)** | PH/APAC availability and cost unknown | Verified fact / Open question | [S40] |
| eSCL / AirScan (Mopria/Apple) | Pull-based: a client **on the LAN** fetches capabilities and POSTs `/ScanJobs`. A cloud server can't reach the device without a local component. | — | Spec free | Verified fact / Anecdotal (flow) | [S41] |

**Portable pattern: scan to the client's cloud drive, DMS pulls it** (Inference, built on Verified facts):
- Every major brand supports scan-to-Google Drive or OneDrive/SharePoint.
- **Google Drive** push notifications (`changes.watch`/`files.watch`) need an HTTPS webhook. They carry headers only, so the app must call `changes.list`. Channels expire (≤1 day for files, ≤1 week for changes) and must be renewed [S42].
- **Microsoft Graph** driveItem subscriptions on OneDrive for Business cover **the root only**. Subscriptions last under 30 days (42,300 min). Latency averages under 1 minute but can reach 6 hours. Pair with delta query [S43][S44].
- So the DMS needs per-tenant OAuth consent, subscription renewal jobs and a polling fallback. That is vendor-side maintenance, not client-PC maintenance.

---

## 5. Browser-initiated scanning (a "Scan" button in the DMS)

Browsers cannot talk to TWAIN/WIA/ICA/SANE drivers, so every option needs a local service that the web page calls over localhost or the LAN.

| Product | Local component | Licensing / price | Label | Src |
|---|---|---|---|---|
| **Dynamsoft Dynamic Web TWAIN** | "Dynamic Web TWAIN Service" on **every** end-user PC (Win 7+, macOS 10.15+, Linux). Uses TWAIN/WIA, ICA or SANE drivers. Runs as LocalSystem on Windows. | **Per end-user computer, annual** (tiers such as up to 25 / up to 100 PCs). Prices masked or quote-based. Per-app, unlimited and perpetual licences on quote. 30-day trial. A reseller figure of ≈ $1,273/yr for 25 PCs could not be opened. | Verified fact (model) / Anecdotal (price) | [S45] |
| DWT **Remote Scan** (v18+) | No install on the user's device. The service runs on each **scanner host PC** plus a "Dynamsoft Proxy Service" on a host. Works with any browser, including phones. | Not published | Vendor claim | [S46] |
| **Asprise Scanner.js** | Local "Scan App" (EXE; MSI only in Enterprise). Updates pushed to users. | **Per website/domain**, tiered by site traffic. Chrome/Edge support needs an **active support subscription**. No public prices. | Verified fact (terms) | [S49] |
| **VintaSoft Web TWAIN** | Local "VintaSoft Web TWAIN service" on the scanner PC (Windows/Linux). | Windows: Single URL **€189.95**, Developer €199.95, Site €599.95. Linux Single URL €189.95. Renewal ≈ 50%/yr (snippet). | Verified fact (prices) | [S50] |
| **NAPS2** | Desktop app (GPL-2.0+; SDK LGPL-2.1+). Can share scanners as an **eSCL server**, but only while the app is running (not as a service). No browser API. | Free | Verified fact | [S51] |

- **New browser friction (Verified fact):** Chrome's **Local Network Access** makes a public site's requests to localhost/private IPs need a user permission prompt [S48]. Dynamsoft documents breakage on Chromium 142+ and Firefox 153+. Admins can pre-allow sites with the `LocalNetworkAccessAllowedForUrls` policy [S47]. This affects **every** localhost-agent product.
- **Maintenance (Inference):** an installer and update cadence per PC, localhost certificate or service failures ("service not found"), OS and driver updates, and browser permission changes. This is the per-PC support burden the owner wants to avoid. It is suitable only as a paid add-on for firms that insist on it.

---

## 6. Phone capture

| Option | Facts | Cost | Label | Src |
|---|---|---|---|---|
| Camera in the browser | `getUserMedia` works only in a secure context (HTTPS). Iframes need Permissions-Policy `camera`. | — | Verified fact | [S52] |
| **jscanify** | MIT. v1.4.3 (2026-07-20). Edge detection and perspective correction on top of **OpenCV.js** (Apache-2.0). `opencv.js` 4.7.0 is ≈ 9 MB uncompressed, so it should be lazy-loaded. | Free | Verified fact | [S53][S54] |
| Dynamsoft Mobile Document Scanner / Mobile Web Capture | Edge detection, normalisation, multi-page capture | **Up to 50 browser clients: 1,299/yr; up to 200: 2,999/yr** (currency not stated). Others on quote. 30-day trial. | Verified fact (page) / Open question (currency, "client" definition) | [S55] |
| Scanbot SDK (Web) | Flat annual fee, unlimited devices, quote per app/domain. "Generally starts at $2,500". 7-day trial. | ≈ $2,500/yr+ | Vendor claim | [S56] |
| **Native OS scanners + plain upload** | iPhone/iPad Notes "Scan Documents" (auto-capture, multi-page) [S57]. Android Google Drive scan (crop/clean, saves PDF or JPG) [S58]. The DMS needs only `<input type="file">` or a share target. | Free | Verified fact | [S57][S58] |

**Quality versus ADF/flatbed:**
- No authoritative "phone = N dpi" standard was found.
- One imaging.org archiving paper reports a smartphone reaching roughly FADGI 3-star on ≤A4 documents under controlled conditions [S59]. **Anecdotal/academic**
- Phones need one capture per page, while an ADF batches stacks unattended. **Inference:** phones suit receipts and field documents. Bulk intake should stay on the office ADF.

---

## 7. Data Privacy Act / security for email ingestion

| Point | Detail | Label | Src |
|---|---|---|---|
| **NPC Circular 2023-06 §23 Emails** | "A PIC or PIP that transfers personal data by email must ensure that the data is adequately protected and use secure transmission and reception of email messages, including attachments." | Verified fact (text of signed Circular, dated 01 Dec 2023, retrieved via Internet Archive copy of privacy.gov.ph PDF because privacy.gov.ph returned 403) | [S24] |
| §10 Service provider as PIP | When a PIC engages a service provider to store personal data, the provider acts as a PIP. The PIC must ensure the PIP has appropriate security measures. The DMS vendor, and any inbound-mail provider it uses (sub-processor), fall here. | Verified fact | [S24] |
| §11 / §16 | Data must be "adequately protected through industry standards". Secure authentication (MFA or encrypted links) for online access to sensitive personal information. | Verified fact | [S24] |
| §25 Removable media | USB transfer must be encrypted if unavoidable, which also bears on "scan to USB stick" workflows. | Verified fact | [S24] |
| §26 Fax machines | "Facsimile technology shall not be used for transmitting documents containing personal data." Do not offer scan/fax-to-DMS. | Verified fact | [S24] |
| Hop 1: device → SMTP server | Encrypted only if the device uses STARTTLS/TLS. Microsoft 365 SMTP AUTH requires TLS 1.2+ [S1]. Google's restricted server (`aspmx.l.google.com:25`) needs **no TLS** [S4], which is weak. | Verified fact + Inference | [S1][S4] |
| Hop 2: mail server → DMS inbound MX | SES can **require TLS** per receipt rule (`TlsPolicy: Require`) [S14]. **Open question:** whether Postmark/Mailgun/SendGrid inbound can enforce TLS. | Verified fact (SES) | [S14] |
| Spoofing | Anyone who learns a tenant's scan address can email documents into the DMS. Mitigations: random per-tenant token, a per-tenant sender allowlist, reject on DMARC FAIL, and ideally **SMTP AUTH on a vendor-run submission endpoint** so the device itself is authenticated. Note that scanner mail relayed through the client's tenant may lack DKIM, so treat SPF/DKIM verdicts as signals, not proof. | Inference | [S12][S7] |
| Untrusted attachments | Accept only allowlisted MIME types (PDF/TIFF/JPEG). Verify magic bytes. Size cap. Virus scan (SES `virusVerdict`, or own ClamAV-type scan for other providers). Rasterise/sanitise PDFs before preview. Quarantine anything failing. | Inference | [S12] |
| Retention at providers | Raw emails sit in S3 (SES) or for up to 3 days at Mailgun (`store()`). Set lifecycle deletion and list providers as sub-processors. | Verified fact (Mailgun 3 days) + Inference | [S20] |

---

## 8. Comparison

| Channel | Client-side setup | Vendor maintenance burden | Security risks | Cost (vendor side) | Fit for PH SME |
|---|---|---|---|---|---|
| **Scan-to-email → per-tenant inbound address** | One-time device web-console config: SMTP server + DMS address in address book. No PC install. | Low: inbound provider + webhook + parser. Support calls for SMTP setup and M365/Google auth changes. | Spoofed senders, guessable address, malware attachments, TLS on device hop not guaranteed, size limits (25–36 MB on mail tenant) | SES ≈ $0.10/1k msgs + $0.09/1k 256 KB chunks; Postmark Pro from $16.50/mo | **High.** Works on nearly every MFP already in the office. |
| ↳ via **vendor-run SMTP submission endpoint** (device authenticates to the DMS, not the client's M365/Google) | Same device config, but with DMS-issued SMTP credentials | Medium: vendor runs and patches a submission MTA (port 587/465, TLS) | Fewer: device authenticated per tenant. Credential stored on device. | VM + ops time (Inference) | **High.** Removes the M365 Basic-auth cliff (end-Dec 2026). |
| **Scan to SFTP (cloud endpoint)** | Device config only (host, user, key/password) | Medium: SFTP service, per-tenant users, ingest watcher | Low if SFTP. Credential on device. Exposed port 22 must be hardened. | AWS Transfer Family ≈ $216/mo per endpoint + $0.04/GB, or self-host | **Medium.** Only SFTP-capable devices (Brother, some Canon). |
| Scan to FTP (cloud) | Device config | Medium | **High:** clear-text credentials and data | Low | **Avoid** (NPC §11/§23 spirit) |
| SMB folder + local sync agent | Install agent on a PC/NAS | **High:** per-site installs, updates, monitoring | Agent compromise, SMB credentials on device | Dev + support time | Low (owner's stated constraint) |
| **Manufacturer cloud → client's OneDrive/Google Drive → DMS pulls** | Register device to the vendor cloud and the client's drive account | Medium: OAuth per tenant, webhook subscription renewal, delta sync | OAuth token scope. Data passes through a third-party cloud (sub-processor). | API usage free (Inference); uniFLOW/RSI subscriptions if used | Medium. Good for firms already on M365/Workspace. |
| Manufacturer push to custom URL (Epson Connect API, Ricoh CloudStream, Brother BSI) | Device registered to the vendor platform | Medium–high: one integration **per brand** | Depends on vendor | Unknown / partner programs | Low for MVP (brand-specific, unverified in PH) |
| **Browser "Scan" button** (Dynamsoft/Asprise/VintaSoft) | Install local service on every scanning PC. Browser LNA permission. | **High** | Local service with broad privileges; localhost certs | VintaSoft from €189.95; Dynamsoft per-PC annual (quote) | Low–medium (paid add-on only) |
| **Phone: native scanner + upload** | None | Very low | Personal devices hold copies (NPC §17 authorised devices) | Free | **High** for ad-hoc docs |
| Phone: in-browser capture (jscanify / commercial SDK) | None | Medium (jscanify) / low (SDK) | Same as above | Free / ≈1,299–2,999/yr / ≈$2,500+/yr | Medium (nice-to-have) |
| Manual web upload / drag-drop (baseline) | None | Very low | Standard upload validation | — | High (always needed) |

---

## 9. Recommended capture channels for an MVP, and why — **Recommendation**

1. **Scan-to-email to a per-tenant, token-based address (primary "scan → appears in DMS" channel).**
   - **Why:** it's the only channel that reaches nearly all installed MFPs with zero client-PC software and one-time device configuration.
   - **Inbound provider:** **Amazon SES receiving in ap-southeast-1** (S3 action, `TlsPolicy: Require`, `ScanEnabled: true`). It is the cheapest option, takes 40 MB, and gives SPF/DKIM/DMARC/spam/virus verdicts. Accept the cost of writing a small S3+SNS→Laravel adapter.
   - **Alternative:** Postmark Pro, if speed of delivery matters more. It has a ready `laravel-mailbox` driver, but no DKIM/DMARC fields and a 35 MB limit.
   - **Controls:** per-tenant sender allowlist, a "quarantine for review" state for unknown senders, attachment type/size checks, and an audit-trail entry per ingest.
2. **Plan, from day one, a vendor-run authenticated SMTP submission endpoint ("DMS SMTP")** with per-tenant credentials.
   - **Why:** Microsoft 365 Basic-auth SMTP AUTH becomes disabled by default at end-Dec 2026 and is not available to new tenants after that [S2]. Google requires app passwords or OAuth [S4]. Many SME devices won't do OAuth.
   - With DMS SMTP the scanner never depends on the client's mail tenant, and the device itself is authenticated, which beats SPF/DMARC heuristics.
   - **Open question:** do PH ISPs block outbound 587/465 from office lines?
3. **Web upload plus native phone scanning (iOS Notes / Google Drive scan → share/upload).**
   - **Why:** zero licence cost, no SDK, and it covers field and ad-hoc documents. Add jscanify-based in-browser capture later only if users ask for it.
4. **Phase 2:** **SFTP ingest** for tenants with SFTP-capable devices (Brother business line), and **OneDrive/Google Drive folder watch** for M365/Workspace firms.
5. **Not in MVP:** browser TWAIN agents, FTP, local sync agents, and per-brand cloud APIs. They are either per-PC maintenance, insecure, or brand-specific and unverified in the Philippines.

---

## 10. Open questions

1. Do the scanners actually installed at target firms support SMTP OAuth, SFTP, or `+` in addresses? Run a **device census in the pilot** (brand/model/firmware).
2. Real file sizes per page (B/W vs colour, 200/300 dpi) on pilot devices, to set email size guidance.
3. Do PH ISPs (PLDT, Globe, Converge) block outbound SMTP ports 25/587/465 on business lines?
4. Epson Connect API `url` destination: payload, terms, PH availability (docs.epsonconnect.com returned 403).
5. Canon WebDAV send to an arbitrary HTTPS WebDAV server, as a possible no-install push for Canon fleets.
6. Can Postmark, Mailgun or SendGrid enforce TLS on inbound? What is Mailgun's documented inbound size limit?
7. Brother BSI and Ricoh CloudStream availability and cost in the Philippines.
8. Brother ADS-4300N standalone email. HP 4103fdw network folder and SharePoint. Epson L6490 network scan.
9. Exact Dynamsoft Web TWAIN and Remote Scan prices (masked or quote-only).
10. Will Microsoft announce the final SMTP AUTH Basic removal date (H2 2027) earlier than expected? Re-check before launch.

---

## 11. Sources (all accessed 2026-10-18)

**Email, SMTP and inbound services**
- [S1] Microsoft Learn, "How to set up a multifunction device or application to send email using Microsoft 365": https://learn.microsoft.com/en-us/exchange/mail-flow-best-practices/how-to-set-up-a-multifunction-device-or-application-to-send-email-using-microsoft-365-or-office-365 ; SMTP AUTH enable/disable: https://learn.microsoft.com/en-us/exchange/clients-and-mobile-in-exchange-online/authenticated-client-smtp-submission
- [S2] Microsoft Exchange Team blog, "Updated Exchange Online SMTP AUTH Basic Authentication Deprecation Timeline": https://techcommunity.microsoft.com/blog/exchange/updated-exchange-online-smtp-auth-basic-authentication-deprecation-timeline/4489835
- [S3] Microsoft Learn, Exchange Online limits: https://learn.microsoft.com/en-us/office365/servicedescriptions/exchange-online-service-description/exchange-online-limits
- [S4] Google Workspace Admin Help, "Send email from a printer, scanner, or app": https://knowledge.workspace.google.com/admin/gmail/send-email-from-a-printer-scanner-or-app
- [S5] Gmail Help, attachments: https://support.google.com/mail/answer/6584
- [S6] Postmark pricing: https://postmarkapp.com/pricing
- [S7] Postmark, parse an inbound email: https://postmarkapp.com/developer/user-guide/inbound/parse-an-email
- [S8] Postmark, "How does monthly pricing work?": https://postmarkapp.com/support/article/1107-how-does-monthly-pricing-work
- [S9] Postmark, inbound domain forwarding: https://postmarkapp.com/developer/user-guide/inbound/inbound-domain-forwarding
- [S10] Amazon SES pricing: https://aws.amazon.com/ses/pricing/
- [S11] Amazon SES, email receiving concepts: https://docs.aws.amazon.com/ses/latest/dg/receiving-email-concepts.html
- [S12] Amazon SES, receiving notification contents: https://docs.aws.amazon.com/ses/latest/dg/receiving-email-notifications-contents.html
- [S13] AWS General Reference, SES endpoints (email receiving): https://docs.aws.amazon.com/general/latest/gr/ses.html
- [S14] Amazon SES API, ReceiptRule: https://docs.aws.amazon.com/ses/latest/APIReference/API_ReceiptRule.html
- [S15] Twilio SendGrid, Inbound Parse webhook: https://www.twilio.com/docs/sendgrid/for-developers/parsing-email/setting-up-the-inbound-parse-webhook
- [S16] Twilio SendGrid Email API pricing: https://www.twilio.com/en-us/pricing/email-api
- [S17] Mailgun pricing: https://www.mailgun.com/pricing/
- [S18] Mailgun, receive via HTTP: https://documentation.mailgun.com/docs/mailgun/user-manual/receive-forward-store/receive-http
- [S19] Mailgun, sending via HTTP (25 MB): https://documentation.mailgun.com/docs/mailgun/user-manual/sending-messages/send-http ; user report on >25 MB inbound (Anecdotal): https://feedback.mailgun.com/forums/156243-feature-requests/suggestions/49292069-process-emails-with-size-limit-25mb
- [S20] Mailgun, route actions: https://documentation.mailgun.com/docs/mailgun/user-manual/receive-forward-store/route-actions
- [S21] beyondcode/laravel-mailbox: https://github.com/beyondcode/laravel-mailbox ; Packagist metadata: https://repo.packagist.org/p2/beyondcode/laravel-mailbox.json
- [S84] Google Workspace, Gmail sending limits: https://knowledge.workspace.google.com/admin/gmail/gmail-sending-limits-in-google-workspace

**Security, privacy, SFTP**
- [S22] RFC 2577, FTP Security Considerations, §5–6: https://www.rfc-editor.org/rfc/rfc2577.html
- [S23] AWS Transfer Family pricing: https://aws.amazon.com/aws-transfer-family/pricing/
- [S24] NPC Circular 2023-06, "Security of Personal Data in the Government and the Private Sector" (dated 01 Dec 2023), §§10, 11, 16, 17, 23, 25, 26. Official URL https://privacy.gov.ph/wp-content/uploads/2024/03/NPC-Circular-Repeal-16-01-Signed.pdf (403 to automated access); read via https://web.archive.org/web/2024id_/https://privacy.gov.ph/wp-content/uploads/2024/03/NPC-Circular-Repeal-16-01-Signed.pdf

**Manufacturer cloud connectors and cloud-drive APIs**
- [S25] Epson Connect guide, Scan to Cloud: https://www.epsonconnect.com/guide/en/html/uses_2.htm
- [S26] Epson ScanSmart guide: https://download4.epson.biz/sec_pubs/ScanSmart/p/en/GUID-D0A4031A-F8B5-4FE9-848C-30791E6D4CAE.htm
- [S27] Epson Connect API developer portal: https://developer.epsonconnect.com ; tutorial https://developer.epsonconnect.com/Portals/tutorial
- [S28] Unofficial Epson Connect Python wrapper (Anecdotal): https://epson-connect-api.readthedocs.io/en/main/source/epson_connect.html
- [S29] Brother Web Connect services: https://support.brother.com/g/s/id/htmldoc/mfc/mfc6959dw/uke/html/GUID-2304AEFE-088F-44D6-8A7F-70A092610BF1_1.html
- [S30] Brother Scan to SharePoint: https://support.brother.com/g/s/id/htmldoc/mfc/cv_mfcj6960dw/use/html/GUID-62E3BF08-554B-4504-B88F-3DEB50D994B4_1.html
- [S31] Brother BSI developer program: https://developerprogram.brother-usa.com/bsi-developer-program ; https://atyourside.brother.co.uk/developer-centre/bsi
- [S32] HP Smart Tasks: https://www.support.hp.com/lt-en/document/ish_4129352-4129404-16 ; HP community (Anecdotal): https://h30434.www3.hp.com/t5/Scanning-Faxing-Copying/HP-Smart-Task-Won-t-save-to-OneDrive/m-p/7549351
- [S33] HP Workpath developers: https://developers.hp.com/workpath
- [S34] Canon Philippines, uniFLOW Online: https://ph.canon/en/business/uniflow-online/main/product
- [S35] Canon NZ, uniFLOW Online Express: https://canon.co.nz/other-products/uniflow-online-express ; trial https://canon.co.nz/business/uniflow-online-3-month-free-trial
- [S36] Canon iR-ADV manual, WebDAV send: https://oip.manual.canon/USRMA-0110-zz-CS-enUS/contents/1T0001919099.html
- [S37] PFU ScanSnap Cloud supported countries: https://www.pfu.ricoh.com/global/scanners/scansnap/scansnap-cloud/country.html
- [S38] PFU ScanSnap Cloud: https://www.pfu.ricoh.com/global/scanners/scansnap/sscloud/ ; 2016 SDK press release: https://www.fujitsu.com/lu/about/resources/news/press-releases/2016/scan-snap-cloud.html
- [S39] Ricoh Smart Integration: https://www.ricoh.com.au/products/software/office-software/ricoh-smart-integration
- [S40] Ricoh CloudStream Generic HTTPS connector: https://manual.na.ps.cloudstream.ricoh.com/docs/generic-https-connector
- [S41] Mopria specifications (eSCL): https://mopria.org/specifications ; flow (Anecdotal): https://pwg.org/archives/ipp/2019/020121.html
- [S42] Google Drive API push notifications: https://developers.google.com/workspace/drive/api/guides/push
- [S43] Microsoft Graph change notifications: https://learn.microsoft.com/en-us/graph/change-notifications-overview
- [S44] Microsoft Graph delta query: https://learn.microsoft.com/en-us/graph/delta-query-overview

**Browser scanning and phone capture**
- [S45] Dynamsoft Dynamic Web TWAIN store: https://www.dynamsoft.com/store/dynamic-web-twain/ ; quote: https://www.dynamsoft.com/web-twain/ask-for-quote
- [S46] Dynamsoft Remote Scan: https://www.dynamsoft.com/remote-scan/docs/differences
- [S47] Dynamsoft, Chromium 142 Local Network Access FAQ: https://www.dynamsoft.com/web-twain/docs/faq/chromium-142-local-network-access-issue.html
- [S48] Chrome for Developers, Local Network Access: https://developer.chrome.com/blog/local-network-access
- [S49] Asprise Scanner.js licensing: https://asprise.com/document-scan-upload-image-browser/web-scanner-source-code-open-order.html
- [S50] VintaSoft pricing: https://www.vintasoft.com/register.html
- [S51] NAPS2 scanner sharing: https://www.naps2.com/doc/scanner-sharing ; repo https://github.com/cyanfish/naps2
- [S52] MDN getUserMedia: https://developer.mozilla.org/en-US/docs/Web/API/MediaDevices/getUserMedia
- [S53] jscanify: https://github.com/puffinsoft/jscanify
- [S54] OpenCV (Apache-2.0): https://github.com/opencv/opencv
- [S55] Dynamsoft Mobile Document Scanner pricing: https://www.dynamsoft.com/mobile-document-scanner/ask-for-quote/
- [S56] Scanbot SDK pricing: https://scanbot.io/pricing/
- [S57] Apple Support, scan documents in Notes: https://support.apple.com/108963
- [S58] Google Drive Help, scan documents: https://support.google.com/drive/answer/3145835
- [S59] IS&T Archiving paper on smartphone digitisation (Anecdotal/academic): https://library.imaging.org/archiving/articles/13/1/art00023

**Device documentation**
- [S60] Epson WF-C5890 scan destinations FAQ: https://epson.com/faq/SPT_C11CK23201~faq-0000603-wfc5810_c5890
- [S61] Epson mail server settings (EM-C800 guide, excerpt): https://download4.epson.biz/sec_pubs/em-c800bam/useg/en/GUID-0CF50114-406B-47D8-9B72-C8F1B81FAA53.htm
- [S62] Epson DS-785W OAuth (excerpt): https://download4.epson.biz/sec_pubs/ds-785w/useg/us/GUID-F5D4639B-E3F2-4894-9DF7-74AB22A39C62.htm ; https://download4.epson.biz/sec_pubs/common/oa2/en/NPD7717.pdf
- [S63] Epson DS-785W network folder/FTP/FTPS (excerpt): https://download4.epson.biz/sec_pubs/ds-785w/useg/us/GUID-80D01288-F98B-481D-BD33-B609EA7E14E3.htm
- [S64] Epson DS-1760WN max attachment size (excerpt): https://download4.epson.biz/sec_pubs/ds-1760wn/useg/us/GUID-1D4F7175-0D67-4E1B-8454-A64698938B59.htm
- [S65] Epson WF-C4810 searchable PDF FAQ: https://epson.com/faq/SPT_C11CJ05205~faq-0000680-wfc4810
- [S66] Epson DS-C490 FAQ: https://epson.com/faq/SPT_B11B271201~faq-0000707-dsc330_c490
- [S67] Epson DS-790WN brochure: https://download.epson.com.sg/product_brochures/scanner/EIN/EN/Epson_DS-790WN.pdf
- [S68] PC Express PH, Epson L6490 listing (Anecdotal): https://pcx.com.ph/collections/all/products/epson-ecotank-l6490-ink-tank-printer
- [S69] Brother ADS-4300N: https://www.brother-usa.com/products/ads4300n (also https://www.brother-usa.com/p/desktop-scanners/ADS4300N)
- [S70] Brother MFC-L5915DW: https://www.brother-usa.com/products/mfcl5915dw
- [S71] Brother "Send to Email Server" (ADS): https://support.brother.com/g/s/id/htmldoc/ads/cv_ads4300n/use/html/GUID-B88ECF4C-31E6-4011-AD1E-5674E909BD3F_41.html
- [S72] Canon iR ADV DX C3926i specifications (excerpt): https://www.canon.co.uk/business/products/office-printers/multifunction/colour/imagerunner-advance-dx-c3900-series/specifications/imagerunner-advance-dx-c3926i.html
- [S73] Canon, cloud mail server with OAuth 2.0: https://oip.manual.canon/USRMA-8299-zz-CS-719-enAU/contents/devu-setup-cloud.html
- [S74] Canon, Maximum Data Size for Sending: https://oip.manual.canon/USRMA-0395-zz-CS-enUV/contents/1T0001202348.html
- [S75] Canon ScanFront 400 specifications (excerpt): https://www.canon.co.uk/scanners/document-scanners/imageformula_scanfront_400/specifications/
- [S76] HP scan methods by printer family (excerpt): https://support.hp.com/th-en/document/ish_7279439-7279483-16
- [S77] HP FutureSmart Office 365 OAuth (excerpt): https://support.hp.com/us-en/document/c05920994 ; third-party guide (Anecdotal): https://www.loffler.com/hubfs/Microsoft%20365%20Training/HP%20Setting%20Up%20Scan%20To%20Email%20OAuth%20v2.0%20Authentication.pdf
- [S78] HP Singapore store, LaserJet Pro MFP 4103fdw: https://www.hp.com/sg-en/shop/hp-laserjet-pro-mfp-4103fdw-printer-2z629a.html
- [S79] Ricoh IM C3010: https://www.ricoh-ap.com/products/office-printing/mfp-colour/im-c3010
- [S80] Ricoh AP, enabling OAuth 2 on Ricoh products: https://www.ricoh-ap.com/support/enabling-oauth-2-authentication-for-ricoh-products
- [S81] ScanSnap Mail: https://www.pfu.ricoh.com/global/scanners/scansnap/sscloud/mail/
- [S82] PFU news 2025-10-15 (iX2500 update): https://www.pfu.ricoh.com/global/scanners/topics/news20251015-1/
- [S83] PaperStream NX Manager: https://www.pfu-us.ricoh.com/scanners/software/paperstream-nx-manager
- [S87] Canon iR 2630 spec sheet via search (Anecdotal; 403): https://www.cla.canon.com/bin/canon/cla/downloadasset/data.___content___dam___cla-assets___brochures___en___copier___iR2630_Spec_sheet.pdf
