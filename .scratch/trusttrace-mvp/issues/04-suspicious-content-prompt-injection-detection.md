# 04: Suspicious-Content & Prompt-Injection Detection

**Assigned work:** B: B03 (engine). A: A02 scenarios and A05 presentation only. See [two-person ticket board](../../../TEAM_TICKETS.md). Detection is backend-only; the UI uses fixtures until integration. Severity also permits info per the API contract. Clean fixture tests must pass; this does not promise zero false positives on arbitrary real content.

**What to build:**
An automated threat detection pipeline that analyzes submitted text for malicious patterns, prompt injection attacks against AI agents, social engineering urgency, impersonation, and altered payment instructions, combining deterministic rules with model assistance.

**Blocked by:** 01: Dev Scaffold, Shared Schemas & Mock Fixtures

**Status:** ready-for-agent

- [ ] Implement deterministic pattern rules to flag common urgency keywords, suspicious links, and changed payment phrases.
- [ ] Implement model-driven analysis using Google Gemini API, configured dynamically via `GEMINI_API_KEY` and `GEMINI_MODEL` (do not hardcode specific model strings in code).
- [ ] Treat all submitted content strictly as untrusted data placed inside isolated prompt fences; ensure submitted text cannot execute commands, trigger tools, or override detector instructions.
- [ ] Output structured findings containing category, severity (`low`, `medium`, `high`, `critical`), explanatory title, detailed reasoning, and exact character/text evidence offsets.
- [ ] Gracefully handle missing credentials, quota/rate limits, invalid model configuration, and external API failures with safe fallbacks and explicit error reporting.
- [ ] Clearly communicate uncertainty; wording must never claim definitive or guaranteed fraud detection.
- [ ] Accurately report clean messages with zero false alarm findings.
