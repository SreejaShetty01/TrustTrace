# TrustTrace — Two-Person Work Plan

> For the current ticket assignments, independent build order, mock API boundary, and connection checklist, use [TEAM_TICKETS.md](TEAM_TICKETS.md). This document retains the feature requirements and earlier planning decisions; interface conflicts are resolved in TEAM_TICKETS.md against the original API contract.

Based on PRD-1.md.txt and API_CONTRACT-1.md.txt, version 0.2.

## 1. How to divide the work

Suggested starting split; swap names if your teammate prefers frontend work.

| Person | Main responsibility | Owns |
|---|---|---|
| Person A — You | Frontend and user workflow | Submission form, reference-data form, loading states, report, evidence viewer, review controls, integration-status display, frontend checks |
| Person B — Teammate | Backend, analysis, and integrations | API endpoints, validation, job processing, detection, extraction, comparison, risk assessment, storage, PayPal adapter, backend tests |
| Both | Agreement and integration | Shared JSON examples, scope decisions, end-to-end checks, demo, deployment |

Each feature has one owner who completes it. The other person reviews it and checks that their side works with it. Person B has more processing work, so Person A also owns shared sample scenarios, integration testing, and demo documentation. If the backend becomes a bottleneck, Person A can take the comparison module after the frontend is connected, provided both agree on its input/output first.

**MVP outcome:** A user pastes a suspicious message, optionally enters trusted invoice details, receives an evidence-backed report, and records a review decision. The application accurately shows whether PayPal data was used.

## 2. Agree on these before building

Spend your first working session on this checklist. Record decisions here.

- [x] Confirm frontend framework and backend language/framework: **Frontend: Vite + React (TypeScript, Tailwind CSS) | Backend: Python FastAPI (Python 3.11+)**
- [x] Choose the detector/model and how backend credentials are supplied: **Hybrid detector (Rule-based heuristics/regex + Google Gemini API, e.g. Gemini 2.0 Flash / 1.5 Flash); backend credentials supplied via `GEMINI_API_KEY` in `.env` loaded with `pydantic-settings` (.env.example committed)**
- [x] Choose storage and where the background analysis worker runs: **Managed PostgreSQL (via SQLAlchemy / SQLModel, e.g. Neon or Supabase free tier) + FastAPI in-process `BackgroundTasks` (zero extra container/queue infrastructure required)**
- [x] Set maximum content size; the contract's 200,000-byte error example is illustrative, not a confirmed limit: **200,000 bytes (~200 KB) as confirmed default (matching API contract), configurable via `MAX_CONTENT_BYTES` environment variable**
- [x] Decide authentication and ownership rules for analysis records: **Open / UUID-based capability access for MVP demo, with an optional `X-API-Key` or `X-User-Id` header for tenant/user scoping**
- [x] Decide retention/deletion behavior for submitted text, evidence, and reviews: **Retain records indefinitely by default for demo review, provide an explicit `DELETE /api/v1/analyses/{id}` endpoint for on-demand deletion, and support an optional configurable TTL (e.g. `RETENTION_DAYS=7`)**
- [x] Confirm demo hosting and deployment responsibility: **Vercel for Frontend (Person A) + Render or Railway for FastAPI Backend (Person B), with shared Neon PostgreSQL, coordinated jointly for end-to-end demo**
- [x] Check PayPal sandbox access early; record supported resource, permissions, and lookup identifier, or explicitly choose unavailable status for the first demo: **Dual-mode adapter: explicitly report `unavailable` status by default with clear UI messaging, but connect to PayPal Sandbox Invoicing API (`v2/invoicing/invoices`, OAuth2 client credentials) when `PAYPAL_CLIENT_ID` / `PAYPAL_CLIENT_SECRET` are provided, with mock sandbox fallback for offline testing**

Start with plain text. Additional content types and uploads are separate extensions. Neither a framework choice nor a working PayPal integration is assumed by this plan.

## 3. Feature checklist and ownership

P0 = required for the text-first MVP. P1 = conditional or later extension.

### F01 — Shared API examples and development setup

