# 07: PayPal Adapter & Integration Status

**Assigned work:** A: A06 (status UI). B: B07 (status/adapter). See [two-person ticket board](../../../TEAM_TICKETS.md). A tests all modes through fixtures without credentials. B can complete truthful status/fallback first; real sandbox lookup is conditional on verified access. Simulated data never sets real PayPal context used to true.

**What to build:**
A tri-modal PayPal verification adapter that verifies invoice/order records against the PayPal Sandbox when credentials are configured, provides an explicitly simulated mock mode for offline testing, and reports transparent integration availability without overpromising capabilities.

**Blocked by:** 01: Dev Scaffold, Shared Schemas & Mock Fixtures

**Status:** ready-for-agent

- [ ] Implement explicit three-mode operation: `sandbox`, `mock`, and `unavailable`.
- [ ] Implement `GET /api/v1/integrations/paypal/status` reporting the current mode, availability, and checked resource details.
- [ ] Connect to PayPal Sandbox Invoicing API using server-side OAuth 2.0 credentials (`PAYPAL_CLIENT_ID`, `PAYPAL_CLIENT_SECRET`) against `PAYPAL_BASE_URL` (defaulting to `https://api-m.sandbox.paypal.com`).
- [ ] Provide an offline `mock` mode for demonstrations, explicitly labelling mock results as `simulated` in both API responses and UI components.
- [ ] Automatically fall back to `unavailable` with clear explanatory messaging when credentials or required permissions are absent.
- [ ] Keep all credentials on the backend; never expose client secrets to the browser or commit them to source control.
- [ ] Strictly respect product boundaries: never claim payment interception, transaction blocking, or verified status unless active verified sandbox data is confirmed.
