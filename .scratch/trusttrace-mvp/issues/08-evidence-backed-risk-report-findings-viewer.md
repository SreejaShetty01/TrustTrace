# 08: Evidence-Backed Risk Report & Findings Viewer

**Assigned work:** A: A05 (viewer). B: B06 (risk/report/evidence API). See [two-person ticket board](../../../TEAM_TICKETS.md). Original dependencies apply to final connected acceptance, not starting the viewer: A can build against fixtures immediately. Support unknown risk as defined by the API contract.

**What to build:**
A unified results report view combining threat findings, extracted financial claims, trusted-reference comparison results, and PayPal integration status into an intuitive, evidence-linked interface for users.

**Blocked by:** 03: Analysis Execution, Persistence & Status Retrieval, 04: Suspicious-Content & Prompt-Injection Detection, 06: Trusted-Reference Comparison Engine, 07: PayPal Adapter & Integration Status

**Status:** ready-for-agent

- [ ] Render a clear overall risk badge (`low`, `medium`, `high`, `critical`) alongside a plain-language summary and recommended safe next steps.
- [ ] Display an interactive evidence text viewer that visually highlights suspicious text spans corresponding to detection findings.
- [ ] Present financial claim comparison tables contrasting claimed vs expected values with visual indicators for matches and discrepancies.
- [ ] Clearly distinguish between observed text evidence, inferred risk ratings, user-provided reference data, and sandbox or simulated mock results.
- [ ] Display transparent integration status showing whether PayPal data was active, unavailable, or simulated.
- [ ] Avoid misleading precision; do not imply risk scores are statistically calibrated unless calibrated models are used.