**Priority:** P0 | **Owner:** Both | **Depends on:** Initial decisions

- [ ] A: Create shared sample inputs: clean message, suspicious payment change, prompt injection, amount mismatch, vague recipient, and no reference data.
- [ ] B: Create example queued, processing, completed, and failed responses, plus structured error examples.
- [ ] Both: Check examples against the API contract; agree on any amendments before coding against them.
- [ ] Both: Document environment setup, start commands, and local API configuration.
- [ ] Both: Keep secrets out of committed files; provide placeholder environment examples.

**Done when:** Both people can run their part locally. A can build against clearly labeled fixture responses while B builds the real API.

### F02 — Content submission

**Priority:** P0 | **Frontend owner:** A | **Backend owner:** B | **PRD:** FR-1

- [ ] A: Build a required text input and Analyze button.
- [ ] A: Add optional source type/label fields and useful empty/oversize/error feedback.
- [ ] A: Submit JSON to POST /api/v1/analyses and keep the returned analysis ID.
- [ ] B: Validate JSON, non-empty content, byte-size limit, supported content type, metadata, and options.
- [ ] B: Return a stable analysis ID, queued status, creation time, and status URL with 202 Accepted.
- [ ] B: Return the documented 400, 413, 415, and reference-validation errors where applicable.

**Done when:** A valid text request creates an analysis; invalid input gives an understandable, structured error.

### F03 — Trusted reference input

**Priority:** P0 | **Frontend owner:** A | **Backend owner:** B | **PRD:** FR-1, FR-4

- [ ] A: Build an optional form for reference type/ID, payee, amount, currency, due date, and payment destination.
- [ ] A: Allow partial reference data and make its user-supplied origin visible.
- [ ] B: Validate supplied types, dates, currency, and monetary values without requiring every optional field.
- [ ] B: Preserve supplied values and their source for comparison.

**Done when:** Analysis works with no reference, a partial reference, or a complete reference. Invalid supplied fields return 422 INVALID_REFERENCE_DATA.

### F04 — Job lifecycle and storage

**Priority:** P0 | **Owner:** B | **Frontend support:** A

- [ ] B: Store analysis records, findings, claims, comparisons, evidence, and review events with stable IDs and UTC timestamps.
- [ ] B: Implement queued → processing → completed or failed transitions.
- [ ] B: Implement GET /api/v1/analyses/{analysis_id} for current status and completed reports.
- [ ] B: Persist a safe failure summary and request ID when processing fails.
- [ ] A: Poll until completed or failed; stop polling on completion, failure, or leaving the view.
- [ ] A: Show progress, recoverable request errors, and failed-analysis feedback without an endless spinner.

**Done when:** A user can submit a request and see the final result or an explicit failure. Worker/integration failures do not leave jobs stuck indefinitely.

### F05 — Suspicious-content and prompt-injection detection

**Priority:** P0 | **Owner:** B | **PRD:** FR-2

- [ ] Detect indicators of impersonation, suspicious urgency, deceptive claims, changed payment instructions, suspicious payment links, and prompt injection.
- [ ] Treat submitted text as data; never allow its instructions to trigger tools, reveal secrets, or change system behavior.
- [ ] Produce finding IDs, category, severity, title, explanation, and evidence links.
- [ ] Include confidence only when a meaningful calibrated estimate is available; do not invent precise scores.
- [ ] Handle detector failures and incomplete checks explicitly.

**Done when:** Clean content can produce no findings; suspicious examples produce specific indicators with supporting evidence. Wording does not claim definitive fraud detection.

### F06 — Financial claim extraction

**Priority:** P0 | **Owner:** B | **PRD:** FR-3

- [ ] Extract payer/payee, amount, currency, invoice/order reference, due date, payment destination/link, requested action, and urgency when present.
- [ ] Capture payment-detail-change statements as findings with evidence.
- [ ] Keep missing or ambiguous values null/omitted; preserve uncertainty.
- [ ] Associate extracted values with their supporting source text internally for traceability.
- [ ] Validate structured detector/model output before using it.

