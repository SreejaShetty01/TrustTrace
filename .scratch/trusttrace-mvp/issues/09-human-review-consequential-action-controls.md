# 09: Human Review, Audit Trail & Deletion

**Assigned work:** A: A07 (controls/history). B: B08 (storage/access/deletion/retention). See [two-person ticket board](../../../TEAM_TICKETS.md). Use mock or stored fixture reports to start independently of the final report engine. Review wire values are proceed, pause, reject, and needs_more_info, superseding alternate values below. Review history and DELETE are documented extensions agreed in C00.

**What to build:**
A human-in-the-loop review workflow, audit trail recording, and data deletion capability allowing reviewers to register decisions, view historical audit events, and delete analysis records on demand.

**Blocked by:** 08: Evidence-Backed Risk Report & Findings Viewer

**Status:** ready-for-agent

- [ ] Provide review decision controls on the report view supporting agreed decisions: `approved`, `flagged_suspicious`, `escalated`, and `rejected`.
- [ ] Implement `POST /api/v1/analyses/{id}/review` to persist reviewer decisions, optional notes, and UTC timestamps.
- [ ] Ensure audit records communicate that reviewer approval reflects human judgment and does not constitute proof that content is safe.
- [ ] Enforce server-side access checks on review submission and analysis retrieval.
- [ ] Support data retention policies with default `RETENTION_DAYS=7` and documented expiry enforcement.
- [ ] Implement `DELETE /api/v1/analyses/{id}` to permanently purge an analysis record and all its associated findings, claims, evidence, and review entries.
- [ ] Update report views to reflect recorded review history and prevent accidental duplicate decisions.
