# Testing

1. **Book property tests:** random order streams; invariants (never crossed after processing, quantity conserved, price-time priority, replay determinism).
2. **FIX conformance suite:** the scripts members receive, run in CI against the gateway.
3. **Risk golden tests:** Python reference -> golden outputs; C++ matches to tolerance.
4. **Backtest harness:** margin report from the catalyst dataset with synthetic price paths.
5. **End-to-end scripted flow:** order -> fill -> collateral lock -> settlement -> payout -> collateral release.
6. **Degradation tests:** collateral service down -> reject all orders; gateway reconnect -> resend.
7. **Future (when margin offered):** PB simulator with accept/reject/latency injection; margin budget enforcement; VM settlement flow.