**Done when:** Stated values are extracted accurately. A phrase such as “new recipient” is not invented into a specific recipient identity or account.

### F07 — Field-by-field verification

**Priority:** P0 | **Owner:** B | **PRD:** FR-4

- [ ] Compare extracted claims with available trusted reference fields.
- [ ] Define normalization for currency, monetary precision, dates, and text values.
- [ ] Return match, mismatch, missing_reference, missing_extracted_value, or unable_to_verify as appropriate.
- [ ] Identify the actual comparison source on every result.
- [ ] Compare only compatible values; never treat a vague phrase as a confirmed destination mismatch.
- [ ] Keep user-supplied and PayPal comparison results distinguishable if both exist.

**Done when:** Amount and explicit recipient mismatches are visible, matching values are reported correctly, and missing/ambiguous values are distinguished from mismatches.

### F08 — Evidence and source locations

**Priority:** P0 | **Backend owner:** B | **Frontend owner:** A | **PRD:** FR-5

- [ ] B: Implement GET /api/v1/analyses/{analysis_id}/evidence.
- [ ] B: Return stable evidence IDs, source, excerpt, explanation, and links to supported findings/comparisons.
- [ ] B: Define normalization and offset semantics; document whether end offsets are exclusive and how Unicode characters are counted.
- [ ] B: Ensure exact-span excerpts match the corresponding normalized text; use null offsets and an explanation when precision is unavailable.
- [ ] A: Display excerpts, source labels, comparison basis, and uncertainty.
- [ ] A: Highlight source text when reliable offsets are available; safely render all submitted content and excerpts as text.

**Done when:** A user can inspect the source behind each finding or discrepancy without executing submitted HTML/scripts.

### F09 — Overall risk and recommendation

**Priority:** P0 | **Backend owner:** B | **Frontend owner:** A | **PRD:** FR-6

- [ ] B: Define risk rules that consider evidence, severity, discrepancies, missing checks, and uncertainty.
- [ ] B: Return overall risk, risk summary, needs_human_review, recommended action, and limitations.
- [ ] B: Use unknown when assessment is unavailable; never equate missing verification with low risk.
- [ ] A: Present the explanation and recommendation prominently, with visible limitations.
- [ ] A: Distinguish suspicious content, confirmed discrepancies, and unverified claims.

**Done when:** A user understands why the report has its risk level and what to do next. Model confidence alone never determines risk.

### F10 — Analysis report screen

**Priority:** P0 | **Owner:** A | **Depends on:** Shared examples, F04–F09

- [ ] Display overall risk, summary, and human-review requirement.
- [ ] Display findings with severity and explanations.
- [ ] Display extracted financial fields and comparison results.
- [ ] Provide access to supporting evidence and source text.
- [ ] Display recommended action, limitations, and analysis-specific PayPal usage.
- [ ] Handle empty findings, no financial claims, missing references, partial verification, and failed analysis.
- [ ] Make forms and report controls keyboard accessible and usable on mobile.

**Done when:** The entire report is understandable from one connected workflow, including incomplete and clean results.

### F11 — Human review decision

**Priority:** P0 | **Frontend owner:** A | **Backend owner:** B | **PRD:** FR-7

- [ ] A: Offer proceed, pause, reject, and needs_more_info, with an optional note.
- [ ] A: Explain that this records a review decision and show confirmation after saving.
- [ ] B: Implement POST /api/v1/analyses/{analysis_id}/review with decision validation.
- [ ] B: Derive reviewer identity from authentication where enabled; do not trust an arbitrary client identity.
- [ ] B: Store the review as an audit event; return review ID, decision, timestamp, and payment_action_taken: false.
- [ ] Both: Agree whether multiple reviews are allowed and whether completion is required before review.
- [ ] B: Return 404, 409, or 400 as appropriate; prevent unintended duplicate events on retries.

**Done when:** A decision and note are saved and confirmed without initiating, approving, or blocking a payment.

### F12 — PayPal readiness and optional verification

**Priority:** P0 status endpoint; P1 real lookup conditional on access | **Backend owner:** B | **Frontend owner:** A | **PRD:** FR-8

