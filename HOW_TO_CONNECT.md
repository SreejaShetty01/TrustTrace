# Connecting the Two Teammates' Work

Assignments and acceptance criteria: [TEAM_TICKETS.md](TEAM_TICKETS.md).

**A owns:** React frontend and mock API. **B owns:** FastAPI backend, analysis pipeline, and database. Both implement the same JSON interface, then A switches the frontend from mock transport to real HTTP.

## 1. Before working separately — C00

Agree these once and record the answers in `contracts/README.md`:

- API routes, request/response fields, enums, and structured errors from [API_CONTRACT-1.md.txt](API_CONTRACT-1.md.txt).
- POST create returns 202; the frontend polls GET for completion or failure.
- Evidence stays on the separate evidence endpoint. Define normalized text availability, end-exclusive offsets, and Unicode character counting so Python offsets and JavaScript highlighting agree.
- Review history, DELETE, simulation metadata, and any added comparison/evidence identifiers are documented contract extensions.
- Local ports, allowed origins, access/session behavior, and credentials/header requirements.
- How an Idempotency-Key is reused for a retry of the same submission/review and how a genuinely new action gets a new key.

Use these exact wire values:

| Field | Values |
|---|---|
| status | queued, processing, completed, failed |
| overall_risk | low, medium, high, critical, unknown |
| comparison status | match, mismatch, missing_reference, missing_extracted_value, unable_to_verify |
| review decision | proceed, pause, reject, needs_more_info |

A creates synthetic JSON fixtures in `contracts/fixtures/`; B validates them against backend schemas. Include queued, processing, completed, failed, evidence, saved review, deletion, integration status, and error responses. These paths describe deliverables to create; this guide does not implement them.

## 2. Build independently

### A — Frontend

Create one API adapter exposing these operations. Components use this adapter rather than directly importing mock data or calling fetch in multiple places.

| Operation | HTTP route when live |
|---|---|
| createAnalysis(request) | POST /api/v1/analyses |
| getAnalysis(id) | GET /api/v1/analyses/{id} |
| getEvidence(id) | GET /api/v1/analyses/{id}/evidence |
| saveReview(id, request) | POST /api/v1/analyses/{id}/review |
| getPayPalStatus() | GET /api/v1/integrations/paypal/status |
| deleteAnalysis(id) | DELETE /api/v1/analyses/{id} — documented extension |

Implement mock and HTTP transports with the same inputs/outputs. Mock mode simulates lifecycle changes, failures, saved reviews, and deletion. Clearly label mock mode in the UI.

Proposed frontend configuration, to implement in A01:

```dotenv
VITE_API_MODE=mock
VITE_API_BASE_URL=http://localhost:8000
```

The base URL is the origin only; the adapter appends `/api/v1/...` once. Never place Gemini, PayPal, database, or a shared backend API secret in a VITE variable: these values are public in the browser bundle.

### B — Backend

Build Pydantic schemas against the agreed fixture JSON. Implement endpoints independently and exercise them with HTTP requests/backend tests. Use synthetic fixtures while detector/extractor modules are unfinished; never label fixture processing as real analysis.

Enforce access checks on every analysis, evidence, review, and deletion route. Do not trust an arbitrary X-User-Id header. Return the agreed safe error shape and truthful unavailable/partial-check states.

Keep actual database/provider credentials in backend environment settings. Use Neon migrations and test record persistence without needing the frontend.

## 3. Backend handoff — B09

B gives A this filled-in checklist, without sending secrets in chat:

- [ ] API origin: __________
- [ ] Allowed frontend origin: __________
- [ ] Server-validated access/session or proxy setup: __________
- [ ] Whether requests require cookies or a user-specific authorization header: __________
- [ ] Confirmation that schemas match the agreed fixtures.
- [ ] Confirmation that a synthetic analysis can be created/retrieved through HTTP.
- [ ] Confirmation that report, evidence, review, and deletion work for that record.
- [ ] PayPal mode/capabilities and any intentionally unavailable checks: __________
- [ ] Backend startup/migration instructions and known deployment limitations.

