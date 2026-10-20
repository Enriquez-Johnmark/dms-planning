# Open Questions

Questions that block a decision. Facts that research can find are research tasks, not entries here, unless research has already failed to find them.

**Owner:** who must answer. **Round:** the grilling round where it is asked.

| ID | Question | Why it matters | Owner | Round / source | Status |
|---|---|---|---|---|---|
| OQ-001 | Who is the first customer? | | | Round 1 Q1 | **Answered → D-001** |
| OQ-002 | What is the product for? | | | Round 1 Q2 | **Answered → D-002** (ERP module) |
| OQ-003 | ISO goal: certification is a **future goal**; which standard (ISO/IEC 27001, ISO 9001:2026, both) and when. App stays ISO-ready (D-011). | Section 04; claims we can make | Senior partner | Round 1 Q3 → Round 3 Q5 | **Future goal; senior partner decides; not blocking (D-028)** |
| OQ-004 | How the module is made | | | Round 3 Q1 | **Answered → D-010** (native build in Officetul) |
| OQ-005 | Which segment first? | | | Round 1 Q5 | **Answered → D-003, D-004** (all ERP clients; no interviews) |
| OQ-006 | Team, timeline, budget | | | Round 1 Q6 | **Answered → D-005** |
| OQ-007 | Existing evidence of demand | | | Round 1 Q7 | **Answered → D-006** |
| OQ-008 | How is the ERP's multi-tenancy implemented, and what are its auth, roles, file storage, queues and hosting? The ERP is multi-tenant Laravel; repository access comes after design. | Decides tenant isolation, file namespaces and search isolation in the DMS | Whoever maintains the ERP | Round 3 Q7 (questionnaire) | Asked |
| OQ-013 | MVP shape | | | Round 4 Q1 | **Answered → D-014** |
| OQ-014 | Scanner-to-DMS route | | | Round 4 Q2–Q3 | **Answered → D-015, D-016** (details pending R04) |
| OQ-015 | Paper originals | | | Round 2 Q4 | **Answered → D-008** |
| OQ-016 | Client samples and calls | | | Round 2 Q6 | **Answered → D-009** (not feasible) |
| OQ-017 | Do the ERP's clients already have office scanners or multifunction printers? | Decides whether we ever supply hardware | Owner | Round 3 Q4 | Unknown; mitigated by phone capture (D-016). Captured in the client document profile. |
| OQ-018 | Should the DMS run its own authenticated mail-submission service for scanners, or rely on clients' email accounts? | Microsoft 365 and Google are dropping password-only device mail; a service of our own is more to operate and secure | Owner (Phase 7 grilling) | R04 §0, §9 | Deferred to Phase 7 |
| OQ-019 | Do Philippine ISPs (PLDT, Globe, Converge) block outgoing mail ports on business lines? Which scanner models do pilot clients actually have? | Whether scan-to-email works on site | Pilot (device census) | R04 §10 | Pilot task |
| OQ-020 | Price the Documents app: included in the subscription or a separate add-on? How is Officetul priced today (per user, per company, tiers), and what margin is needed? | Revenue model; the one-stop pitch | Senior partner | Round 5 Q1 | Deferred to Section 18 (D-018) |
| OQ-021 | Tax adviser: can scans replace paper purchase invoices and 2307s; does RR 5-2014's e-storage standard survive; e-invoice retention; default review period of 5 or 10 years? | Claims we can make; paper-original defaults | Tax adviser | R07 §4 | Needs professional |
| OQ-022 | Lawyer: client-file retention and copies after turnover; notarial records custody; cloud hosting of privileged files; AMLA coverage; DOLE, PhilHealth and Pag-IBIG retention; digital logbooks | Law pack; retention defaults; privacy terms | Philippine lawyer | R07 §4, R08 §5 | Needs professional |
| OQ-023 | Research gaps: current PCAB cycle, RA 12009 IRR warranty and retention terms, COA retention for contractors, FSIC validity, official SC e-filing text | Construction and law packs | Research (later) | R07, R08 | Deferred to pack design |
| OQ-024 | Is the company already registered with the NPC (as PIC/PIP for Officetul), with a named DPO? Who owns the D-032 pre-launch privacy tasks? | Pre-launch gate for external clients | Senior partner / owner | Round 7 Q4 | Not answered |
| OQ-009 | Do scanned manual or loose-leaf books of accounts satisfy the BIR, given RR 7-2024's hard-copy rule and RA 8792 s.13? | Accounting-segment feature scope | Philippine lawyer / tax adviser | R01 §3a | Needs professional |
| OQ-010 | Does the CPRA set a minimum retention period for client files? | Law-firm retention rules | Philippine lawyer | R01 §3b | Needs professional |
| OQ-011 | How many Philippine organisations hold ISO 9001 and ISO 27001 certificates? | Sizes goal B | Senior partner (optional free CertSearch login) | R03 §8 | Future goal; senior partner decides (D-028) |
| OQ-012 | What would a Philippine certification body quote an SME of the partner's size for 9001:2026 or 27001:2022? | ISO goals A and C cost | Senior partner (request quotes) | R03 §8 | Future goal; senior partner decides (D-028) |
