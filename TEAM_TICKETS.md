# TrustTrace — Tickets for Two Independent Work Tracks

Use this document for assignments and build order. Original tickets 01–10 remain feature descriptions; the A/B tickets below divide their implementation work between two people.

## Ownership

| Teammate | Owns | Can work without |
|---|---|---|
| **A — You** | React frontend, API client, mock API, demo inputs, UI checks, frontend deployment | A running backend, database, Gemini key, or PayPal credentials |
| **B — Your teammate** | FastAPI backend, database, analysis pipeline, comparisons, risk, evidence, review/deletion, backend deployment | A finished UI; use HTTP requests and automated tests |

Names can be swapped. B has the heavier analysis workload; A owns mock scenarios, browser verification, integration testing, and demo documentation to reduce B's coordination work. Do not duplicate detection or comparison logic in the frontend.

**Start:** Complete C00 together, then A starts A01 and B starts B01. After that, each person follows their own ticket order. Mock responses replace cross-person dependencies until connection time.

## C00 — Agree the interface once

**Owner:** A facilitates; B reviews | **Priority:** P0 | **Timebox:** One short setup session

- [ ] Use [API_CONTRACT-1.md.txt](API_CONTRACT-1.md.txt) as the baseline wire contract.
- [ ] Keep `POST /api/v1/analyses` asynchronous: `202` with `analysis_id`, `status`, `created_at`, and `status_url`. Poll GET for the report. This selects the original contract flow over the alternative synchronous flow mentioned in older tickets.
- [ ] Define schemas for queued/processing/failed responses, evidence normalization, and error details before implementing them.
- [ ] Keep the five original endpoints, including separate evidence retrieval. Add deletion and persisted review-history fields as explicitly documented extensions.
- [ ] Record mock/sandbox/unavailable mode and simulation fields as additive contract extensions; simulated data must never set real `paypal_context.used` to true.
- [ ] Choose access handling for the demo. Browser code must not bundle a shared server API key. Use server-validated user/session credentials or a server-side proxy that holds the configured key. Do not use `X-User-Id` alone as authentication.
- [ ] Agree local ports, allowed frontend origin, API base URL, and credentials/header behavior.
- [ ] Record the resolved schema decisions in `contracts/README.md`; do not silently change the API while working separately.

**Deliverable:** A creates `contracts/README.md` and JSON examples under `contracts/fixtures/`; B checks that they are valid backend inputs/outputs. These files are planning deliverables to create, not existing implementation files.

**Done when:** Both can point to the same request, queued response, completed report, evidence response, review response, deletion behavior, and error examples. Frontend work no longer waits for backend implementation.

## Fixed values at the connection boundary

| Item | Agreed value |
|---|---|
| Risk | `low`, `medium`, `high`, `critical`, `unknown` |
| Severity | `info`, `low`, `medium`, `high`, `critical` |
| Job status | `queued`, `processing`, `completed`, `failed` |
| Comparison status | `match`, `mismatch`, `missing_reference`, `missing_extracted_value`, `unable_to_verify` |
| Review decision | `proceed`, `pause`, `reject`, `needs_more_info` |
| Content | Plain text first; unknown values null/omitted |
| Input limit | 200,000 UTF-8 bytes by default; backend configurable |
| Review effect | Record only; `payment_action_taken: false` |
| PayPal used | True only for relevant PayPal data actually retrieved and used |

Older uppercase comparison labels and alternate review values are superseded by this table. UI labels can be friendly, but JSON must use the exact values above.

## File ownership to avoid merge conflicts

These are proposed paths for implementation; the repository currently contains planning documents.

| Path | Owner |
|---|---|
| `frontend/` including its tests/config | A |
| `contracts/fixtures/`, `docs/demo.md`, `docs/integration-checklist.md` | A; B reviews examples |
| `backend/` including migrations/tests/config | B |
| `contracts/README.md` and root API contract | A edits agreed interface changes; B reviews |
| Root README/environment/deployment files | Coordinate; each person edits only their section |

Use separate feature branches and avoid editing each other's folders. Frontend TypeScript types and backend Pydantic models implement the same JSON examples; neither person needs the other's language toolchain to work.

## Teammate A — Frontend ticket queue

### A01 — Frontend scaffold and API adapter

**Original:** 01 | **Depends on:** C00 | **Priority:** P0

- [ ] Initialize Vite + React + TypeScript + Tailwind in `frontend/`.
- [ ] Create a single API adapter for create, retrieve, evidence, review, status, and delete operations.
- [ ] Keep components independent of fetch details; implement real and mock transports with identical method signatures.
- [ ] Configure the non-secret API URL and mock/live mode; do not put Gemini, PayPal, database, or shared server secrets in frontend environment variables.

**Done:** The frontend starts independently and can switch transports through configuration without rewriting components.

