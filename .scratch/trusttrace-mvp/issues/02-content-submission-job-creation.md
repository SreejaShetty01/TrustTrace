# 02: Content Submission & Analysis Creation

**Assigned work:** A: A03 (form/client). B: B02 (validation/create endpoint). See [two-person ticket board](../../../TEAM_TICKETS.md). A uses A02 mocks; B uses HTTP tests. Keep 202 creation plus GET polling, as selected by C00; neither person's implementation blocks the other.

**What to build:**
The primary user submission workflow where a user pastes suspicious text into the interface, configures optional source metadata, triggers analysis, and receives immediate confirmation with a persisted analysis record and stable UUID.

**Blocked by:** 01: Dev Scaffold, Shared Schemas & Mock Fixtures

**Status:** ready-for-agent

- [ ] Provide a primary text input area with an Analyze action button and real-time character/byte counters.
- [ ] Enforce client-side and backend validation rejecting empty text and content exceeding `MAX_CONTENT_BYTES` (200,000 bytes default) with clear 413/400 error states.
- [ ] Implement `POST /api/v1/analyses` endpoint: define one unified success/error contract. If synchronous, return the full analysis/report resource; if asynchronous, return `202 Accepted` with a stable UUID and status URL aligned with Ticket 03.
- [ ] Ensure submitted content is treated strictly as data; never log raw submitted content in application or server logs.
- [ ] Transition the frontend upon submission into a responsive loading/pending state without full page refresh.
