# Build plan

| Milestone | Weeks | Deliverable | Depends on |
|---|---|---|---|
| M1 | 1-6 | `refdata_gen` + reference data service on the catalogue; trades and position services on a scripted stream; `libswansrisk` interfaces with first `EsMarginModel` (MPOR + terminal engines); margin service producing PB margin schedule and price/risk file; collateral service core (balance tracking, locks); two-engine divergence reporting | Catalogue CSV; margin methodology docs; risk engine spec v3.0 |
| M2 | 6-12 | Engine with pre-trade stage (margin budgets + balance checks); QuickFIX gateway; market data over FIX and WebSocket; PB adapter (FIX drop copy + margin delivery); collateral service (VM transfers, settlement payouts); end-to-end scripted day with PB simulator; PB API for limits/kill switch; full-collateral end-to-end flow | M1 |
| M3 | 12-20 | Settlement service (four-eyes, disputes, fallback); VM window marks; VM; margin call lifecycle (full state machine); RTS 22/24 extracts; SMARTS feed; REST API; model governance registry; backtesting framework | M2 |
| M4 | post PB onboarding | Real PB integration; RTS 26 timings; commercial FIX decision; LD4 deployment; DR | PB agreement signed |

M1 first: its output (margin schedule, price/risk file, backtest, two-engine comparison) is what PB meetings need.

## Development epics (aligned with risk engine spec v3.0)

| Epic | Key outputs |
|---|---|
| E1 Domain model | Member/account/product/trade/position/collateral schemas + migrations; account hierarchy (member → origin → account → margin account) |
| E2 Connectivity | FIX gateway, REST API, PB adapter, SFTP batch processor |
| E3 Ledger & event backbone | Aeron streams, append-only journals, idempotent event processing, reconciliation |
| E4 Fair marks | Market ingestion, mark hierarchy, structural projection, mark quality, VM window snapshots |
| E5 Quant core | Scenario API, MPOR simulator, terminal engine, structural state engine, VaR/ES/stress, two-engine consistency contract |
| E6 Margin policy | IM components, event ramp, netting layers, margin call lifecycle, limits, hypothetical margin |
| E7 Collateral & VM | Collateral service (locks, transfers, payouts), eligibility/haircuts, balance checks, VM settlement |
| E8 Reporting | Member reports, PB margin schedules, price/risk files, regulatory extracts |
| E9 Governance | Model registry, audit/lineage, backtesting, overrides, two-engine divergence monitoring |
| E10 Production engineering | HA, security, observability, DR, performance certification |

## Future milestones (post-launch)

| Milestone | Deliverable | Trigger |
|---|---|---|
| M5 | CCP adapter; CCP parameter file; default fund integration; auction tooling | CCP partnership secured |
| M6 | Auctions, iceberg orders, multicast market data, HA failover | Volume/demand justifies |
| M7 | Calibrated liquidity add-ons (phase 2); broader portfolio-margin offsets; live-data close-out calibration | Data sufficiency + independent validation |
| M8 | Partner-DCO deployment; multi-jurisdiction reporting profiles | Governance/regulatory approval |