- [ ] B: Implement GET /api/v1/integrations/paypal/status with truthful configured, authorized, environment, capability, and missing-requirement fields.
- [ ] A: Show integration readiness and unavailable-state explanations.
- [ ] A: Offer a lookup request only when supported; display whether each analysis actually used PayPal data.
- [ ] B: If real sandbox access is available, implement and test a supported resource lookup with credentials stored on the server.
- [ ] B: Identify which lookup data supports each comparison.
- [ ] B: Continue content analysis when PayPal is unavailable, unauthorized, or fails; record the reason and used: false.

**Done when:** The endpoint works even without credentials. PayPal is marked used only after relevant data was actually retrieved and used. Demo fixtures never imply a real lookup.

### F13 — Security, privacy, and reliability

**Priority:** P0 | **Backend owner:** B | **Frontend owner:** A

- [ ] B: Enforce authentication and record ownership for any shared/user-account deployment; test cross-user access restrictions.
- [ ] B: Add content limits, rate limits, structured errors, and request IDs.
- [ ] B: Protect secrets and avoid logging raw submitted content by default.
- [ ] B: Implement/document retention and cleanup behavior for content, evidence, and reviews.
- [ ] B: Define idempotency-key behavior for create/review retries as recommended by the contract.
- [ ] A: Explain processing/retention behavior in the submission flow and sanitize evidence rendering.
- [ ] Both: Keep logs, committed fixtures, and screenshots free of real credentials or sensitive customer information.

**Done when:** Shared deployments protect record access, errors are safe and structured, retry behavior is defined, and retention behavior is documented. A local single-user demo must clearly record its deployment limitation.

### F14 — Verification, demo, and handoff

**Priority:** P0 | **Owner:** A leads integration/demo; B leads backend tests

- [ ] A: Maintain synthetic demo inputs and expected behaviors.
- [ ] B: Test validation, detection/extraction outputs, comparison rules, job failures, evidence spans, review recording, and integration fallback.
- [ ] A: Check forms, polling, empty/error states, evidence rendering, accessibility, and review confirmation against the real API.
- [ ] Both: Run every scenario in section 6 end to end.
- [ ] Both: Write README setup instructions, environment placeholders, known limitations, and demo steps.
- [ ] Both: Verify the deployed frontend reaches the backend and secrets stay server-side.

**Done when:** A fresh checkout can be started using the README, and the complete demo works with real API responses.

## 4. API ownership and handoff

| Endpoint | B implements | A connects |
|---|---|---|
| POST /api/v1/analyses | Validation, job creation, 202 response | Submit content/reference data, retain ID |
| GET /api/v1/analyses/{analysis_id} | Status and result report | Poll and render report/failure |
| GET /api/v1/analyses/{analysis_id}/evidence | Evidence and source spans | Evidence viewer and highlighting |
| POST /api/v1/analyses/{analysis_id}/review | Validation and audit-event storage | Decision form and save confirmation |
| GET /api/v1/integrations/paypal/status | Readiness and capability flags | Status display and lookup availability |

B should deliver one valid completed report example early, before all analysis logic is finished. A should build against that agreed shape, then switch to the real endpoints. Keep fixture mode visibly labeled and out of the final live-analysis path.

## 5. Suggested build order

### Milestone 1 — Agree and unblock each other

**Together:** Finish initial decisions, agree response examples, and inspect PayPal access.

**A:** Prepare synthetic sample scenarios and the submission/report skeleton using fixtures.

**B:** Set up the API, storage, and create/status lifecycle.

**Exit condition:** Both sides run locally and use the same request/response shapes.

### Milestone 2 — Connect the smallest complete flow

**A:** Connect submission, polling, a basic report, and failed-state feedback.

**B:** Produce a basic evidence-backed analysis result through the real processing path.

**Exit condition:** Pasting text produces a real report in the UI without manually copying JSON.

### Milestone 3 — Complete the core product

**A:** Complete reference inputs, financial comparisons, evidence viewer, and review controls.

**B:** Complete detection, extraction, verification, risk rules, evidence retrieval, and review storage.

