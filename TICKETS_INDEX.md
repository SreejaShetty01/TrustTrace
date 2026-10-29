# TrustTrace MVP — Ticket Index & Work Distribution

> Current assignments: [TEAM_TICKETS.md](TEAM_TICKETS.md). Use its A/B ticket queues and C00 interface agreement to work independently. The original tickets below are feature descriptions; their cross-feature dependencies do not block frontend development against mocks. Its contract-aligned values supersede conflicting examples below.

**Status:** Planning only. Do not begin implementation until the team approves this index.  
**Purpose:** Align the existing 10-ticket plan with the final TrustTrace MVP architecture, API contract, security boundaries, and deployment choices.

---

### Final MVP Architecture Decisions

| Area | Decision |
|---|---|
| **Frontend** | Vite + React + TypeScript + Tailwind CSS |
| **Backend** | Python 3.11+ + FastAPI |
| **Detection** | Deterministic rules/regex + Gemini API; model configured with `GEMINI_MODEL` |
| **Secrets/Config** | `.env` loaded with `pydantic-settings`; commit `.env.example`, never real secrets |
| **Database** | PostgreSQL on Neon, accessed through SQLAlchemy or SQLModel |
| **Processing** | Start with a straightforward async request/response flow. Add FastAPI `BackgroundTasks` only if needed; it is not a durable queue. |
| **Input Limit** | `MAX_CONTENT_BYTES=200000` by default, configurable |
| **Record Access** | Use unpredictable analysis UUIDs plus server-side access checks. `X-User-Id` alone is not authentication. Protect demo access with a configured API key or equivalent validation. |
| **Retention** | Default `RETENTION_DAYS=7`; support deletion and avoid logging raw submitted content |
| **Deployment** | Frontend: Vercel (Person A); backend: Render (Person B); database: Neon PostgreSQL |
| **PayPal** | Explicit `sandbox`, `mock`, and `unavailable` modes. Mock results must be labelled simulated. |
| **Product Boundary** | Analyze submitted content and compare financial claims with user-provided trusted details or authorized sandbox data. Do not claim to intercept or block arbitrary real PayPal payments. |

---

### Ticket Overview & Dependencies

| # | Ticket | Blocked By | Can Run Concurrently With | Owner / Assigned To |
|---|---|---|---|---|
| **01** | [01-dev-scaffold-shared-schemas-mock-fixtures.md](./.scratch/trusttrace-mvp/issues/01-dev-scaffold-shared-schemas-mock-fixtures.md) | None *(Start immediately)* | Start point for all tracks | `[ ]` |
| **02** | [02-content-submission-job-creation.md](./.scratch/trusttrace-mvp/issues/02-content-submission-job-creation.md) | `01` | `04`, `05`, `07` | `[ ]` |
| **03** | [03-background-job-lifecycle-status-polling.md](./.scratch/trusttrace-mvp/issues/03-background-job-lifecycle-status-polling.md) | `01`, `02` | `04`, `05`, `07` | `[ ]` |
| **04** | [04-suspicious-content-prompt-injection-detection.md](./.scratch/trusttrace-mvp/issues/04-suspicious-content-prompt-injection-detection.md) | `01` | `02`, `03`, `05`, `07` | `[ ]` |
| **05** | [05-financial-claim-extraction-pipeline.md](./.scratch/trusttrace-mvp/issues/05-financial-claim-extraction-pipeline.md) | `01` | `02`, `03`, `04`, `07` | `[ ]` |
| **06** | [06-trusted-reference-input-comparison-engine.md](./.scratch/trusttrace-mvp/issues/06-trusted-reference-input-comparison-engine.md) | `05` | `02`, `03`, `04`, `07` | `[ ]` |
| **07** | [07-paypal-sandbox-adapter-status-indicator.md](./.scratch/trusttrace-mvp/issues/07-paypal-sandbox-adapter-status-indicator.md) | `01` | `02`, `03`, `04`, `05` | `[ ]` |
| **08** | [08-evidence-backed-risk-report-findings-viewer.md](./.scratch/trusttrace-mvp/issues/08-evidence-backed-risk-report-findings-viewer.md) | `03`, `04`, `06`, `07` | — | `[ ]` |
| **09** | [09-human-review-consequential-action-controls.md](./.scratch/trusttrace-mvp/issues/09-human-review-consequential-action-controls.md) | `08` | — | `[ ]` |
| **10** | [10-end-to-end-hardening-deployment.md](./.scratch/trusttrace-mvp/issues/10-end-to-end-hardening-deployment.md) | `09` | — | `[ ]` |

