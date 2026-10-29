# TrustTrace MVP — Updated Ticket Index & Work Distribution

> Current two-person assignment board: [TEAM_TICKETS.md](TEAM_TICKETS.md). Follow its A/B queues, C00 interface agreement, and I01/I02 connection checks. This index remains background context; the new board selects asynchronous 202/polling behavior and preserves the original API enums where older tickets disagree.

**Status:** Planning only. Do not begin implementation until the team approves this index.  
**Purpose:** Align the existing 10-ticket plan with the final TrustTrace MVP architecture, API contract, security boundaries, and deployment choices.

## 1. Final MVP decisions

| Area | Decision |
|---|---|
| Frontend | Vite + React + TypeScript + Tailwind CSS |
| Backend | Python 3.11+ + FastAPI |
| Detection | Deterministic rules/regex + Gemini API; model configured with `GEMINI_MODEL` |
| Secrets/config | `.env` loaded with `pydantic-settings`; commit `.env.example`, never real secrets |
| Database | PostgreSQL on Neon, accessed through SQLAlchemy or SQLModel |
| Processing | Start with a straightforward async request/response flow. Add FastAPI `BackgroundTasks` only if needed; it is not a durable queue. |
| Input limit | `MAX_CONTENT_BYTES=200000` by default, configurable |
| Record access | Use unpredictable analysis UUIDs plus server-side access checks. `X-User-Id` alone is not authentication. Protect demo access with a configured API key or equivalent validation. |
| Retention | Default `RETENTION_DAYS=7`; support deletion and avoid logging raw submitted content |
| Deployment | Frontend: Vercel (Person A); backend: Render; database: Neon PostgreSQL |
| PayPal | Explicit `sandbox`, `mock`, and `unavailable` modes. Mock results must be labelled simulated. |
| Product boundary | Analyze submitted content and compare financial claims with user-provided trusted details or authorized sandbox data. Do not claim to intercept or block arbitrary real PayPal payments. |

**Model note:** Do not hardcode Gemini 2.0 Flash or Gemini 1.5 Flash. Check current model availability and quota, then configure the selected supported model through `GEMINI_MODEL`.

## 2. Ticket overview and dependencies

| # | Ticket | Blocked by | Can run concurrently with | Main deliverable |
|---|---|---|---|---|
| **01** | Dev scaffold, shared schemas & mock fixtures | None | Start point for all tracks | Runnable frontend/backend, config template, schemas, fixtures |
| **02** | Content submission & analysis creation | `01` | `04`, `05`, `07` | Validated submission endpoint and UI |
| **03** | Analysis execution, persistence & status retrieval | `01`, `02` | `04`, `05`, `07` | Persisted lifecycle and retrieval; polling only if async flow is retained |
| **04** | Suspicious-content & prompt-injection detection | `01` | `02`, `03`, `05`, `07` | Rule + Gemini findings with evidence |
| **05** | Financial-claim extraction pipeline | `01` | `02`, `03`, `04`, `07` | Structured claims with evidence and uncertainty |
| **06** | Trusted-reference comparison engine | `05` | `02`, `03`, `04`, `07` | Deterministic field-by-field comparison |
| **07** | PayPal adapter & integration status | `01` | `02`, `03`, `04`, `05` | Explicit sandbox/mock/unavailable modes |
| **08** | Evidence-backed risk report & findings viewer | `03`, `04`, `06`, `07` | — | Combined report and evidence-linked findings |
| **09** | Human review, audit trail & deletion | `08` | — | Review decision, audit record, deletion |
| **10** | End-to-end hardening & deployment | `09` | — | Scenario tests, security/config checks, deployment guide |

### Dependency notes
- Ticket `01` defines shared schemas, environment settings, and fixtures before parallel work starts.
- Tickets `04`, `05`, and `07` can proceed in parallel after `01`.
- Ticket `06` depends on `05` because it compares extracted claims against trusted reference details.
- Ticket `03` must not add a queue or complex job infrastructure by default. If `BackgroundTasks` and polling are used, document that in-process tasks can be interrupted by a restart.
- Ticket `08` integrates lifecycle, threat findings, financial comparisons, and PayPal status.
- Ticket `10` must test clean, suspicious, mismatched-financial-detail, and mock/unavailable PayPal scenarios against the deployed app.

## 3. Dependency graph

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

## 4. Ticket summaries and acceptance focus

### Ticket 01 — Dev scaffold, shared schemas & mock fixtures
**File:** `01-dev-scaffold-shared-schemas-mock-fixtures.md`  
Set up Vite/React/TypeScript/Tailwind and FastAPI/Python. Add Pydantic request/response schemas, PostgreSQL configuration, `pydantic-settings`, `.env.example`, and fixtures for clean content, prompt injection, urgency/deception, and altered payment details.

**Acceptance:** Both apps run locally; schemas match the agreed API contract; no real credentials are committed; fixtures label mock PayPal data as simulated.

### Ticket 02 — Content submission & analysis creation
**File:** `02-content-submission-job-creation.md`  
Build the text-submission UI and `POST /api/v1/analyses`. Validate non-empty input and enforce `MAX_CONTENT_BYTES` on client and server. Persist an analysis record and return a stable UUID.

**Acceptance:** Define one success/error contract. If analysis is synchronous, return the documented analysis/report resource. If asynchronous, return `202 Accepted` with a status URL and align Ticket 03. Do not log full submitted content.

### Ticket 03 — Analysis execution, persistence & status retrieval
**File:** `03-background-job-lifecycle-status-polling.md`  
Persist analysis records and status in PostgreSQL using SQLAlchemy or SQLModel. Prefer a straightforward async workflow first; retain `BackgroundTasks` and polling only if required.

