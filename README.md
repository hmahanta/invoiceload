<div align="center">

# 📄 InvoiceLoad

### Enterprise Intelligent Document Processing (IDP) Platform for Invoice Staging

**Offline-first invoice ingestion, understanding, extraction, validation, and Oracle staging - no cloud OCR, no API keys, no outbound network dependency.**

[![Python](https://img.shields.io/badge/Python-3.11-3776AB?logo=python&logoColor=white)](#)
[![SQLAlchemy](https://img.shields.io/badge/SQLAlchemy-2.x-D71F00?logo=sqlalchemy&logoColor=white)](#)
[![Oracle](https://img.shields.io/badge/Oracle-XE%20%7C%20SQLAlchemy-F80000?logo=oracle&logoColor=white)](#)
[![OCR](https://img.shields.io/badge/OCR-PaddleOCR%20%7C%20Offline-2E7D32?logo=googlelens&logoColor=white)](#)
[![Architecture](https://img.shields.io/badge/Architecture-ERP--Agnostic%20Staging-informational)](#)
[![License](https://img.shields.io/badge/License-Internal%20Use-lightgrey)](#)
[![Status](https://img.shields.io/badge/Status-Active%20Development-yellow)](#)

*Turn any invoice - scanned, photographed, or born-digital - into clean, staged, ERP-ready data.*

</div>

---

> [!NOTE]
> This README documents what the codebase actually does today. Where a
> capability is still in progress, it's labeled **🚧 In Progress** or
> **⏳ Planned** rather than presented as shipped. See
> [Honesty & Status Legend](#-honesty--status-legend) below.

---

## 📌 Table of Contents

1. [Overview](#-overview)
2. [Honesty & Status Legend](#-honesty--status-legend)
3. [Key Highlights](#-key-highlights)
4. [Why This Project Exists](#-why-this-project-exists)
5. [The OCR Engine Journey](#-the-ocr-engine-journey)
6. [Architecture at a Glance](#-architecture-at-a-glance)
7. [Processing Pipeline](#-processing-pipeline)
8. [The Knowledge Engine](#-the-knowledge-engine)
9. [Supported Input Formats & Processor Status](#-supported-input-formats--processor-status)
10. [Oracle Staging Framework](#-oracle-staging-framework)
11. [Downstream ERP Consumption](#-downstream-erp-consumption)
12. [Notification Framework](#-notification-framework)
13. [Frontend (Prototype UI)](#-frontend-prototype-ui)
14. [Project Structure](#-project-structure)
15. [Configuration Reference](#-configuration-reference)
16. [Installation & Quick Start](#-installation--quick-start)
17. [Technologies Used](#-technologies-used)
18. [Error Handling Philosophy](#-error-handling-philosophy)
19. [Offline & Air-Gapped Deployment](#-offline--air-gapped-deployment)
20. [Performance](#-performance)
21. [Screenshots](#-screenshots)
22. [Roadmap](#-roadmap)
23. [FAQ](#-faq)
24. [License](#-license)
25. [Acknowledgements](#-acknowledgements)

---

## 🚀 Overview

**InvoiceLoad** is a batch pipeline that ingests, understands, extracts,
validates, and stages invoice data from multiple document formats into
an ERP-agnostic Oracle (or any SQLAlchemy-supported) staging schema.

It exists to solve one recurring enterprise problem: vendor invoices
never arrive in one shape. A single AP/AR function typically receives
scanned paper, phone-camera photos, native digital PDFs, spreadsheets,
and structured JSON/XML feeds from EDI or e-invoicing partners - and
every one of them ultimately needs to land as a clean, validated row
that a downstream ERP posting process can consume.

InvoiceLoad is that landing zone: one router, a shared set of
processors per format, a JSON-driven Knowledge Engine instead of
hardcoded extraction rules, and a single staging schema that doesn't
assume which ERP is on the other end.

---

## 🧭 Honesty & Status Legend

This README uses three status markers throughout, applied per feature
based on what has actually been verified in the codebase (not aspiration):

| Marker | Meaning |
|---|---|
| ✅ **Implemented** | Confirmed working end-to-end in the reviewed source |
| 🚧 **Partial** | The plumbing exists (file is validated, content is captured, routed, and staged), but full field-level extraction is not yet wired in |
| ⏳ **Planned** | Described in code comments / roadmap as a future step, not yet implemented |

> [!IMPORTANT]
> Performance numbers, a full `requirements.txt`, and the exact contents
> of the `app/services/*.py` and `app/knowledge/*.json` files were not
> all available for direct review while writing this document. Sections
> describing them are based on the project's own architecture
> documentation; anything not independently verified against source is
> marked accordingly rather than stated as fact.

---

## ✨ Key Highlights

| Capability | Status |
|---|---|
| Scanned invoice processing (image → OCR → extraction) | 🚧 Partial *(documented pipeline; OCR/extraction internals not directly reviewed)* |
| Native PDF text-layer extraction | 🚧 Partial *(documented pipeline; not directly reviewed)* |
| JSON invoice processing (GSTN e-invoice IRP schema) | ✅ Implemented |
| XML invoice processing (Sage Company/Invoice schema) | ✅ Implemented |
| CSV invoice processing | 🚧 Partial - validates & captures, field mapping not yet implemented |
| Excel invoice processing | 🚧 Partial - validates & captures, field mapping not yet implemented |
| JSON-driven Knowledge Engine (aliases, regex, UOM, currency, vendor overrides) | ✅ Implemented |
| Business-rule-based validation (non-blocking) | ✅ Implemented |
| Confidence scoring engine | ⏳ Planned - rules file exists (`confidence_rules.json`), not yet wired into extraction |
| ERP-agnostic Oracle staging (header/line/error schema) | ✅ Implemented |
| Built-in ERP connector code (SAP, PeopleSoft, etc.) | ⏳ Not in scope - see [Downstream ERP Consumption](#-downstream-erp-consumption) |
| Email notification framework | 🚧 Partial *(documented; not directly reviewed this session)* |
| Vendor-specific extraction profiles | 🚧 Partial *(documented; not directly reviewed this session)* |
| Web dashboard | 🚧 Prototype UI exists (`login.html`, `dashboard.html`); backend API wiring not confirmed |

---

## 🎯 Why This Project Exists

Every enterprise that processes vendor invoices eventually hits the
same wall: invoices arrive in a dozen different shapes, and every one
of them needs to land in the same place - a clean, validated row in an
ERP staging table.

InvoiceLoad's design philosophy, taken directly from its own service
layer's docstrings, is **"capture first, validate later"**: one bad
invoice should never stop a batch of ten thousand good ones. Every
document is processed and staged independently; failures are recorded
as auditable rows, not silently dropped and not fatal to the run.

Concretely, this shows up as a hard rule enforced in `staging_service.py`:
a business-data gap (a missing invoice date, an unparseable PO number)
is **never** a reason to reject a document - it's defaulted, logged as
an `INFO`/`WARNING` message in the error table, and the invoice still
stages successfully. `ERROR`-severity status is reserved exclusively for
genuine technical failures - a database error, a malformed file, an
unhandled exception - never for missing business data on an otherwise
successfully captured invoice.

---

## 🧭 The OCR Engine Journey

According to the project's own architecture notes, the OCR layer went
through three iterations before landing on its current design:

| Stage | Engine | What it offered | Why it wasn't the final answer |
|---|---|---|---|
| 1️⃣ | **PaddleOCR** | Strong multilingual accuracy, actively maintained, fully local models | Model-hosting probes could reach a remote host on first run; thread-count defaults caused OpenMP deadlocks under batch load |
| 2️⃣ | **Surya OCR** | Clean API, promising layout awareness | Local-model support was a moving target, pinned to a pre-breaking-change release that capped further evolution |
| 3️⃣ | **Mistral OCR (API)** | Excellent accuracy, minimal local compute | A hosted API call is incompatible with locked-down / air-gapped environments |

**Landing point:** OCR was decoupled from the extraction engine behind a
stable `OCRService` contract, and the shipped configuration runs
PaddleOCR fully local, fully cached, with model directories pinned to
disk paths and thread counts capped via environment variables - no
outbound network call once models are cached, no API key, no
subscription.

---

## 🏗 Architecture at a Glance

```mermaid
flowchart TB
    subgraph Ingest["📥 Ingest - inputs/&lt;type&gt;/"]
        A1[Image]
        A2[Native PDF]
        A3[Scanned PDF]
        A4[CSV / Excel]
        A5[JSON]
        A6[XML]
        A7[ZIP / RAR]
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

    subgraph Core["🧠 Extraction Core"]
        D1[OCRService]
        D2[Knowledge Engine]
        D3[extraction_service]
        D4[validation_service]
    end

    subgraph Staging["🗄️ Oracle Staging"]
        E1[StagingService]
        E2[(inv_header_stg)]
        E3[(inv_line_stg)]
        E4[(inv_error_stg)]
    end

    F[📧 Notification Framework]

    Ingest --> Router --> Processors --> Core --> Staging --> F
    D2 -. loaded once at startup .-> D3
```

---

## 🔄 Processing Pipeline

```mermaid
sequenceDiagram
    participant M as main.py
    participant R as DocumentRouter
    participant P as Processor
    participant S as StagingService
    participant DB as Oracle Staging

    M->>M: build BatchContext (batch id, env, app version)
    M->>DB: init_db()
    M->>M: scan inputs/&lt;type&gt;/ for misplaced files (non-fatal alert)
    loop for each file in each inputs/&lt;type&gt;
        M->>R: classify(file)
        R->>P: route() -> processor.process()
        alt XML / JSON (structured)
            P->>P: parse header + line items directly
        else Image / Scanned PDF
            P->>P: OCR -> layout-aware line reconstruction
        else Native PDF
            P->>P: read text layer directly (no OCR)
        end
        P->>S: stage_invoice() / stage_structured_invoice()
        S->>S: apply mandatory defaults (never rejects on business data)
        S->>DB: insert header + lines + INFO/WARNING messages (one transaction)
        S-->>P: StagingResult(status, issue_count)
        P-->>M: ProcessingOutcome(success, error_message)
        M->>M: move file to outputs/processed or outputs/failed
    end
    M->>M: unhandled exception anywhere? -> failure email, non-zero exit
    M->>M: batch completes -> success / partial-success summary email
```

> [!TIP]
> One failed document **never** stops the batch. Only a truly fatal,
> unhandled exception (database down, out of memory) ends a run early -
> and even then, a notification is sent before the process exits.

---

## 🧠 The Knowledge Engine

The core architectural decision in this project: **business knowledge is
not code.** Field aliases, regex patterns, unit-of-measure vocabulary,
currency symbols, vendor overrides, and numeric tolerances live in
versioned JSON under `app/knowledge/`, loaded once at startup by a
singleton `KnowledgeManager`.

```
main() → initialize_application() → knowledge_manager.load() → process_batch()
```

At load time, per the project's own documentation, the Knowledge Engine:

- Loads every JSON file exactly once
- Validates structure and cross-references (e.g. every vendor's currency must exist in `reference_data.json`)
- Compiles every regex once, never mid-batch
- Converts vocabulary lists into frozensets/dicts for O(1) lookups
- Freezes everything into immutable structures - no document's processing can mutate shared knowledge for the next one

<details>
<summary>📚 <strong>The eight knowledge libraries</strong> (click to expand)</summary>

| File | Responsibility |
|---|---|
| `field_dictionary.json` | Canonical invoice field names + their aliases across languages/regions |
| `regex_library.json` | Compiled patterns for invoice #, PO #, GST/VAT/TIN/PAN, IBAN, SWIFT, and similar identifiers |
| `reference_data.json` | Currencies, units of measure, countries, Incoterms, tax types |
| `business_rules.json` | Tolerances and thresholds used by the validation layer - single source of truth |
| `confidence_rules.json` | Scoring weights intended for a confidence-scoring engine - **⏳ not yet wired into extraction** |
| `vendor_profiles.json` | Per-vendor header/regex/layout overrides |
| `ocr_knowledge.json` | Noise-word lists and known OCR misread substitutions |
| `system_metadata.json` | Knowledge-base version and file manifest |

> [!NOTE]
> A top-level `knowledge/` folder (outside `app/`) also exists in this
> repository but is explicitly marked deprecated - see `DEPRECATED.md`.
> Nothing in the running pipeline reads from it; the canonical library is
> `app/knowledge/`, read exclusively by `KnowledgeManager`.

</details>

**Why this matters:** adding a new field alias, currency, or a vendor's
column-header quirks is a JSON edit - not a code change, review, or
redeploy.

---

## 📄 Supported Input Formats & Processor Status

| Format | Extension(s) | Processor | Uses OCR? | Extraction Status |
|---|---|---|---|---|
| 🖼️ Image | `.jpg` `.jpeg` `.png` `.tif` `.tiff` `.bmp` | `ImageProcessor` | ✅ Yes | 🚧 Documented, not directly reviewed this session |
| 📄 Native PDF | `.pdf` (text layer present) | `NativePDFProcessor` | ❌ No - reads text layer directly | 🚧 Documented, not directly reviewed this session |
| 📄 Scanned PDF | `.pdf` (no text layer) | `ScannedPDFProcessor` | ✅ Yes | 🚧 Documented, not directly reviewed this session |
| 🧾 JSON | `.json` | `JSONProcessor` | ❌ No | ✅ **Full header + line-item extraction** (GSTN e-invoice IRP schema), confirmed in `staging_service.stage_structured_invoice()` |
| 🧾 XML | `.xml` | `XMLProcessor` | ❌ No | ✅ **Full header + line-item extraction** (Sage `Company`/`Invoices`/`Invoice` schema - Customer, Invoice, InvoiceItems, Carriage) |
| 📊 CSV | `.csv` | `CSVProcessor` | ❌ No | 🚧 Validates & captures content; header/line-item mapping not yet implemented |
| 📈 Excel | `.xls` `.xlsx` | `ExcelProcessor` | ❌ No | 🚧 Validates & captures content; header/line-item mapping not yet implemented |
| 🗜️ Archive | `.zip` `.rar`* | `ArchiveProcessor` | Depends on contents | 🚧 Extracts and re-routes contents; `.rar` support listed as planned |

\* `.rar` support is listed in the roadmap as an optional future addition (`rarfile` dependency).

> [!TIP]
> Adding a new document type follows an open/closed pattern by design:
> one new `DocumentType`, one new processor class, one registration line
> in the router - nothing else in the pipeline needs to change.

---

## 📥 Oracle Staging Framework

The staging schema is deliberately **ERP-agnostic** - named after
generic invoice concepts, not any one ERP's tables - so the same
pipeline can feed different downstream targets without a rewrite.

```mermaid
erDiagram
    INV_HEADER_STG ||--o{ INV_LINE_STG : "has lines"
    INV_HEADER_STG ||--o{ INV_ERROR_STG : "has messages"

    INV_HEADER_STG {
        string staging_id PK
        string batch_id
        string business_unit
        string source_file_name
        string source_file_type "PDF | IMAGE | JSON | XML"
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
        string erp_target "set later by downstream meta engine"
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

**Lifecycle status semantics** (as encoded in `StagingStatus`):

| Status | Set by | Meaning |
|---|---|---|
| `PENDING` | Transient | Row created, not yet finalized |
| `LOADED` | This pipeline | Captured & staged, no issues recorded |
| `WARNING` | This pipeline | Captured & staged, but INFO/WARNING messages exist - **not** a rejection |
| `VALIDATED` | Downstream service | Passed business validation |
| `READY` | Downstream service | Ready for ERP injection |
| `POSTED` / `PROCESSED` | Downstream service | Posted to the target ERP |
| `ERROR` | This pipeline | System/technical failure only - never a business-data gap |

**Mandatory-default policy** - the *only* fields ever auto-defaulted
(everything else is left `NULL` if extraction can't find it, and every
default is logged as a non-blocking `INFO` note):

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
> This project does **not** ship built-in ERP connector code for SAP,
> PeopleSoft, Oracle Fusion, Dynamics, or NetSuite. That distinction
> matters for anyone evaluating this repository against enterprise
> integration requirements.

Per the staging schema's own documentation, a separate **downstream
"meta engine"** - explicitly out of this project's scope - is expected
to read rows where `STATUS = 'READY'` and map them onto the AP voucher
structure of whichever target ERP is in use:

- Oracle ERP Cloud / Oracle Fusion
- Oracle E-Business Suite (EBS)
- PeopleSoft
- SAP
- Microsoft Dynamics
- NetSuite
- Any other system that can consume a generic staging row

InvoiceLoad's job ends at producing a clean, validated, ERP-agnostic
staging row (plus a full audit trail of every decision made getting
there) - not at posting it into a specific ERP.

---

## 📧 Notification Framework

Per the project's architecture documentation, a self-contained branded
HTML email framework reports batch outcomes with zero external mail
service dependency (SMTP only):

| Scenario | Trigger | Header Color |
|---|---|---|
| 🟧 Invalid folder placement | A file's extension doesn't match the `inputs/<type>` folder it was found in | Orange |
| 🟥 Batch failed | An unhandled exception terminates the run | Red |
| 🟩 Batch succeeded | Clean completion, zero warnings | Green |
| 🟧 Partial success | Completion with warnings or non-fatal failures | Orange |

Notifications are optional (`SMTP_ENABLED=false` by default) - the
pipeline runs identically and skips sending. A failed SMTP send is
caught and logged, never raised: email is a side effect, never a
pipeline dependency.

*(🚧 The notification service source itself wasn't directly reviewed
this session - this section reflects the project's own documentation.)*

---

## 🖥 Frontend (Prototype UI)

Two HTML mockups exist in the repository: a sign-in page and an
operations dashboard, built with Alpine.js and inline styling, with a
`window.IDP_API_BASE` pointing at an expected `/api/v1` backend.

> [!WARNING]
> These are UI prototypes. Several dashboard values are bound via
> Alpine's `x-text`/`x-data` to mock in-page data, and no backend API
> server matching `/api/v1` was available for review - treat this as a
> design reference, not a confirmed live integration.

**Sign-in page** - username/email + password fields, "Welcome back" greeting.

**Dashboard** - left-navigation with:

- 📊 Dashboard (overview)
- ☁️ Upload Documents
- 📋 Processing Queue
- 🔍 Extraction Results
- ✅ Validation
- 📈 Analytics
- 🔔 Notifications
- ⚙️ Settings
- ❓ Help & Docs

Main panels observed in the markup: **Live Processing Pipeline**,
**Extraction Results**, **Activity Timeline**, and **Processing Volume
· Last 7 Days**, plus a document search bar ("Search invoices, vendors,
PO numbers…").

---

## 📁 Project Structure

```text
InvoiceLoad/
├── main.py                          # batch entry point - no file-type logic
├── app/
│   ├── core/
│   │   ├── config.py                 # Settings - every tunable value, env-driven
│   │   ├── database.py               # SQLAlchemy engine/session, init_db()
│   │   ├── logger.py                 # console + rotating-file logging
│   │   ├── batch_context.py          # BatchContext - carries batch state to templates
│   │   ├── startup.py                # initialize_application() - loads Knowledge Engine once
│   │   └── knowledge_manager.py      # Singleton Knowledge Engine
│   ├── knowledge/                    # 🧠 business knowledge, as JSON - never hardcoded
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
│   │   ├── document_router.py        # classify() + route() - single dispatch point
│   │   ├── ocr_service.py            # offline OCR wrapper (images + scanned PDFs)
│   │   ├── extraction_service.py     # header/line parsing engine
│   │   ├── validation_service.py     # non-blocking data-capture checks
│   │   └── staging_service.py        # orchestrates parse → validate → stage
│   ├── processors/                   # one per DocumentType
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
│   ├── login.html                    # sign-in prototype
│   └── dashboard.html                # operations dashboard prototype
├── knowledge/                        # ⚠️ DEPRECATED - see DEPRECATED.md; superseded by app/knowledge/
├── inputs/{pdf,image,spreadsheet,json,xml,archive}/
├── outputs/{processed,failed}/
├── sql/create_staging_tables_oracle_final.sql
├── logs/
├── requirements.txt
└── .env.example
```

> [!NOTE]
> Everything under `app/` above (except `staging_models.py`,
> `staging_service.py`, and `xml_processor.py`) reflects the project's
> own architecture documentation rather than files directly reviewed in
> this session.

---

## ⚙ Configuration Reference

Every setting has a working default - `.env` only needs to declare what
you're overriding.

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
> `.env` is where real secrets belong. Never commit a filled-in `.env`
> to source control - rotate any credential that ever leaves your
> machine by accident.

---

## 🚀 Installation & Quick Start

```bash
# 1. Set up your environment
python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate
pip install -r requirements.txt

# 2. Configure (optional - sane defaults work with zero setup)
cp .env.example .env

# 3. Run
python main.py
```

Local development uses **SQLite** out of the box. For production, point
`DATABASE_URL` at Oracle (or any SQLAlchemy-supported database) and run
`sql/create_staging_tables_oracle_final.sql` through your normal
change-control process - remember that adding a new `SourceFileType`
enum member (e.g. `XML`) does **not** retroactively alter an existing
Oracle `CHECK` constraint; that requires its own migration.

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

## 🛠 Technologies Used

| Layer | Technology |
|---|---|
| Language | Python 3.11 |
| ORM / DB toolkit | SQLAlchemy 2.x |
| Production database | Oracle (via `oracledb`), SQLite for local dev |
| OCR | PaddleOCR, run fully offline with locally cached models |
| PDF text extraction | pdfplumber / PyMuPDF (per architecture docs) |
| Email delivery | Standard SMTP - no third-party mail SaaS |
| Frontend prototype | Alpine.js, static HTML/CSS |

> [!NOTE]
> A pinned `requirements.txt` was not available for direct review - the
> table above lists technologies named explicitly in the project's own
> documentation, not a dependency-by-dependency audit.

---

## 🛡 Error Handling Philosophy

- **Business-data gaps never fail a document.** Missing invoice date, PO
  number, or receipt ID → defaulted, logged as `INFO`, invoice still
  stages as `LOADED`.
- **`WARNING` status** is used when non-blocking issues exist (e.g. no
  line items found, header/line total mismatch) - the invoice is still
  staged, just flagged for review.
- **`ERROR` status is reserved for genuine system failures** - a
  database error, malformed/unreadable file, or unhandled exception.
  It is never used for a business-data gap on an otherwise
  successfully captured invoice.
- **One bad file never stops a batch.** Each document is processed and
  staged inside its own transaction; a hard failure on one document
  rolls back only that document's transaction.
- **Multi-invoice files degrade gracefully.** For source formats that
  can contain more than one invoice per file (confirmed for XML), one
  invoice failing to stage doesn't prevent the others in the same file
  from staging successfully.

---

## 📴 Offline & Air-Gapped Deployment

Per the project's architecture documentation, this pipeline is
purpose-built to run without internet access:

- OCR models are read from local disk paths - no first-run "phone home" to a model host
- Database can be local SQLite - no cloud database required
- Email notifications are optional (`SMTP_ENABLED=false`) - the pipeline doesn't need to reach a mail server to function
- No third-party OCR API keys, no per-document billing, no rate limits
- The Knowledge Engine loads from local JSON - no external configuration service

**To deploy air-gapped:** copy the project plus your cached OCR model
directories onto the target machine, set the `OCR_*_MODEL_DIR`
variables in `.env` to their local paths, and run.

---

## 📊 Performance

> [!IMPORTANT]
> No formal benchmark run was available for this document, so no
> specific timing numbers are quoted here - a fabricated benchmark
> table would be actively misleading in an enterprise-facing README.

Processing time for any given document is a function of:

- OCR requirement (scanned/photographed documents cost meaningfully more than structured JSON/XML, which skip OCR entirely)
- CPU and available memory
- Invoice complexity (line-item count, multi-column layouts)
- Page count

**⏳ Planned:** publish a reproducible benchmark script and real
throughput numbers per document type once available, rather than
estimated figures.

---

## 📸 Screenshots

> [!NOTE]
> No captured screenshots were provided for this document. The
> [Frontend](#-frontend-prototype-ui) section above describes the
> actual UI structure found in `login.html` / `dashboard.html`.
> Placeholders below are intentional - replace with real captures
> before publishing.

| View | Status |
|---|---|
| Sign-in | 📸 _add screenshot_ |
| Dashboard overview | 📸 _add screenshot_ |
| Upload flow | 📸 _add screenshot_ |
| Processing queue | 📸 _add screenshot_ |
| Extraction results | 📸 _add screenshot_ |
| Notifications | 📸 _add screenshot_ |

---

## 🗺 Roadmap

- [ ] Map CSV and Excel processor output onto full invoice header/line-item schema (currently capture-only)
- [ ] Wire the JSON-driven confidence-scoring engine (`confidence_rules.json`) into `extraction_service.py`
- [ ] Extend vendor-profile pattern overrides beyond `invoice_number` (e.g. PO number, tax ID)
- [ ] Optional `rarfile` support for legacy `.rar` archives
- [ ] Confirm and document a live backend API (`/api/v1`) behind the dashboard prototype
- [ ] Publish a reproducible performance benchmark suite
- [x] JSON structured invoice extraction (GSTN e-invoice IRP schema)
- [x] XML structured invoice extraction (Sage Company/Invoice schema)

---

## ❓ FAQ

**Does this need an internet connection to run?**
No. Once OCR models are cached locally, the pipeline runs fully offline
- per the project's own architecture documentation.

**What happens if one invoice fails to process?**
It's logged, and the batch continues. Business-data gaps are defaulted
and staged with a `WARNING`; only genuine system failures are recorded
as `ERROR`.

**Can I add a new invoice field alias or vendor without touching code?**
Yes, by design - the Knowledge Engine reads from JSON under
`app/knowledge/`, not from hardcoded Python.

**Does this post directly into SAP / PeopleSoft / Oracle Fusion?**
No - see [Downstream ERP Consumption](#-downstream-erp-consumption).
InvoiceLoad produces a validated, ERP-agnostic staging row; a separate
downstream process is responsible for ERP-specific posting.

**Is the web dashboard live and connected to the backend?**
Not confirmed. The HTML in this repository is a prototype UI pointing
at an expected `/api/v1`; the API server itself wasn't available for
review.

---

## 📄 License

Existing project badge indicates **Internal Use**. Confirm actual
license terms with the project owner before treating this as
open-source-licensed software.

---

## 🙏 Acknowledgements

- **PaddleOCR** - offline OCR engine
- **SQLAlchemy** - ORM and database toolkit
- **Alpine.js** - frontend prototype interactivity

---

<div align="center">

**Built for enterprises that need invoice data staged correctly, offline, and without vendor lock-in to a single ERP.**

</div>