---

### Dependency Graph

```text
                    01: Scaffold + shared schemas
                    /          |           \
                   /           |            \
          02: Submit/API  04: Threats  05: Claim extraction
                 |             |               |
                 v             |               v
          03: Lifecycle <------+        06: Comparison
                 |                             |
                 |             07: PayPal -----+
                 |                adapter      |
                 +--------------+--------------+
                                v
                     08: Report + findings UI
                                |
                                v
                     09: Review + deletion
                                |
                                v
                     10: Hardening + deployment
```

---

### Standardized API Routes

| Method | Route | Purpose |
|---|---|---|
| `POST` | `/api/v1/analyses` | Submit content and create analysis |
| `GET` | `/api/v1/analyses/{id}` | Retrieve analysis status / completed report |
| `GET` | `/api/v1/analyses/{id}/evidence` | Retrieve evidence separately (if maintained) |
| `POST` | `/api/v1/analyses/{id}/review` | Record a human review decision |
| `DELETE` | `/api/v1/analyses/{id}` | Delete analysis and associated records |
| `GET` | `/api/v1/integrations/paypal/status` | Report PayPal integration mode/status |

---

### Environment Configuration Template

```dotenv
GEMINI_API_KEY=
GEMINI_MODEL=
DATABASE_URL=
MAX_CONTENT_BYTES=200000
RETENTION_DAYS=7
API_KEY=
PAYPAL_CLIENT_ID=
PAYPAL_CLIENT_SECRET=
PAYPAL_BASE_URL=https://api-m.sandbox.paypal.com
```

---

### Ticket Summaries

#### Ticket 01: Dev Scaffold, Shared Schemas & Mock Fixtures
* **File:** [`01-dev-scaffold-shared-schemas-mock-fixtures.md`](./.scratch/trusttrace-mvp/issues/01-dev-scaffold-shared-schemas-mock-fixtures.md)
* **Summary:** Sets up Vite/React/TypeScript/Tailwind and FastAPI/Python with Neon PostgreSQL configuration, `pydantic-settings`, `.env.example`, and fixtures for clean content, prompt injection, urgency/deception, and altered payment details. Fixtures label mock PayPal data as simulated.
* **Delivers:** Both apps run locally; schemas match the agreed contract; no real credentials committed.

#### Ticket 02: Content Submission & Analysis Creation
* **File:** [`02-content-submission-job-creation.md`](./.scratch/trusttrace-mvp/issues/02-content-submission-job-creation.md)
* **Summary:** Build text-submission UI and `POST /api/v1/analyses`. Enforces `MAX_CONTENT_BYTES` (200,000 bytes default) on client & server, persisting records with stable UUIDs. Never logs raw submitted text.
* **Delivers:** Validated submission workflow and pending UI state without page reload.

#### Ticket 03: Analysis Execution, Persistence & Status Retrieval
* **File:** [`03-background-job-lifecycle-status-polling.md`](./.scratch/trusttrace-mvp/issues/03-background-job-lifecycle-status-polling.md)
* **Summary:** Persist analysis records and status in Neon PostgreSQL using SQLAlchemy or SQLModel. Prefers a straightforward async workflow; enforces server-side access checks on `GET /api/v1/analyses/{id}`.
* **Delivers:** Reports survive page refreshes with predictable status transitions. Documents that in-process background tasks are not durable.

