# 05: Financial-Claim Extraction Pipeline

**Assigned work:** B: B04 (extraction). A: A02 scenarios and A05 presentation only. See [two-person ticket board](../../../TEAM_TICKETS.md). B tests text inputs independently; A renders agreed claim fixtures without waiting for extraction.

**What to build:**
An extraction engine that identifies and structures financial claims present in unstructured messages, emails, or invoices using deterministic extraction and model assistance, providing evidence linkages and explicit uncertainty handling.

**Blocked by:** 01: Dev Scaffold, Shared Schemas & Mock Fixtures

**Status:** ready-for-agent

- [ ] Extract monetary amounts, numeric values, and currency codes (e.g. USD, EUR, GBP) with supporting text offsets.
- [ ] Extract beneficiary/payee names, billing entity identities, invoice/order reference numbers, and due dates.
- [ ] Detect claims that payment details or accounts have changed, alongside the new destination account or payment link.
- [ ] Tie all extracted fields directly to source evidence text spans.
- [ ] Represent absent, unspecified, or uncertain fields as `null`/unknown rather than inventing or hallucinating plausible values.
- [ ] Preserve both original raw string expressions and normalized parsed values where useful.
- [ ] Never silently assume a default currency when currency symbols or codes are absent.