If using a shared API key to protect the demo, keep it in a server-side proxy and have the browser call that proxy. Alternatively use properly validated user/session credentials. Do not embed the shared key in frontend code or use an unverified user ID as authentication.

## 4. Connect locally — I01

1. B starts the backend/database and confirms direct API requests work.
2. A starts the frontend. A typical setup is frontend `http://localhost:5173`, backend `http://localhost:8000`; agree actual ports rather than assuming them.
3. B allows the exact frontend origin in CORS, including required methods and headers. If cookie sessions are selected, configure credentials on both sides and avoid wildcard origins.
4. A sets `VITE_API_MODE=live` and `VITE_API_BASE_URL` to the agreed origin (or proxy origin), then restarts Vite so environment changes are loaded.
5. A submits one synthetic message. Confirm the browser makes a real POST and receives 202 plus an analysis ID.
6. Poll GET for that ID until completed/failed. Stop polling on terminal status, navigation, or bounded timeout; do not resubmit just because polling timed out.
7. Fetch evidence and verify excerpts/highlighting against the normalized text. Include Unicode text to catch offset mismatches.
8. Save a review, reload, and verify it persists. Confirm `payment_action_taken` is false.
9. Delete the synthetic record with confirmation and verify it is no longer accessible.
10. Check PayPal unavailable behavior first; test simulated mode separately. Real `used` stays false unless relevant PayPal data was actually retrieved and used.

**Connection is complete when:** The workflow succeeds with mock transport disabled, refresh retrieves persisted backend data, and no component changes were needed to switch transports.

## 5. Verify together — I02

- [ ] Clean message; suspicious instructions; prompt injection.
- [ ] Explicit amount and recipient mismatch; vague recipient; no/partial reference.
- [ ] Malformed/empty/oversized input and structured reference errors.
- [ ] Provider failure, failed job, polling timeout, evidence-fetch error.
- [ ] Unavailable/simulated PayPal; real sandbox only if implemented and authorized.
- [ ] Review retry does not accidentally duplicate an audit event.
- [ ] Unauthorized record access is denied and evidence renders safely.
- [ ] Deletion and retention cleanup behave as documented.

A records browser results in `docs/integration-checklist.md`. B fixes backend issues; A fixes frontend issues. Agree any schema changes before changing fixtures/types/models.

## 6. Connect the deployed versions

1. B deploys FastAPI to Render, configures server secrets and Neon, applies migrations, and verifies API operations.
2. A configures Vercel with live mode and the deployed API/proxy origin, then builds/redeploys. Vite values are set at build time.
3. B adds the exact deployed frontend origin to allowed origins. Configure preview origins explicitly if previews need API access.
4. Verify the chosen session/proxy behavior across deployment origins; local access behavior is not proof deployed sessions work.
5. Run the shared scenarios again, including browser refresh, saved review, and unavailable PayPal.
6. Check that no backend secret appears in frontend assets or responses. Document free-tier cold starts and in-process job restart limitations.

## Troubleshooting

| Symptom | Check |
|---|---|
| UI still shows fixture results | Mode flag, mock interceptors, Vite restart/rebuild, mock-mode badge |
| Requests go to the wrong route | API origin and duplicate/missing `/api/v1` |
| Browser blocks request but direct HTTP works | Exact CORS origin, allowed headers/methods, selected credential behavior |
| 401/403 | Session/proxy setup and server-side access validation |
| UI cannot render backend result | JSON shape, lowercase enums, null handling, documented extensions |
| Polling never ends | Worker execution, persisted state, timeouts, restart recovery |
| Highlight selects wrong text | Normalized text, Unicode counting, end-offset convention |
| Review disappears after refresh | Database write and agreed persisted review-history retrieval |

All ticket checkboxes remain uncompleted until their implementation and acceptance checks have actually been performed.
