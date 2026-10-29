# 01: Dev Scaffold, Shared Schemas & Mock Fixtures

**Assigned work:** A: C00, A01, A02. B: C00, B01. See [two-person ticket board](../../../TEAM_TICKETS.md). After C00, each scaffold and mock/backend work can proceed independently. New-board interface decisions supersede older alternatives below.

**What to build:**
A baseline developer setup providing a runnable Vite + React frontend, a runnable FastAPI backend, shared data contract schemas, PostgreSQL database configuration via SQLAlchemy or SQLModel, and mock fixture scenarios. This enables both teammates to run their environments locally and develop features against concrete API contracts without blocking on real backend services.

**Blocked by:** None (can start immediately)

**Status:** ready-for-agent

- [ ] Initialize frontend project with Vite, React, TypeScript, and Tailwind CSS.
- [ ] Initialize backend project with Python 3.11+, FastAPI, and Pydantic configuration via `.env.example` using `pydantic-settings`.
- [ ] Establish standardized environment variable template (`GEMINI_API_KEY`, `GEMINI_MODEL`, `DATABASE_URL`, `MAX_CONTENT_BYTES=200000`, `RETENTION_DAYS=7`, `API_KEY`, `PAYPAL_CLIENT_ID`, `PAYPAL_CLIENT_SECRET`, `PAYPAL_BASE_URL=https://api-m.sandbox.paypal.com`) ensuring no real secrets are committed.
- [ ] Configure database connection for PostgreSQL on Neon with SQLAlchemy or SQLModel.
- [ ] Establish shared contract schemas matching the API specification for requests, responses, findings, claims, comparisons, and error structures.
- [ ] Provide reusable mock JSON fixtures for core scenarios: clean message, prompt injection attempt, urgency/deception cues, altered payment details, and absent reference data.
- [ ] Explicitly label mock PayPal fixture data as `simulated`.
- [ ] Document local run commands and setup verification in a README or quickstart guide.
