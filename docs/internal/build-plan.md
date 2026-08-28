# Build plan

| Milestone | Weeks | Deliverable | Depends on |
|---|---|---|---|
| M1 | 1-6 | `refdata_gen` + reference data service on the catalogue; trades and position services on a scripted stream; `libswansrisk` interfaces with first `EsMarginModel`; margin service producing PB margin schedule and price/risk file | Catalogue CSV; margin methodology docs |
| M2 | 6-12 | Engine with pre-trade stage and margin budgets; QuickFIX gateway; market data over FIX and WebSocket; PB adapter (FIX drop copy + margin delivery); end-to-end scripted day with PB simulator; PB API for limits/kill switch | M1 |
| M3 | 12-20 | Settlement service (four-eyes, disputes, fallback); daily settle; VM; RTS 22/24 extracts; SMARTS feed; REST API | M2 |
| M4 | post PB onboarding | Real PB integration; RTS 26 timings; commercial FIX decision; LD4 deployment; DR | PB agreement signed |

M1 first: its output (margin schedule, price/risk file, backtest) is what PB meetings need.

## Future milestones (post-launch)

| Milestone | Deliverable | Trigger |
|---|---|---|
| M5 | CCP adapter; CCP parameter file; default fund integration | CCP partnership secured |
| M6 | Auctions, iceberg orders, multicast market data, HA failover | Volume/demand justifies |
