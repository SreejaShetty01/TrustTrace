# 06: Trusted-Reference Comparison Engine

**Assigned work:** A: A03 reference form and A05 comparison table. B: B05 comparison engine. See [two-person ticket board](../../../TEAM_TICKETS.md). Use fixture claims to start B05 without waiting for extraction; A uses mock reports. JSON statuses are match, mismatch, missing_reference, missing_extracted_value, and unable_to_verify, superseding uppercase labels below.

**What to build:**
A comparison module allowing users to supply known trusted reference records (e.g., expected vendor, agreed amount, currency, due date, invoice ID) and deterministically evaluating extracted claims against this reference data to highlight matches and discrepancies.

**Blocked by:** 05: Financial-Claim Extraction Pipeline

**Status:** ready-for-agent

- [ ] Provide a form for entering expected trusted reference details (payee, expected amount, currency, invoice/reference ID, due date).
- [ ] Support partial reference submissions without requiring every optional field to be populated.
- [ ] Implement deterministic verification comparing extracted claims against reference records, producing field-level verdicts: `MATCH`, `MISMATCH`, `MISSING`, or `UNABLE_TO_VERIFY`.
- [ ] Display side-by-side expected and extracted values accompanied by supporting text evidence spans.
- [ ] Validate reference formats, rejecting malformed currencies, dates, or negative amounts with structured 422 errors.
- [ ] Enforce product boundary: discrepancies and mismatches prompt human review; they must never trigger an automatic payment decision or claim to block transactions.
