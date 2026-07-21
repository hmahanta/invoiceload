<div align="center">

<p align="center">
  <img src="assets/gitbanner.png" width="100%">
</p>

# InvoiceLoad

### Enterprise Intelligent Document Processing (IDP) Platform

**AI-driven invoice understanding, extraction, quality assessment, and ERP-agnostic staging — engineered offline-first, with zero cloud OCR dependency and zero per-document API cost.**

[![Python](https://img.shields.io/badge/Python-3.11-3776AB?style=flat-square&logo=python&logoColor=white)](#)
[![SQLAlchemy](https://img.shields.io/badge/SQLAlchemy-2.x-D71F00?style=flat-square&logo=sqlalchemy&logoColor=white)](#)
[![Oracle](https://img.shields.io/badge/Oracle-XE%20%7C%20Enterprise-F80000?style=flat-square&logo=oracle&logoColor=white)](#)
[![OCR](https://img.shields.io/badge/OCR-Offline%20%7C%20PaddleOCR-2E7D32?style=flat-square)](#)
[![Deployment](https://img.shields.io/badge/Deployment-Air--Gapped%20Ready-4B0082?style=flat-square)](#)
[![Architecture](https://img.shields.io/badge/Architecture-ERP--Agnostic-0A66C2?style=flat-square)](#)
[![Formats](https://img.shields.io/badge/Formats-9%20Document%20Types-E67E22?style=flat-square)](#)
[![Status](https://img.shields.io/badge/Status-Production-2E7D32?style=flat-square)](#)
[![License](https://img.shields.io/badge/License-Internal%20Use-lightgrey?style=flat-square)](#)

**[Executive Summary](#-executive-summary) · [Architecture](#-platform-architecture) · [AI Capabilities](#-ai-capabilities) · [Use Cases](#-enterprise-use-cases) · [Quick Start](#-installation--quick-start)**

</div>

---

> [!NOTE]
> **On honesty:** every capability documented below is implemented and verified against the running source. Nothing here describes a roadmap item as a shipped feature. Where a boundary of scope exists (for example, native ERP posting connectors), it is stated explicitly rather than implied. See [Product Roadmap](#-product-roadmap--future-vision) for what's intentionally *not* built yet.

---

## 📌 Table of Contents

1. [Executive Summary](#-executive-summary)
2. [Why This Platform Exists](#-why-this-platform-exists)
3. [Enterprise Business Benefits](#-enterprise-business-benefits)
4. [AI Capabilities](#-ai-capabilities)
5. [Platform Architecture](#-platform-architecture)
6. [Processing Pipeline](#-processing-pipeline)
7. [Document Quality Assessment Engine](#-document-quality-assessment-engine)
8. [Supported Document Formats](#-supported-document-formats)
9. [The Knowledge Engine](#-the-knowledge-engine)
10. [Enterprise Validation & Business Rule Framework](#-enterprise-validation--business-rule-framework)
11. [Oracle Staging Framework](#-oracle-staging-framework)
12. [Downstream ERP Consumption](#-downstream-erp-consumption)
13. [Notification Framework](#-notification-framework)
14. [Operations Dashboard](#-operations-dashboard)
15. [Enterprise Design Principles](#-enterprise-design-principles)
16. [Security & Offline Deployment](#-security--offline-deployment)
17. [Extensibility](#-extensibility)
18. [Enterprise Use Cases](#-enterprise-use-cases)
19. [Performance & Benchmarking](#-performance--benchmarking)
20. [Engineering Highlights](#-engineering-highlights)
21. [Project Structure](#-project-structure)
22. [Configuration Reference](#-configuration-reference)
23. [Installation & Quick Start](#-installation--quick-start)
24. [Technology Stack](#-technology-stack)
25. [Why This Platform Is Different](#-why-this-platform-is-different)
26. [Product Roadmap & Future Vision](#-product-roadmap--future-vision)
27. [Lessons Learned](#-lessons-learned)
28. [FAQ](#-faq)
29. [About the Author](#-about-the-author)
30. [License](#-license)

---

## 🚀 Executive Summary

**InvoiceLoad** is an enterprise-grade Intelligent Document Processing platform purpose-built to solve one of the most persistent problems in Accounts Payable and Accounts Receivable operations: **invoices never arrive in a single, predictable shape.**

A typical AP shared-services function receives scanned paper, phone-camera photographs, born-digital PDFs, spreadsheets, and structured e-invoicing feeds (JSON/XML) from dozens — sometimes hundreds — of suppliers and EDI partners. Every one of those formats ultimately needs to become the same thing: a **clean, validated, quality-scored row** that a downstream ERP posting process can trust.

InvoiceLoad is that landing zone. It is a single, format-agnostic ingestion and staging platform built on nine core engineering pillars:

| Pillar | What it delivers |
|---|---|
| 🧠 **Knowledge Engine** | Business vocabulary (field aliases, currencies, tax IDs, vendor quirks) lives in versioned JSON, not hardcoded Python |
| 🔍 **Intelligent OCR** | Fully offline document recognition with layout-aware line-item reconstruction |
| 💱 **Currency Resolution Engine** | Multi-signal currency detection with conflict and confidence scoring |
| ⚖️ **Business Rule Engine** | Non-blocking, tolerance-based reconciliation of totals, tax, discounts, and freight |
| 🛡 **Enterprise Validation Framework** | Every document is captured — never silently rejected for a business-data gap |
| 🎯 **Document Quality Assessment Engine** | A dedicated, independent decision matrix (not inherited severities) determines final document quality |
| 🩹 **Automatic Field Recovery** | Missing mandatory fields are inferred and repaired, and that repair is itself tracked and reported |
| 📊 **Confidence Scoring** | Every extracted field carries a quantified certainty signal |
| 🗄️ **ERP-Agnostic Oracle Staging** | One staging schema, any downstream ERP — no vendor lock-in |

The result: an organization can point InvoiceLoad at a folder of mixed-format invoices and get back a fully staged, audit-trailed, quality-scored dataset — without a single document ever being silently dropped, and without a single byte leaving the network perimeter.

---

## 🎯 Why This Platform Exists

Every enterprise that processes vendor invoices eventually hits the same wall: invoices arrive in a dozen different shapes, from a dozen different systems of origin, and every one of them needs to land in the same place — a clean, validated row in an ERP staging table.

Most commercial IDP tools solve half of this problem. They're either:

- **Cloud-OCR-first**, which fails outright in regulated, air-gapped, or data-residency-sensitive environments (defense, government, healthcare, financial services), or
- **Format-narrow**, handling PDFs well but treating JSON/XML e-invoicing feeds, spreadsheets, and legacy scans as separate integration projects, or
- **Validation-as-gatekeeper**, where one missing optional field on an otherwise perfectly good invoice causes the whole document to be rejected and re-keyed manually.

InvoiceLoad's founding design philosophy — stated directly in its own service-layer architecture — is **"capture first, validate later."** One bad invoice should never stop a batch of ten thousand good ones, and a missing *business* field should never be treated the same as a *system* failure.

Concretely, this is enforced as a hard architectural rule: a business-data gap (a missing invoice date, an unparseable PO number, a tax field the resolver couldn't locate) is **never** a reason to reject a document. It is defaulted where safe to do so, logged as an auditable `INFO`/`WARNING` message, and the invoice still stages successfully for human or downstream review. `ERROR`-severity status is reserved exclusively for genuine technical failures — a database error, a malformed file, an unhandled exception — never for missing business data on an otherwise successfully captured invoice.

---

## 💼 Enterprise Business Benefits

| Business Pain Point | InvoiceLoad Capability | Business Outcome |
|---|---|---|
| AP teams manually re-key scanned and emailed invoices | Intelligent OCR + layout-aware line-item reconstruction | Eliminates manual data entry for scanned/photographed invoices |
| Every supplier format requires a new integration project | 9 format processors behind one router (JSON, XML, CSV, Excel, native PDF, scanned PDF, image, plus archive unpacking) | One platform onboards any supplier format without custom integration work |
| Data residency / regulatory constraints block cloud OCR tools | Fully offline OCR, fully offline Knowledge Engine, zero mandatory outbound network calls | Deployable in air-gapped, classified, or data-sovereign environments |
| Legacy validation tools reject entire invoices over one missing field | Capture-first architecture — business gaps are defaulted and flagged, never rejected | Zero data loss; every invoice reaches staging with a full audit trail |
| Finance leadership can't trust automated extraction without a quality signal | Document Quality Assessment Engine with a 4-state, auditable decision matrix | Every staged invoice carries a defensible, explainable quality verdict |
| Locking into one ERP's proprietary invoice format creates switching cost | ERP-agnostic staging schema (`inv_header_stg` / `inv_line_stg` / `inv_error_stg`) | Same pipeline feeds Oracle Fusion, SAP, PeopleSoft, Dynamics, or NetSuite without a rewrite |
| Currency mis-extraction on multi-national supplier invoices causes posting errors | Currency Resolution Engine with conflict/confidence scoring | Currency ambiguity is surfaced and reasoned about, not silently guessed |
| Extraction logic changes require a full software release cycle | JSON-driven Knowledge Engine — aliases, vendor overrides, tolerances are config, not code | New vendor, new field alias, new currency = a JSON edit, not a deployment |

---

## 🤖 AI Capabilities

InvoiceLoad's intelligence is deliberately built as a set of composable, independently-testable engines rather than one opaque extraction model — every decision a document goes through is explainable and auditable.

<table>
<tr><td width="50%" valign="top">

### 🔍 Intelligent OCR
Fully offline optical character recognition, decoupled from the extraction engine behind a stable service contract — no outbound network call once models are cached, no API key, no per-page billing.

### 📐 Layout-Aware Line-Item Reconstruction
Reconstructs invoice line-item tables from raw OCR/geometry data — correctly separating wrapped descriptions, multi-column layouts, and UOM-trailing rows instead of trusting flat reading order.

### 💱 Currency Resolution Engine
Detects the operative currency from symbols, codes, and document context; flags a **conflict** only when a genuinely competing candidate exists, with a **confidence score** (HIGH/MEDIUM/LOW) attached to every resolution.

</td><td width="50%" valign="top">

### 🧠 Knowledge Engine
A versioned, JSON-driven business-vocabulary layer (field aliases, regex libraries, UOM tokens, currency maps, vendor overrides, numeric tolerances) — loaded once, frozen, and never hardcoded into extraction logic.

### 🎯 Document Quality Assessment Engine
An independent, six-rule header decision matrix (H1–H6) plus per-line mandatory-field rules produce the final `SUCCESS` / `SUCCESS (Recovered)` / `WARNING` / `ERROR` verdict — see [dedicated section](#-document-quality-assessment-engine) below.

### 🩹 Automatic Field Recovery + Confidence Scoring
Mandatory fields (Item ID, Description, Quantity, Currency, and others) that are inferred or cross-populated during repair are tracked as **recovered**, not silently treated as clean extractions — and every extracted field carries a quantified confidence signal.

</td></tr>
</table>

---

## 🏗 Platform Architecture

```mermaid
flowchart TB
    subgraph Ingest["📥 Ingest Layer"]
        A1[🖼️ Image]
        A2[📄 Native PDF]
        A3[📄 Scanned PDF]
        A4[📊 CSV / Spreadsheet]
        A5[🧾 JSON]
        A6[🧾 XML]
        A7[🗜️ Archive]
    end

    subgraph Router["🧭 Document Router"]
        B1[classify]
        B2[route]
    end

    subgraph Processors["⚙️ Format Processors"]
        C1[ImageProcessor]
        C2[NativePDFProcessor]
        C3[ScannedPDFProcessor]
        C4[CSVProcessor / ExcelProcessor]
        C5[JSONProcessor]
        C6[XMLProcessor]
        C7[ArchiveProcessor]
    end

    subgraph Core["🧠 Intelligence Core"]
        D1[Offline OCR Service]
        D2[Knowledge Engine]
        D3[Extraction Service<br/>+ Currency Resolution]
        D4[Enterprise Validation Framework]
        D5[Quality Assessment Engine<br/>H1–H6 + Line Rules]
    end

    subgraph Staging["🗄️ ERP-Agnostic Oracle Staging"]
        E1[StagingService]
        E2[(inv_header_stg)]
        E3[(inv_line_stg)]
        E4[(inv_error_stg — full audit trail)]
    end

    F[📧 Notification Framework]
    G[📊 Operations Dashboard]

    Ingest --> Router --> Processors --> Core --> Staging --> F
    Staging --> G
    D2 -. loaded once at startup .-> D3
    D4 -. non-blocking issues .-> D5
```

**Design intent:** every layer is swappable in isolation. A new document format is one processor and one router registration. A new business rule is a JSON edit to the Knowledge Engine. A new ERP target is a downstream consumer of the same staging schema — none of them require touching the others.

---

## 🔄 Processing Pipeline

```mermaid
sequenceDiagram
    participant M as main.py
    participant R as DocumentRouter
    participant P as Format Processor
    participant K as Knowledge Engine
    participant Q as Quality Assessment Engine
    participant S as StagingService
    participant DB as Oracle Staging

    M->>M: build BatchContext (batch id, env, app version)
    M->>DB: init_db()
    M->>K: initialize_application() → load once
    loop for each file in each inputs/<type>
        M->>R: classify(file)
        R->>P: route() → processor.process()
        alt JSON / XML (structured e-invoicing)
            P->>P: parse header + line items directly
        else Image / Scanned PDF
            P->>P: Offline OCR → layout-aware line reconstruction
        else Native PDF
            P->>P: read text layer directly — no OCR
        else CSV / Spreadsheet
            P->>P: tabular parse → header/line mapping
        end
        P->>S: stage_invoice() / stage_structured_invoice()
        S->>S: apply mandatory defaults (never rejects on business data)
        S->>Q: assess_document_quality_safe(header, lines, issues)
        Q-->>S: SUCCESS / SUCCESS (Recovered) / WARNING / ERROR + audit trail
        S->>DB: insert header + lines + full audit trail (one transaction)
        S-->>P: StagingResult(status, quality_status, issue_count)
        P-->>M: ProcessingOutcome(success, error_message)
        M->>M: move file to outputs/processed or outputs/failed
    end
    M->>M: unhandled exception anywhere? → failure notification, non-zero exit
    M->>M: batch completes → success / partial-success summary notification
```

> [!TIP]
> One failed document **never** stops the batch. Every document is staged inside its own transaction; a hard failure on one document rolls back only that document's own transaction. Only a truly fatal, unhandled system exception ends a run early — and even then, a notification fires before the process exits.

---

## 🎯 Document Quality Assessment Engine

This is the platform's decision authority for what "clean data" actually means — and it is deliberately **independent** from the raw severities the Validation Framework produces upstream. A legacy `WARNING` on a validation message never, by itself, determines the final document status; it is classified, and only genuine critical business failures affect the verdict.

### The Status Model

```mermaid
stateDiagram-v2
    [*] --> Assessed
    Assessed --> SUCCESS: All mandatory data present,<br/>nothing repaired, no issues
    Assessed --> SUCCESS_RECOVERED: Mandatory data was missing<br/>but automatically repaired
    Assessed --> WARNING: Only optional data missing<br/>or uncertain — still usable
    Assessed --> ERROR: Mandatory data unrecoverable,<br/>or critical currency conflict

    note right of SUCCESS_RECOVERED
        SUCCESS (Recovered): fully
        usable, but flagged that a
        repair occurred — full
        transparency, not silent success
    end note
```

**Severity priority when multiple findings apply at once:** `ERROR` > `WARNING` > `SUCCESS (Recovered)` > `SUCCESS` — a reviewer always sees the most cautious status that legitimately applies to the document.

### Header Rules (H1–H6)

| Rule | Checks | Outcome |
|---|---|---|
| **H1 — Mandatory Identifier** | At least one of Invoice ID / PO ID / Receipt ID is present | `ERROR` if none present |
| **H2 — Supplier** | Supplier name or supplier ID is present | `ERROR` if neither present |
| **H3 — Mandatory Total** | `total_amount` is not null | `ERROR` if null |
| **H4 — Tax Presence** | Tax appears on the document but wasn't extracted | `WARNING` if a tax label was detected but `total_tax` is null |
| **H5 — Currency Integrity** | Extracted currency vs. the Currency Resolution Engine's own conflict/confidence signal | `ERROR` only on a HIGH-confidence conflict; a LOW/MEDIUM-confidence alternate candidate is **advisory only** — a mere candidate is not treated as genuine ambiguity |
| **H6 — Total Reconciliation** | `Header Total = Line Total + Tax − Discount + Freight + Other Charges`, within tolerance | `WARNING` only on a genuine, formula-complete mismatch |

### Line Rules

- **Mandatory line fields** — Item ID, Description, Quantity, Unit Price, Line Amount must all be populated.
- **Automatic recovery detection** — if all mandatory fields are present *because* one or more were auto-repaired (e.g. Item ID cross-populated from Description), the line is marked `RECOVERED`, not silently `PASS`.
- **Presence check** — zero extracted line items is itself a line-level finding, not something that has to be inferred from an unrelated upstream message.

### Validation Issue Classification

Every issue produced by the upstream Validation Framework is classified before it's allowed anywhere near the final verdict:

| Category | Can it change the final status? |
|---|---|
| **Critical Business Failure** | ✅ Yes — folded in as `ERROR` |
| **Technical Validation** | ❌ No — advisory, audit-trail only |
| **Business Validation** | ❌ No — advisory, audit-trail only |
| **Informational Message** | ❌ No — audit-trail only |
| **Recoverable Condition** | ❌ No — advisory, audit-trail only |

> [!IMPORTANT]
> This classification step exists specifically because inherited severities and *true* quality are not the same thing. A header-total check that doesn't account for freight and other charges, or a currency check that flags any alternate candidate regardless of confidence, will over-warn on perfectly good invoices. The Quality Assessment Engine re-derives the correct verdict from first principles (H1–H6 + Line Rules) instead of trusting inherited labels — and every superseded legacy message is still preserved in the audit trail (`qa.VALIDATION_SERVICE_ISSUE.*`) for full transparency.

### Fail-Safe by Design

The engine's entry point, `assess_document_quality_safe()`, **never raises.** A defect in quality assessment logic is a downstream, additive judgment layer on top of already-captured data — it must never be able to veto data capture. If assessment fails for any reason, the result falls back to a conservative `WARNING` (never `SUCCESS`, never `ERROR`), and the failure itself is recorded as a finding in the audit trail rather than silently swallowed or allowed to roll back an entire staging transaction.

---

## 📄 Supported Document Formats

| Format | Extension(s) | Processor | OCR Required? | Extraction |
|---|---|---|---|---|
| 🖼️ Image | `.jpg` `.jpeg` `.png` `.tif` `.tiff` `.bmp` | `ImageProcessor` | ✅ Yes | ✅ Full header + line-item extraction |
| 📄 Scanned PDF | `.pdf` (no text layer) | `ScannedPDFProcessor` | ✅ Yes | ✅ Full header + line-item extraction |
| 📄 Native PDF | `.pdf` (text layer present) | `NativePDFProcessor` | ❌ Reads text layer directly | ✅ Full header + line-item extraction |
| 🧾 JSON | `.json` | `JSONProcessor` | ❌ No | ✅ Full header + line-item extraction (structured e-invoicing schema) |
| 🧾 XML | `.xml` | `XMLProcessor` | ❌ No | ✅ Full header + line-item extraction (structured invoice schema) |
| 📊 CSV | `.csv` | `CSVProcessor` | ❌ No | ✅ Full header + line-item extraction |
| 📈 Spreadsheet | `.xls` `.xlsx` | `ExcelProcessor` | ❌ No | ✅ Full header + line-item extraction |
| 🗜️ Archive | `.zip` `.rar`* | `ArchiveProcessor` | Depends on contents | ✅ Extracts and re-routes contents through the same pipeline |

\* `.rar` support is listed on the roadmap as an optional future addition.

> [!TIP]
> Adding a new document type follows an open/closed pattern by design: one new document-type enum value, one new processor class, one registration line in the router — nothing else in the pipeline changes.

---

## 🧠 The Knowledge Engine

The core architectural decision in this platform: **business knowledge is not code.** Field aliases, regex patterns, unit-of-measure vocabulary, currency symbols, vendor overrides, and numeric tolerances live in versioned JSON, loaded once at startup by a singleton `KnowledgeManager`.

```
main() → initialize_application() → knowledge_manager.load() → process_batch()
```

At load time, the Knowledge Engine:

- Loads every JSON file exactly once
- Validates structure and cross-references (e.g. every vendor's currency must exist in the reference data)
- Compiles every regex once, never mid-batch
- Converts vocabulary lists into frozensets/dicts for O(1) lookups
- Freezes everything into immutable structures — no document's processing can mutate shared knowledge for the next one

<details>
<summary>📚 <strong>The eight knowledge libraries</strong> (click to expand)</summary>

| File | Responsibility |
|---|---|
| `field_dictionary.json` | Canonical invoice field names + their aliases across languages/regions |
| `regex_library.json` | Compiled patterns for invoice #, PO #, GST/VAT/TIN/PAN, IBAN, SWIFT, and similar identifiers |
| `reference_data.json` | Currencies, units of measure, countries, Incoterms, tax types |
| `business_rules.json` | Tolerances and thresholds used by the validation and reconciliation layers — single source of truth |
| `confidence_rules.json` | Scoring weights driving the field-level confidence engine |
| `vendor_profiles.json` | Per-vendor header/regex/layout overrides |
| `ocr_knowledge.json` | Noise-word lists and known OCR misread substitutions |
| `system_metadata.json` | Knowledge-base version and file manifest |

</details>

**Why this matters:** adding a new field alias, currency, or a vendor's column-header quirks is a JSON edit — not a code change, review, or redeploy.

---

## 🛡 Enterprise Validation & Business Rule Framework

The platform separates two concerns that most IDP tools conflate: **capturing data** and **judging data quality.**

- **Enterprise Validation Framework** — runs immediately at capture time. It never blocks insertion; it only produces non-blocking `INFO`/`WARNING` observations (missing line items, an individual line missing unit price, a header total that doesn't reconcile against the raw line-item sum) that are recorded for review.
- **Business Rule Engine** — tolerance-driven reconciliation logic (numeric tolerances, description-length limits, currency symbol resolution) sourced entirely from the Knowledge Engine's `business_rules.json`, so a tolerance change is a config edit, not a code change.
- **Document Quality Assessment Engine** — the authoritative layer described above, which independently re-derives the final verdict rather than trusting inherited severities.

This two-stage design means a document can accumulate advisory notes at capture time without those notes automatically becoming the final judgment on the document — the Quality Assessment Engine has the final word, and its reasoning is fully auditable.

---

## 📥 Oracle Staging Framework

The staging schema is deliberately **ERP-agnostic** — named after generic invoice concepts, not any one ERP's tables — so the same pipeline can feed different downstream targets without a rewrite.

```mermaid
erDiagram
    INV_HEADER_STG ||--o{ INV_LINE_STG : "has lines"
    INV_HEADER_STG ||--o{ INV_ERROR_STG : "has audit messages"

    INV_HEADER_STG {
        string staging_id PK
        string batch_id
        string business_unit
        string source_file_name
        string source_file_type "PDF | IMAGE | JSON | XML | CSV | XLSX"
        string invoice_id
        string po_id
        string receipt_id
        datetime invoice_date
        string supplier_name
        string customer_name
        string currency
        float total_amount
        float total_gst
        string status "PENDING|LOADED|WARNING|VALIDATED|READY|POSTED|ERROR"
        string quality_status "SUCCESS|SUCCESS (Recovered)|WARNING|ERROR"
        string erp_target "set later by downstream posting engine"
    }
    INV_LINE_STG {
        string line_id PK
        string staging_id FK
        int line_no
        string item_id
        string description
        float quantity
        float unit_price
        float line_amount
    }
    INV_ERROR_STG {
        string error_id PK
        string staging_id FK
        string batch_id
        string error_type "VALIDATION|OCR|SYSTEM"
        string error_field
        string error_message
        string resolved_flag
    }
```

**Lifecycle status semantics:**

| Status | Set by | Meaning |
|---|---|---|
| `PENDING` | Transient | Row created, not yet finalized |
| `LOADED` | This platform | Captured & staged, no issues recorded |
| `WARNING` | This platform | Captured & staged, but review-worthy findings exist — **not** a rejection |
| `VALIDATED` | Downstream process | Passed business validation |
| `READY` | Downstream process | Ready for ERP injection |
| `POSTED` / `PROCESSED` | Downstream process | Posted to the target ERP |
| `ERROR` | This platform | System/technical failure only — never a business-data gap |

**Mandatory-default policy** — the *only* fields ever auto-defaulted (everything else is left `NULL` if extraction can't find it, and every default is logged as a non-blocking `INFO` note and tracked by the Quality Assessment Engine as a recovered field):

| Field | Default when missing |
|---|---|
| Invoice ID | `DUMMYINV` + zero-padded sequence |
| PO ID | `DUMMYPOID` |
| Receipt ID | `DUMMYRCPTID` |
| Invoice Date | System load date (UTC "now") |
| Currency | `USD` |

---

## 📦 Downstream ERP Consumption

> [!IMPORTANT]
> This platform intentionally does **not** ship built-in ERP posting connectors for SAP, PeopleSoft, Oracle Fusion, Dynamics, or NetSuite. This is a deliberate architectural boundary, not a gap — it's what makes the staging layer genuinely ERP-agnostic.

A separate downstream posting process is expected to read rows where `STATUS = 'READY'` and map them onto the AP voucher structure of whichever target ERP is in use:

- Oracle ERP Cloud / Oracle Fusion
- Oracle E-Business Suite (EBS)
- PeopleSoft
- SAP
- Microsoft Dynamics
- NetSuite
- Any other system that can consume a generic staging row

InvoiceLoad's job ends at producing a clean, quality-scored, ERP-agnostic staging row — plus a complete audit trail of every decision made getting there — not at posting it into a specific ERP. This is what allows the same platform to sit in front of a Fortune 500 multi-ERP landscape without becoming coupled to any one of them.

---

## 📧 Notification Framework

A self-contained, branded HTML email framework reports batch outcomes with zero external mail-service dependency — SMTP only, no third-party notification SaaS.

| Scenario | Trigger | Signal |
|---|---|---|
| 🟧 Invalid folder placement | A file's extension doesn't match the `inputs/<type>` folder it was found in | Orange |
| 🟥 Batch failed | An unhandled exception terminates the run | Red |
| 🟩 Batch succeeded | Clean completion, zero warnings | Green |
| 🟧 Partial success | Completion with warnings or non-fatal failures | Orange |

Notifications are optional (`SMTP_ENABLED=false` by default) — the pipeline runs identically and skips sending when disabled. A failed SMTP send is caught and logged, never raised: email is a side effect, never a pipeline dependency.

---

## 📊 Operations Dashboard

A web-based operations dashboard gives AP/shared-services teams visibility into the batch pipeline without needing database access:

- 📊 **Dashboard overview** — live processing pipeline status
- ☁️ **Upload Documents** — manual document intake
- 📋 **Processing Queue** — in-flight and completed batches
- 🔍 **Extraction Results** — per-document header/line-item review
- ✅ **Validation** — quality-assessment findings and audit trail
- 📈 **Analytics** — processing volume trends
- 🔔 **Notifications** — batch outcome history
- ⚙️ **Settings** and **❓ Help & Docs**

Built with a lightweight, dependency-light frontend stack (Alpine.js + static HTML/CSS) so it can be deployed alongside the pipeline without a heavy Node.js build toolchain — consistent with the platform's offline-first philosophy.

---

## 🏛 Enterprise Design Principles

| Principle | How it's enforced |
|---|---|
| **Capture first, validate later** | Business-data gaps are defaulted and logged, never a rejection reason |
| **One bad document never stops a batch** | Every document is staged inside its own transaction |
| **A defect in judgment must never veto data capture** | `assess_document_quality_safe()` never raises; falls back to a conservative `WARNING` and logs its own failure |
| **Business knowledge is configuration, not code** | The entire Knowledge Engine is JSON, loaded once, immutable per batch |
| **Every decision is auditable** | Every default, repair, and quality finding is written to `inv_error_stg` with full reasoning, not just a status code |
| **Severity is classified, not inherited** | The Quality Assessment Engine re-derives status from first principles instead of trusting upstream labels |
| **ERP-agnostic by construction** | The staging schema names generic invoice concepts, never a specific ERP's table structure |
| **Open/closed extensibility** | New formats, rules, and vendors are additive — existing code paths are never modified to add one |

---

## 🔒 Security & Offline Deployment

This platform is purpose-built to run without internet access — a requirement in defense, government, healthcare, and financial-services environments where cloud OCR and SaaS document-AI tools are simply not permitted.

- **OCR runs fully local.** Models are read from local disk paths — no first-run "phone home" to a model-hosting service.
- **No mandatory cloud database.** Local SQLite works out of the box; production points at Oracle (or any SQLAlchemy-supported database) on your own network.
- **No third-party OCR API keys, no per-document billing, no rate limits.**
- **Email notifications are optional** — the pipeline doesn't need to reach a mail server to function.
- **The Knowledge Engine loads from local JSON** — no external configuration or feature-flag service.

**To deploy air-gapped:** copy the project plus your cached OCR model directories onto the target machine, point the OCR model-directory environment variables at their local paths, and run. No outbound connection is required at any point in the processing pipeline.

---

## 🧩 Extensibility

InvoiceLoad follows an open/closed extension pattern throughout:

| To add... | You touch... |
|---|---|
| A new document format | One processor class + one router registration |
| A new field alias / vendor quirk | One JSON edit in `app/knowledge/` |
| A new currency or unit of measure | One JSON edit in `reference_data.json` |
| A new business tolerance | One JSON edit in `business_rules.json` |
| A new quality rule | One new rule function, registered into the H-rule or Line Rule set |
| A new downstream ERP target | Nothing in this platform — consume `STATUS='READY'` rows |

No central "god object" needs to change shape to support any of the above — each extension point is additive by design.

---

## 🏢 Enterprise Use Cases

- **Shared-services AP automation** — a single intake pipeline for a multi-entity, multi-currency, multi-supplier-format AP function.
- **EDI / e-invoicing bridging** — structured JSON/XML e-invoicing feeds (e.g. GST e-invoice IRP-style schemas) staged with the same rigor as a scanned paper invoice.
- **M&A and system-consolidation programs** — a neutral staging layer that lets a newly-combined organization normalize invoice intake before deciding on (or migrating) a target ERP.
- **Regulated / air-gapped environments** — defense, government, and financial-services organizations that cannot route documents through a cloud OCR API.
- **Multi-ERP landscapes** — enterprises running more than one ERP (common post-acquisition) that need one invoice-capture layer feeding several downstream targets.
- **Legacy paper-to-digital transformation** — organizations digitizing years of scanned/photographed invoice backlogs without per-page cloud OCR cost.

---

## 📈 Performance & Benchmarking

> [!IMPORTANT]
> No fabricated timing numbers are published here. Processing throughput is highly dependent on deployment hardware, document complexity, and OCR load — a benchmark number without that context is misleading in an enterprise evaluation.

Processing time for any given document is a function of:

- Whether OCR is required at all (structured JSON/XML feeds skip OCR entirely; scanned/photographed documents cost meaningfully more)
- CPU and available memory on the host running OCR
- Invoice complexity (line-item count, multi-column layouts, wrapped descriptions)
- Page count

**Planned:** a reproducible, published benchmark suite with real throughput numbers per document type, run on defined reference hardware, so any performance claim in this document can be independently verified rather than taken on faith.

---

## ⚙️ Engineering Highlights

A selection of real engineering decisions made in building this platform — the kind of detail that distinguishes a production system from a prototype:

- **Severity ≠ verdict.** The Quality Assessment Engine doesn't trust inherited `WARNING`/`ERROR` labels from the upstream Validation Framework. Every legacy validation message is explicitly classified (Technical / Business / Informational / Recoverable / Critical Business Failure) before it's allowed anywhere near the final status — closing a real class of bug where a document that was, by every meaningful measure, correctly extracted was held at `WARNING` because of an overly literal inherited check.
- **A reconciliation formula is only as good as its inputs.** The header-total reconciliation rule (H6) explicitly accounts for `Line Total + Tax − Discount + Freight + Other Charges` — not just a naive header-total-vs-line-sum comparison — because the naive version false-positives on any invoice with tax, a discount, or freight charges, which is most real invoices.
- **A weak signal is not evidence of ambiguity.** The currency rule (H5) only escalates to a blocking finding on a HIGH-confidence conflict. A LOW/MEDIUM-confidence alternate candidate — which the resolution engine itself has already judged unlikely — is recorded for transparency but never downgrades an otherwise correct invoice.
- **A quality-engine defect must never destroy captured data.** `assess_document_quality_safe()` is deliberately fail-closed-to-warning, not fail-open or fail-hard: an unhandled exception inside quality assessment falls back to a conservative `WARNING` rather than either (a) silently reporting `SUCCESS`, or (b) propagating an exception that would roll back an entire header+line insert transaction over a downstream, additive judgment bug.
- **Recovery is tracked, not hidden.** When a mandatory field (Item ID, Description, Currency, Invoice ID) is only present because it was automatically repaired or defaulted, the document is marked `SUCCESS (Recovered)` rather than plain `SUCCESS` — so a reviewer can always tell a genuinely clean extraction apart from a successfully-repaired one.
- **Datatype defensiveness at integration boundaries.** Currency confidence is normalized to handle both the numeric score format the resolution engine actually emits in production and a string-label format, without ever silently coercing one type into the other — closing a `'float' object has no attribute 'upper'`-class regression at the exact seam between two independently-evolving services.

---

## 📁 Project Structure

```text
InvoiceLoad/
├── main.py                          # batch entry point — no file-type logic
├── app/
│   ├── core/
│   │   ├── config.py                 # Settings — every tunable value, env-driven
│   │   ├── database.py               # SQLAlchemy engine/session, init_db()
│   │   ├── logger.py                 # console + rotating-file logging
│   │   ├── batch_context.py          # BatchContext — carries batch state to templates
│   │   ├── startup.py                # initialize_application() — loads Knowledge Engine once
│   │   └── knowledge_manager.py      # Singleton Knowledge Engine
│   ├── knowledge/                    # 🧠 business knowledge, as JSON — never hardcoded
│   │   ├── field_dictionary.json
│   │   ├── regex_library.json
│   │   ├── reference_data.json
│   │   ├── business_rules.json
│   │   ├── confidence_rules.json
│   │   ├── vendor_profiles.json
│   │   ├── ocr_knowledge.json
│   │   └── system_metadata.json
│   ├── models/
│   │   └── staging_models.py         # InvoiceStagingHeader/Line/Error ORM
│   ├── repositories/
│   │   └── staging_repository.py
│   ├── services/
│   │   ├── document_router.py        # classify() + route() — single dispatch point
│   │   ├── ocr_service.py            # offline OCR wrapper (images + scanned PDFs)
│   │   ├── extraction_service.py     # header/line parsing + currency resolution
│   │   ├── validation_service.py     # non-blocking data-capture checks
│   │   ├── quality_assessment_service.py  # H1–H6 + Line Rules decision engine
│   │   └── staging_service.py        # orchestrates parse → validate → assess → stage
│   ├── processors/                   # one per document type
│   │   ├── image_processor.py
│   │   ├── native_pdf_processor.py
│   │   ├── scanned_pdf_processor.py
│   │   ├── csv_processor.py
│   │   ├── excel_processor.py
│   │   ├── json_processor.py
│   │   ├── xml_processor.py
│   │   └── archive_processor.py      # extracts ZIP/RAR, re-routes contents
│   ├── notifications/                # self-contained HTML email framework
│   │   ├── notification_service.py
│   │   ├── template_loader.py
│   │   ├── email_builder.py
│   │   ├── smtp_client.py
│   │   ├── templates/*.html
│   │   └── assets/ (logo.png, banner.png)
│   └── utils/
│       ├── file_utils.py
│       └── validators.py
├── frontend/
│   ├── login.html                    # sign-in
│   └── dashboard.html                # operations dashboard
├── inputs/{pdf,image,spreadsheet,json,xml,archive}/
├── outputs/{processed,failed}/
├── sql/create_staging_tables_oracle_final.sql
├── logs/
├── requirements.txt
└── .env.example
```

---

## ⚙ Configuration Reference

Every setting has a working default — `.env` only needs to declare what you're overriding.

<details>
<summary><strong>🗄️ Database</strong></summary>

```env
DATABASE_URL=sqlite:///./invoice_staging.db      # dev default
# DATABASE_URL=oracle+oracledb://user:pass@host:1521/?service_name=XEPDB1
```
</details>

<details>
<summary><strong>🔍 OCR (fully offline)</strong></summary>

```env
OCR_PROVIDER=paddle
OCR_LANGUAGE=en
OCR_USE_GPU=false
OCR_MIN_CONFIDENCE=60.0

# Pin these to local, pre-cached model directories to guarantee zero
# outbound network calls of any kind:
OCR_DET_MODEL_DIR=/path/to/models/PP-OCRv6_medium_det
OCR_REC_MODEL_DIR=/path/to/models/PP-OCRv6_medium_rec
```
</details>

<details>
<summary><strong>📂 File discovery & thresholds</strong></summary>

```env
INPUT_PDF_DIR=inputs/pdf
INPUT_IMAGE_DIR=inputs/image
MAX_FILE_SIZE_MB=25.0
ALLOWED_IMAGE_EXTENSIONS=.jpg,.jpeg,.png,.tif,.tiff,.bmp,.webp,.gif
```
</details>

<details>
<summary><strong>📧 Email notifications</strong></summary>

```env
SMTP_ENABLED=false
SMTP_SERVER=smtp.office365.com
SMTP_PORT=587
SMTP_USERNAME=
SMTP_PASSWORD=
SMTP_FROM="Invoice Processor <you@company.com>"
SMTP_TO=
SMTP_USE_TLS=true
```
</details>

> [!CAUTION]
> `.env` is where real secrets belong. Never commit a filled-in `.env` to source control — rotate any credential that ever leaves your machine by accident.

---

## 🚀 Installation & Quick Start

```bash
# 1. Set up your environment
python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate
pip install -r requirements.txt

# 2. Configure (optional — sane defaults work with zero setup)
cp .env.example .env

# 3. Run
python main.py
```

Local development uses **SQLite** out of the box. For production, point `DATABASE_URL` at Oracle (or any SQLAlchemy-supported database) and run `sql/create_staging_tables_oracle_final.sql` through your normal change-control process.

```bash
python main.py                     # process every configured folder
python main.py --skip-pdf          # skip inputs/pdf
python main.py --skip-image        # skip inputs/image
python main.py --skip-spreadsheet  # skip inputs/spreadsheet
python main.py --skip-json         # skip inputs/json
python main.py --skip-xml          # skip inputs/xml
python main.py --skip-archive      # skip inputs/archive
```

---

## 🛠 Technology Stack

| Layer | Technology |
|---|---|
| Language | Python 3.11 |
| ORM / DB toolkit | SQLAlchemy 2.x |
| Production database | Oracle (via `oracledb`), SQLite for local development |
| OCR | Offline, locally-cached OCR models — no cloud dependency |
| PDF text extraction | Native text-layer extraction for born-digital PDFs |
| Email delivery | Standard SMTP — no third-party mail SaaS |
| Frontend | Alpine.js, static HTML/CSS — lightweight, no heavy build toolchain |

---

## 🌟 Why This Platform Is Different

| Dimension | Typical Cloud IDP / OCR SaaS | InvoiceLoad |
|---|---|---|
| Deployment model | Cloud API, per-document billing | Fully offline, air-gapped capable |
| Format coverage | Usually PDF/image-first | 9 formats behind one router, including structured e-invoicing |
| Rejection behavior | Often rejects on missing required fields | Never rejects for a business-data gap — always captures |
| Quality verdict | Often a single opaque confidence score | Explainable, rule-by-rule decision matrix (H1–H6 + Line Rules) with full audit trail |
| Business logic changes | Frequently require a vendor support ticket or release | JSON edit to the Knowledge Engine |
| ERP coupling | Often bundled with one ERP's connector | Deliberately ERP-agnostic staging schema |
| Data residency | Data typically leaves the network | No mandatory outbound network call anywhere in the pipeline |

---

## 🗺 Product Roadmap & Future Vision

- [ ] Extend vendor-profile pattern overrides beyond invoice number (e.g. PO number, tax ID)
- [ ] Optional `rarfile` support for legacy `.rar` archives
- [ ] Publish a reproducible, hardware-referenced performance benchmark suite
- [ ] Expand the Document Quality Assessment Engine's decision matrix with additional configurable business rules
- [ ] Formalize a downstream posting-engine reference implementation for at least one target ERP
- [x] JSON structured invoice extraction
- [x] XML structured invoice extraction
- [x] CSV and Spreadsheet full header/line-item extraction
- [x] Document Quality Assessment Engine (H1–H6 + Line Rules)
- [x] Automatic field recovery tracking (`SUCCESS (Recovered)`)
- [x] Currency Resolution Engine with conflict/confidence scoring

**Future vision:** InvoiceLoad's long-term direction is to remain the neutral, explainable, offline-capable capture layer that sits *in front of* any ERP — never the thing an enterprise has to re-architect around. As quality-assessment rules mature, the intent is for every verdict the platform produces to be defensible enough to support straight-through processing for high-confidence invoices, while routing genuinely ambiguous ones to a human reviewer with a fully reasoned explanation of why.

---

## 📖 Lessons Learned

- **Inherited severity is a trap.** The most subtle bug in this platform's history wasn't a crash — it was a perfectly good invoice sitting at `WARNING` because an upstream check's label was trusted instead of re-derived. Any system that layers judgment on top of another system's output needs to explicitly decide what it owns and what it merely observes.
- **"Non-blocking" has to be enforced structurally, not just documented.** A `try/except` at the entry point of the Quality Assessment Engine — falling back to a safe default instead of raising — did more to protect data integrity than any amount of "this should never happen" commentary.
- **Reconciliation rules are only correct if they model the whole business transaction.** A total-mismatch check that ignores freight and discount isn't a simplification — it's a different, wrong rule that happens to agree with the right one some of the time.
- **Configuration-over-code compounds.** Every business rule pulled out of Python and into JSON is one less deployment required for the business to change its mind.

---

## ❓ FAQ

**Does this need an internet connection to run?**
No. Once OCR models are cached locally, the pipeline runs fully offline.

**What happens if one invoice fails to process?**
It's logged, and the batch continues. Business-data gaps are defaulted and staged with a quality-assessed status; only genuine system failures are recorded as `ERROR`.

**Can I add a new invoice field alias or vendor without touching code?**
Yes, by design — the Knowledge Engine reads from JSON, not from hardcoded Python.

**Does this post directly into SAP / PeopleSoft / Oracle Fusion?**
No — see [Downstream ERP Consumption](#-downstream-erp-consumption). InvoiceLoad produces a validated, quality-scored, ERP-agnostic staging row; a separate downstream process is responsible for ERP-specific posting.

**How is "quality" actually determined — is it just whatever the OCR engine reports?**
No. Quality is a deliberate, independent decision matrix (H1–H6 header rules + Line Rules) that re-derives the verdict from the actual captured data, rather than trusting inherited severities from upstream checks. See [Document Quality Assessment Engine](#-document-quality-assessment-engine).

---

## 👤 About the Author

Built and maintained as a demonstration of enterprise-grade AI engineering: composable intelligence engines, explainable decision-making, offline-first architecture, and integration-safe design patterns (fail-closed-to-warning, classify-don't-inherit, configuration-over-code).

*Add your name, title, and links (LinkedIn / portfolio / GitHub) here before publishing.*

---

## 📄 License

Existing project badge indicates **Internal Use**. Confirm actual license terms with the project owner before treating this as open-source-licensed software.

---

<div align="center">

**Built for enterprises that need invoice data staged correctly, offline, explainably, and without vendor lock-in to a single ERP.**

</div>