**Acceptance:** Reports survive page refreshes; analysis retrieval enforces access checks. If async, support `queued`, `processing`, `completed`, and `failed`, with bounded polling and visible errors. Document that in-process tasks are not durable.

### Ticket 04 — Suspicious-content & prompt-injection detection
**File:** `04-suspicious-content-prompt-injection-detection.md`  
Combine Python rules/regex with Gemini, configured using `GEMINI_API_KEY` and `GEMINI_MODEL`. Treat submitted content as untrusted data; it must not override detector instructions.

**Acceptance:** Return finding category, severity, evidence text/spans where available, and concise reasoning. Handle missing credentials, rate limits, invalid model configuration, and API failures. Communicate uncertainty; do not present model output as proof.

### Ticket 05 — Financial-claim extraction pipeline
**File:** `05-financial-claim-extraction-pipeline.md`  
Extract supported fields such as amount, currency, payee, invoice/reference number, due date, and payment destination using deterministic extraction and model assistance where appropriate.

**Acceptance:** Tie fields to source evidence; represent absent or ambiguous values as `null`/unknown rather than inventing them; preserve original and normalized values where useful; never silently assume a currency.

### Ticket 06 — Trusted-reference input & comparison engine
**File:** `06-trusted-reference-input-comparison-engine.md`  
Provide a form for expected trusted details and compare them deterministically with extracted claims.

**Acceptance:** Support agreed fields such as payee, amount, currency, and invoice/reference ID. Use `MATCH`, `MISMATCH`, `MISSING`, and `UNABLE_TO_VERIFY`. Show expected and extracted values with evidence. A mismatch prompts human review; it does not trigger an automatic payment decision.

### Ticket 07 — PayPal adapter & integration status
**File:** `07-paypal-sandbox-adapter-status-indicator.md`  
Implement explicit `sandbox`, `mock`, and `unavailable` modes. The sandbox adapter uses server-side OAuth credentials and the PayPal sandbox base URL. Mock mode supports offline demos and must be labelled simulated in API responses and UI.

**Acceptance:** Configure `PAYPAL_CLIENT_ID`, `PAYPAL_CLIENT_SECRET`, and `PAYPAL_BASE_URL` via environment settings. Never expose secrets to the browser or commit them. Verify API operations/permissions before promising invoice or order verification. Missing credentials/permissions means `unavailable`; never claim arbitrary payment interception or blocking.

### Ticket 08 — Evidence-backed risk report & findings viewer
**File:** `08-evidence-backed-risk-report-findings-viewer.md`  
Combine threat findings, extracted claims, trusted-reference comparisons, and PayPal integration status into one report.

**Acceptance:** Display summary, risk level, findings, evidence, discrepancies, and safe next steps. Distinguish observed evidence, inferred risk, user-provided references, and sandbox/mock results. Do not imply a score is statistically calibrated unless validated.

### Ticket 09 — Human review, audit trail & deletion
**File:** `09-human-review-consequential-action-controls.md`  
Record a human decision and optional notes; support on-demand deletion. Use the consistent route `POST /api/v1/analyses/{id}/review` and `DELETE /api/v1/analyses/{id}`.

**Acceptance:** Agree on decisions such as `approved`, `flagged_suspicious`, `escalated`, and `rejected`. Store decision and timestamp; do not imply reviewer approval proves content is safe. Enforce access checks. Default `RETENTION_DAYS=7`; document expiry enforcement. Delete analysis and associated findings/reviews according to the data model.

### Ticket 10 — End-to-end hardening & deployment
**File:** `10-end-to-end-hardening-deployment.md`  
Complete API error consistency, input/access controls, environment setup, scenario testing, and deployment to Vercel, Render, and Neon.

**Acceptance:** Test clean content, suspicious instructions, prompt injection, altered financial details, missing references, Gemini failures, and all PayPal modes. Verify CORS, secrets, request limits, access controls, deletion, and retention. Include `.env.example` and a secret-free deployment/demo guide. Document cold starts/free-tier limitations.

## 5. API routes to keep consistent

| Method | Route | Purpose |
|---|---|---|
| `POST` | `/api/v1/analyses` | Submit content and create analysis |
| `GET` | `/api/v1/analyses/{id}` | Retrieve analysis status/report |
| `GET` | `/api/v1/analyses/{id}/evidence` | Retrieve evidence separately, if the team keeps this endpoint |
| `POST` | `/api/v1/analyses/{id}/review` | Record a human review decision |
| `DELETE` | `/api/v1/analyses/{id}` | Delete analysis and associated records |
| `GET` | `/api/v1/integrations/paypal/status` | Report PayPal integration mode/status |

Agree whether evidence is embedded in the main report or served separately; do not build both without a reason. Keep routes and field names synchronized across API contract, Pydantic models, frontend types, and tickets.

## 6. Environment configuration checklist

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

Use the same environment-variable names in code and documentation. Missing optional credentials must result in documented safe behavior.

## 7. Ownership and next step

Assignments are defined in [TEAM_TICKETS.md](TEAM_TICKETS.md): A owns the frontend/mock scenarios, B owns backend/analysis/integrations, and I01/I02 cover the final connection. Complete C00 once, then each person can follow their own queue without waiting for the other's implementation. The older synchronous-flow alternative and conflicting enums are superseded by that board. This task creates planning documents only; implementation has not started.

**Scope reminder:** TrustTrace is an evidence-backed content and financial-claim analysis tool that supports informed human review. It does not independently authorize, intercept, or block real payments.