### A02 — Mock API and shared scenarios

**Original:** 01, 10 | **Depends on:** A01 | **Priority:** P0

- [ ] Implement local mocked responses using the agreed fixtures, including queued → processing → completed and failed states.
- [ ] Support evidence, review save, status, deletion, and structured validation/access errors.
- [ ] Include clean content, prompt injection, explicit recipient mismatch, amount mismatch, vague recipient, missing reference, and unavailable/simulated PayPal scenarios.
- [ ] Make mock mode visibly labeled. Do not present fixture output as a real analysis.
- [ ] Use deterministic scenarios so B can reuse their expected outcomes in backend tests.

**Done:** Every frontend workflow is testable with the backend stopped and no external credentials.

### A03 — Submission and trusted-reference forms

**Original:** 02, frontend portion of 06 | **Depends on:** A01–A02 | **Priority:** P0

- [ ] Build text submission and optional source metadata.
- [ ] Build optional reference type/ID, payee, amount, currency, due date, and destination inputs; allow partial records.
- [ ] Validate empty text and UTF-8 byte size; display structured server errors, including 422 reference errors.
- [ ] Submit the exact agreed request JSON; prevent accidental duplicate submits.

**Done:** Valid and invalid requests work against mocks; form data survives recoverable request errors.

### A04 — Status, polling, and report recovery

**Original:** Frontend portion of 03 | **Depends on:** A03 | **Priority:** P0

- [ ] Retain the returned analysis ID and use a refreshable analysis route/view.
- [ ] Poll GET while queued/processing; stop on completion, failure, navigation, or an agreed timeout.
- [ ] Show progress, network errors, unknown IDs, and failed-analysis feedback.
- [ ] Offer explicit retry/status refresh after a bounded polling timeout; do not silently resubmit an analysis.

**Done:** All job states and a browser refresh behave correctly using mock data.

### A05 — Report, comparisons, and evidence viewer

**Original:** Frontend portions of 06, 08 | **Depends on:** A04 | **Priority:** P0

- [ ] Display risk/unknown, summary, human-review flag, findings, financial claims, comparisons, recommendations, and limitations.
- [ ] Fetch evidence separately and link it to findings/discrepancies.
- [ ] Highlight only valid spans in the corresponding normalized text; handle null offsets without guessing.
- [ ] Handle empty findings/claims, ambiguous values, missing references, and partial checks.
- [ ] Render untrusted content as text; support keyboard use and mobile widths.

**Done:** All report fixture variants are understandable and safely rendered. The UI does not calculate authoritative risk or comparison results.

### A06 — PayPal status presentation

**Original:** Frontend portion of 07 | **Depends on:** A02, A05 | **Priority:** P0

- [ ] Display configured/authorized/capability status and missing requirements.
- [ ] Distinguish general integration readiness from whether this analysis used PayPal context.
- [ ] Label simulated results clearly; show actual unavailable/lookup-failure reasons.
- [ ] Send lookup requests only for supported operations; never ask for backend credentials in the browser.

**Done:** Mock, unavailable, and sandbox fixture states display truthfully without any real PayPal connection.

### A07 — Review and deletion controls

**Original:** Frontend portion of 09 | **Depends on:** A05 | **Priority:** P0 for review; planned extension for deletion

- [ ] Submit `proceed`, `pause`, `reject`, or `needs_more_info` with an optional note.
- [ ] Display saved review confirmation and persisted review entries using the C00 history extension.
- [ ] Prevent unintended double saves; handle not-ready and access errors.
- [ ] Add explicit deletion confirmation, call DELETE, and clear the report on success.
- [ ] Explain that review decisions do not execute, approve, or block PayPal payments.

**Done:** Review/save/reload and delete flows work against mocks using agreed response shapes.

### A08 — Frontend verification and handoff

**Original:** Frontend portion of 10 | **Depends on:** A03–A07 | **Priority:** P0

- [ ] Verify required scenarios against mocks and record results in `docs/integration-checklist.md`.
- [ ] Check polling cancellation, error recovery, evidence safety, keyboard access, and narrow screens.
- [ ] Write synthetic demo scenarios and frontend startup instructions in `docs/demo.md`.
- [ ] Prepare Vercel settings; connect to the live API only during I01.

**Done:** A can demonstrate the entire UI with mock mode, hand B exact requests, and switch to live mode through configuration.

## Teammate B — Backend ticket queue

### B01 — Backend scaffold, schemas, and persistence

**Original:** Backend portion of 01 | **Depends on:** C00 | **Priority:** P0