#### Ticket 04: Suspicious-Content & Prompt-Injection Detection
* **File:** [`04-suspicious-content-prompt-injection-detection.md`](./.scratch/trusttrace-mvp/issues/04-suspicious-content-prompt-injection-detection.md)
* **Summary:** Threat analysis engine combining Python rules/regex with Gemini API configured via `GEMINI_MODEL`. Input is placed inside isolated prompt fences.
* **Delivers:** Structured findings with category, severity, evidence text offsets, and concise reasoning, communicating uncertainty without claiming absolute fraud proof.

#### Ticket 05: Financial-Claim Extraction Pipeline
* **File:** [`05-financial-claim-extraction-pipeline.md`](./.scratch/trusttrace-mvp/issues/05-financial-claim-extraction-pipeline.md)
* **Summary:** Extracts amount, currency, payee, invoice number, due date, and payment destinations using regex and model assistance, tying fields to source evidence text.
* **Delivers:** Structured financial claim objects; handles absent fields as `null`/unknown without hallucinating and never silently assumes default currencies.

#### Ticket 06: Trusted-Reference Comparison Engine
* **File:** [`06-trusted-reference-input-comparison-engine.md`](./.scratch/trusttrace-mvp/issues/06-trusted-reference-input-comparison-engine.md)
* **Summary:** UI form for entering expected reference details and a deterministic comparison engine producing `MATCH`, `MISMATCH`, `MISSING`, and `UNABLE_TO_VERIFY` verdicts.
* **Delivers:** Side-by-side expected vs extracted comparisons with evidence. Mismatches prompt human review; never triggers automatic payment actions.

#### Ticket 07: PayPal Adapter & Integration Status
* **File:** [`07-paypal-sandbox-adapter-status-indicator.md`](./.scratch/trusttrace-mvp/issues/07-paypal-sandbox-adapter-status-indicator.md)
* **Summary:** Tri-modal adapter supporting `sandbox`, `mock`, and `unavailable` modes exposed at `GET /api/v1/integrations/paypal/status`.
* **Delivers:** Transparent status reporting; mock results clearly labelled `simulated`; never claims payment interception or blocking.

#### Ticket 08: Evidence-Backed Risk Report & Findings Viewer
* **File:** [`08-evidence-backed-risk-report-findings-viewer.md`](./.scratch/trusttrace-mvp/issues/08-evidence-backed-risk-report-findings-viewer.md)
* **Summary:** Comprehensive report view rendering overall risk score, plain-language summary, recommended safe next actions, and interactive evidence text highlighting.
* **Delivers:** Clear distinction between observed evidence, inferred risk, user references, and sandbox/mock status without uncalibrated score precision.

#### Ticket 09: Human Review, Audit Trail & Deletion
* **File:** [`09-human-review-consequential-action-controls.md`](./.scratch/trusttrace-mvp/issues/09-human-review-consequential-action-controls.md)
* **Summary:** Human-in-the-loop review workflow (`POST /api/v1/analyses/{id}/review`), audit trail, retention enforcement (`RETENTION_DAYS=7`), and on-demand deletion (`DELETE /api/v1/analyses/{id}`).
* **Delivers:** Auditable decision history ensuring payment choices remain strictly human-controlled.

#### Ticket 10: End-to-End Hardening & Deployment
* **File:** [`10-end-to-end-hardening-deployment.md`](./.scratch/trusttrace-mvp/issues/10-end-to-end-hardening-deployment.md)
* **Summary:** End-to-end scenario verification (clean content, prompt injection, tampered invoices, PayPal modes), CORS/rate-limiting enforcement, and deployment to Vercel, Render, and Neon.
* **Delivers:** Hardened production deployment with secret-free walkthrough guides and documented cold start behaviors.
