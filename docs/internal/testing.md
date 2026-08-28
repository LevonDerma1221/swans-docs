# Testing

1. **Book property tests:** random order streams; invariants (never crossed after processing, quantity conserved, price-time priority, replay determinism).
2. **FIX conformance suite:** the scripts members receive, run in CI against the gateway.
3. **PB simulator:** accept/reject with latency and failure injection; verifies state machine and pending-position reversion.
4. **Risk golden tests:** Python reference -> golden outputs; C++ matches to tolerance.
5. **Backtest harness:** margin report from the catalyst dataset with synthetic price paths; deliverable one.
6. **End-to-end scripted day:** order -> fill -> PB accept -> position -> EOD margin -> PB file -> settlement -> cash flow.
7. **Degradation tests:** margin service down -> fast-bound mode; PB kill -> cancel within 1 s; gateway reconnect -> resend.