- [ ] Initialize Python 3.11+ / FastAPI, Pydantic settings, and backend environment examples.
- [ ] Define request/response models matching `contracts/fixtures/` and fixed enums above.
- [ ] Configure Neon PostgreSQL through SQLAlchemy or SQLModel and add migrations for analyses, findings/claims/comparisons, evidence, and reviews.
- [ ] Configure server-side Gemini model/key, database, PayPal, access, size, and retention settings; never commit real secrets.

**Done:** Backend starts without the frontend, schemas accept agreed examples, and a database migration works.

### B02 — Create/retrieve API and job lifecycle

**Original:** Backend portions of 02, 03 | **Depends on:** B01 | **Priority:** P0

- [ ] Implement POST create with 202 and GET status/report retrieval.
- [ ] Validate input, content type/size, optional references, and options with contract-shaped errors.
- [ ] Persist queued/processing/completed/failed states and UTC timestamps; enforce record access on every operation.
- [ ] Add a processor interface; use a labeled test processor locally until B03–B06 are connected.
- [ ] Bound external work with timeouts; handle interrupted jobs and document in-process BackgroundTasks restart limitations.

**Done:** HTTP requests alone can create and retrieve a persisted job; failures do not leave records processing forever. Test output is never represented as real model analysis.

### B03 — Detection engine

**Original:** 04 | **Depends on:** B01 | **Priority:** P0

- [ ] Implement rules and Gemini-assisted detection for supported suspicious-content categories.
- [ ] Treat source text as untrusted data; do not expose tools/secrets or permit submitted instructions to control execution. Prompt delimiters alone are not the security boundary.
- [ ] Validate model output and return structured findings with evidence; omit invented confidence scores.
- [ ] Handle missing credentials, quota, invalid model, and provider failures with explicit incomplete-check/failure information.
- [ ] Test clean, suspicious, and injection scenarios independently of HTTP/UI.

**Done:** The detector produces validated, evidence-backed output or an explicit limitation; no guaranteed fraud verdict.

### B04 — Financial extraction

**Original:** 05 | **Depends on:** B01 | **Priority:** P0

- [ ] Extract supported parties, amount/currency, invoice/order reference, dates, destination/link, requested action, and urgency.
- [ ] Link claims to source evidence; preserve raw values where useful.
- [ ] Keep absent/ambiguous values unknown and never silently assume currency or recipient identity.
- [ ] Validate structured output and test extraction using synthetic input fixtures.

**Done:** The extractor can be tested with text alone and does not depend on frontend input forms or PayPal.

### B05 — Deterministic reference comparison

**Original:** Backend portion of 06 | **Depends on:** B01; use fixture claims until B04 is ready | **Priority:** P0

- [ ] Compare compatible extracted/reference values with defined text/date/currency/money normalization.
- [ ] Return the five lowercase comparison statuses, source, explanation, and evidence linkage.
- [ ] Handle partial data and vague destinations without inventing mismatches.
- [ ] Keep the module independent of Gemini and PayPal; accept structured values from either authorized source.

**Done:** Fixture claims/reference records produce deterministic results without the extraction pipeline or UI running.

### B06 — Evidence, risk, and report assembly

**Original:** Backend portion of 08; evidence endpoint | **Depends on:** B02–B05 | **Priority:** P0

- [ ] Combine findings, claims, comparisons, integration usage, and limitations into the completed report.
- [ ] Implement evidence retrieval with stable IDs and traceability for discrepancies as well as findings.
- [ ] Verify exact source spans against normalized text; use null offsets plus explanation when unavailable.
- [ ] Compute risk from evidence/severity/discrepancies and uncertainty, not model confidence alone; include human-review flag and recommendation.
- [ ] Return unknown/limited results explicitly when checks cannot complete.

**Done:** HTTP requests retrieve a full report and its evidence using the agreed fixtures' structure.

### B07 — PayPal status and adapter

**Original:** Backend portion of 07 | **Depends on:** B01; connect into B06 later | **Priority:** P0 status/fallback; conditional real lookup

- [ ] Implement status with truthful configuration, authorization, environment, and capability flags.
- [ ] Support explicit unavailable and simulated mock modes, without treating mock data as real PayPal context.
- [ ] Inspect actual sandbox resource/permission access before implementing supported lookups; resolve invoice display-reference versus provider-resource ID mapping.
- [ ] Keep OAuth credentials/tokens server-side and out of evidence/logs.
- [ ] Continue analysis on lookup failure; identify the actual source of each comparison.
- [ ] Mark context used only when relevant data was retrieved and used.

**Done:** Unavailable and simulated modes pass tests without credentials. A real lookup is added only when access is verified and is separately tested.

### B08 — Reviews, deletion, and retention

**Original:** Backend portion of 09 | **Depends on:** B02; use stored fixture reports until B06 is ready | **Priority:** P0 review; planned deletion/retention extension