**Exit condition:** A changed-payment message can be reviewed against a trusted record with inspectable evidence and a saved decision.

### Milestone 4 — Verify and prepare the demo

**A:** Run end-to-end scenarios, polish errors/accessibility, and write demo instructions.

**B:** Complete security/reliability checks and truthful integration status; add real PayPal lookup if access and time permit.

**Together:** Fix integration issues, run the full checklist, and verify deployment.

**Exit condition:** Every required scenario passes and the demo accurately states what was checked.

## 6. Shared acceptance scenarios

| Scenario | Expected behavior |
|---|---|
| Clean message | No invented threat; explain the assessment and verification limits |
| Prompt injection | Specific instruction is flagged with evidence and is never followed |
| Payment change plus urgency | Explain each supported indicator and recommend appropriate review |
| Explicit recipient/account mismatch | Report mismatch against the identified trusted value |
| Vague “new recipient” phrase | Flag change intent; use uncertainty/missing value rather than a fabricated account mismatch |
| Amount mismatch | Show extracted amount, reference amount, currency context, and source |
| Partial/no trusted reference | Continue analysis; label unavailable comparisons accurately |
| PayPal absent, unauthorized, or lookup failure | Continue where possible; show used: false and the reason |
| PayPal lookup succeeds, if implemented | Show used: true only for relevant retrieved data and identify its comparison source |
| Empty, oversized, malformed, or unsupported input | Return the documented validation error and show useful UI feedback |
| Detector/worker failure | Reach failed or explicitly limited output; do not show a successful complete check |
| Evidence span | Displayed exact excerpt matches its documented normalized-text span |
| HTML/script in submitted content | Display safely without execution |
| Review save/retry | Save the decision with payment_action_taken: false; avoid unintended duplicates |
| Unauthorized record access | Deny access without exposing another user's content |

## 7. Contract details to settle together

These are small gaps to resolve before their related features are implemented. Update the API contract so both people follow the same decision.

- [ ] Clarify when overall_risk may be unknown and the fields present in queued/processing/failed responses.
- [ ] Define comparison identifiers/evidence links so discrepancies are as traceable as findings; PRD FR-5 requires stable identifiers and evidence for discrepancies too.
- [ ] Make source mandatory on comparisons; it is absent from the completed-response example but present in the suggested schema.
- [ ] Correct illustrative evidence excerpts/offsets and define normalized text availability for frontend highlighting, including Unicode offset semantics.
- [ ] Correct the example that treats “new recipient” as a verified payee/payment-destination mismatch.
- [ ] Define unsupported option behavior and how skipped/failed checks appear in the report.
- [ ] Align the general paypal_context_used wording with the response's paypal_context.used field; choose one documented representation.
- [ ] Confirm review history/repeat-review rules, pre-completion review behavior, and idempotency semantics.

## 8. Keep these out of the first MVP

| Deferred feature | Why it is separate |
|---|---|
| PDF/file uploads, OCR, attachment scanning | Require an upload/extraction mechanism and content-type-specific validation |
| Automatic inbox or website monitoring | Require source integrations and additional authorization |
| Automatic payment execution | Outside the current review-recording API |
| Universal PayPal interception/blocking | Not supported by this product contract |
| Malware detection for arbitrary binary files | Outside the text-first analysis scope |
| History dashboard, exports, notifications | Useful additions after the required analysis/review workflow |

## 9. Simple collaboration rules

1. Use one task/feature branch at a time and include its feature ID in the branch or change description.
2. A owns frontend files; B owns backend files. Coordinate before editing shared schemas or configuration.
3. Agree API changes together and update examples immediately.
4. Integrate early and regularly; do not wait until both halves are finished.
5. Use checklist boxes to track completion. Mark a feature done only after its acceptance condition works through the connected application.
6. At each check-in, share what works, what is blocked, and what response/data you need from the other person next.

**Your first task:** A starts F01 sample scenarios and the F02/F03 forms. B starts F01 response examples and F04 API/job setup. Connect F02 to F04 before polishing the report.
