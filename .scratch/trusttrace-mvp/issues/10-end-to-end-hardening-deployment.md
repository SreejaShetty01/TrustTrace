# 10: End-to-End Hardening & Deployment

**Assigned work:** A: A08 (frontend checks/demo/Vercel). B: B09 (backend checks/Render/Neon). Both: I01/I02 connection and acceptance. See [two-person ticket board](../../../TEAM_TICKETS.md). Local checks/deployment preparation can start independently; final end-to-end acceptance requires both sides.

**What to build:**
Comprehensive end-to-end scenario verification, API error normalization, input and access controls, and cloud deployment pipelines ensuring TrustTrace runs reliably on Vercel, Render, and Neon PostgreSQL.

**Blocked by:** 09: Human Review, Audit Trail & Deletion

**Status:** ready-for-agent

- [ ] Execute comprehensive end-to-end verification covering: clean content, deceptive urgency cues, prompt injection attacks, altered financial details, absent reference data, Gemini API failures, and all three PayPal modes (`sandbox`, `mock`, `unavailable`).
- [ ] Enforce consistent API error formats across all endpoints conforming strictly to the API contract.
- [ ] Enforce CORS policies, rate limiting, and server-side request size and access checks.
- [ ] Verify that no real credentials or sensitive secrets are committed or leaked via client bundles.
- [ ] Deploy frontend on Vercel and backend on Render connected to Neon PostgreSQL.
- [ ] Verify data retention expiry and the on-demand `DELETE` workflow in the deployed environment.
- [ ] Provide a secret-free deployment and demo walkthrough guide documenting free-tier cold starts and service limitations.