- [ ] Implement review validation/storage with original contract decisions and `payment_action_taken: false`.
- [ ] Return saved reviews on retrieval using the agreed history extension; derive actor identity from validated access context.
- [ ] Define repeated-review/idempotency behavior and reject disallowed reviews before completion.
- [ ] Implement authorized DELETE and cascade removal of related analysis data.
- [ ] Enforce `RETENTION_DAYS=7` by a documented runnable cleanup mechanism; an environment variable alone does not expire records.

**Done:** HTTP-only tests cover save/reload, duplicate retry, denial, deletion, and expiry behavior.

### B09 — Backend verification and deployment

**Original:** Backend portion of 10 | **Depends on:** B02–B08 | **Priority:** P0

- [ ] Verify agreed fixtures, failed providers, missing references, evidence spans, reviews, deletion, retention, and access isolation.
- [ ] Normalize errors/request IDs; enforce CORS, rate limits, content limits, and safe logging.
- [ ] Document backend setup and HTTP examples; return CORS/access configuration details to A.
- [ ] Deploy to Render with Neon, apply migrations, and verify API operations before connecting the UI.

**Done:** A receives a reachable API URL and safe setup instructions; B can demonstrate required scenarios using HTTP requests alone.

## Original-ticket mapping

This table covers all ten original tickets without making the frontend wait on backend features.

| Original | A assignment | B assignment |
|---|---|---|
| 01 Scaffold/schemas/fixtures | C00, A01, A02 | C00, B01 |
| 02 Submission/create | A03 | B02 |
| 03 Lifecycle/status | A04 | B02 |
| 04 Detection | A02 scenarios; A05 presentation | B03 |
| 05 Extraction | A02 scenarios; A05 presentation | B04 |
| 06 Reference/comparison | A03 form; A05 table | B05 |
| 07 PayPal | A06 | B07 |
| 08 Report/evidence | A05 | B06 |
| 09 Review/deletion | A07 | B08 |
| 10 Hardening/deploy | A08, I01–I02 | B09, I01–I02 |

## Recommended independent build order

| Stage | A — You | B — Teammate |
|---|---|---|
| Interface agreement | C00 | C00 |
| Start independently | A01 → A02 | B01 → B02 |
| Build core workflow | A03 → A04 → A05 | B03 → B04 → B05 → B06 |
| Supporting features | A06 → A07 | B07 → B08 |
| Prepare handoff | A08 | B09 |
| Connect and verify | I01 → I02 | I01 → I02 |

Within B's track, B05 can start with fixture claims and B08 with stored fixture reports. Neither waits on another person. A's mock API removes the backend dependency during all frontend work. Final acceptance still requires the connected application.

## I01 — Connect the completed pieces

**Owner:** A drives browser integration; B fixes API mismatches | **Depends on:** A08 and B09

- [ ] B supplies the API URL, allowed origin, access/session setup, and a safe synthetic test record.
- [ ] A switches the API adapter from mock to live using configuration; no component rewrite.
- [ ] Check requests/responses against C00, including enums, errors, nulls, normalized text, and review history extensions.
- [ ] Exercise create → poll → report → evidence → review → reload → delete.
- [ ] Test PayPal unavailable first, simulated separately, and a real sandbox lookup only if implemented.
- [ ] Confirm deployment CORS/access handling and that no server secret is in the browser bundle.

**Done:** The browser completes the real workflow against persisted backend data with mock mode disabled.

## I02 — Final shared acceptance

**Owner:** A records results; B fixes backend issues; each person fixes their owned files

- [ ] Clean content does not invent findings; explain verification limits.
- [ ] Injection/urgency/payment-change cases produce inspectable evidence.
- [ ] Explicit recipient and amount mismatches show the actual reference source.
- [ ] Vague recipient and missing references remain uncertain/unverified.
- [ ] PayPal unavailable/provider failure does not falsely report a completed verification.
- [ ] Invalid input, access denial, worker/model failures, and polling timeouts show useful feedback.
- [ ] Evidence text renders safely and offsets work for Unicode examples.
- [ ] Review survives reload and never takes a payment action.
- [ ] Deletion, retention, and access restrictions behave as documented.
- [ ] README and synthetic demo steps work from a fresh checkout/deployment.

**Done:** Both confirm the deployed demo matches the PRD and API contract, including explicitly documented extensions and limitations.

## Rules while working separately

1. An A ticket depends on mock contract behavior until I01; B tickets are independently verified using HTTP/module tests.
2. Contract changes require a short joint agreement. A updates fixtures/docs and B updates schemas in the same agreed change before either side relies on it.
3. Share one small progress update: completed ticket, next ticket, and any interface question.
4. Mark implementation tickets complete after their local acceptance criteria pass; mark original feature tickets complete only after I01/I02 validate both halves.
5. Keep real PayPal lookup conditional. Never replace missing real verification with an unlabeled mock result.
