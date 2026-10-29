# 03: Analysis Execution, Persistence & Status Retrieval

**Assigned work:** A: A04 (polling/recovery). B: B02 (storage/lifecycle/retrieval). See [two-person ticket board](../../../TEAM_TICKETS.md). A uses mocked lifecycle responses; B tests through HTTP. C00 selects 202 creation with queued/processing/completed/failed status polling over older synchronous alternatives.

**What to build:**
Analysis lifecycle management and persistent storage enabling analysis reports to survive browser page refreshes, enforcing server-side access checks, and allowing users to retrieve current status and completed results reliably.

**Blocked by:** 01: Dev Scaffold, Shared Schemas & Mock Fixtures, 02: Content Submission & Analysis Creation

**Status:** ready-for-agent

- [ ] Persist analysis records, statuses, and timestamps in PostgreSQL via SQLAlchemy or SQLModel.
- [ ] Implement `GET /api/v1/analyses/{id}` to retrieve current status and completed reports, ensuring reports survive page refreshes.
- [ ] Enforce server-side access checks on analysis retrieval using unpredictable UUIDs and configured API key/tenant validation (do not rely on unverified `X-User-Id` headers as authentication).
- [ ] Implement analysis execution: prefer a straightforward async request/response flow; retain FastAPI `BackgroundTasks` only if deferred async processing is chosen.
- [ ] If asynchronous processing with status polling is used, support `queued`, `processing`, `completed`, and `failed` status states with bounded frontend polling and visible error feedback.
- [ ] Document clearly in architecture notes that in-process `BackgroundTasks` do not form a durable queue and can be interrupted by container restarts.
