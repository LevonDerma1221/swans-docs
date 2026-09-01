# Build plan

| Milestone | Weeks | Deliverable | Depends on |
|---|---|---|---|
| M1 | 1-6 | `refdata_gen` + reference data service; trades and position services; `libswansrisk` interfaces with first model (MPOR + terminal engines in shadow mode); collateral service core (balance tracking, locks, settlement payouts); price/risk file | Catalogue CSV; margin methodology docs; risk engine spec |
| M2 | 6-12 | Engine with pre-trade stage (balance checks); QuickFIX gateway; market data over FIX and WebSocket; full-collateral end-to-end flow (deposit → order → match → lock → settle → payout); REST API | M1 |
| M3 | 12-20 | Settlement service (four-eyes, disputes, fallback); marks pipeline; RTS 22/24 extracts; SMARTS feed; model governance registry; backtesting framework | M2 |
| M4 | post launch | Margin mode (PB integration, CCP, or alternative); VM; margin call lifecycle; real PB/CCP adapter; LD4 deployment; DR | Strategic decision on how to offer margin |

M1 first: its output (risk analytics, price/risk file, backtest, two-engine comparison) demonstrates the margin engine capability.

## Development epics

| Epic | Key outputs |
|---|---|
| E1 Domain model | Member/account/product/trade/position/collateral schemas + migrations |
| E2 Connectivity | FIX gateway, REST API |
| E3 Ledger and event backbone | Aeron streams, append-only journals, idempotent event processing, reconciliation |
| E4 Fair marks | Market ingestion, mark hierarchy, structural projection, mark quality |
| E5 Quant core | Scenario API, MPOR simulator, terminal engine, structural state engine, VaR/ES/stress, two-engine consistency contract |
| E6 Margin policy | IM components, event ramp, netting layers, limits, hypothetical margin (all shadow mode at launch) |
| E7 Collateral | Collateral service (locks, settlement payouts), balance checks, deposits, withdrawals |
| E8 Reporting | Member reports, price/risk files, regulatory extracts |
| E9 Governance | Model registry, audit/lineage, backtesting, overrides, two-engine divergence monitoring |
| E10 Production engineering | HA, security, observability, DR, performance certification |

## Future milestones (post-launch)

| Milestone | Deliverable | Trigger |
|---|---|---|
| M5 | Margin mode: PB adapter or CCP adapter or alternative (blockchain, SWANS-managed margin); VM; margin calls | Strategic decision + partnership/regulatory approval |
| M6 | Auctions, iceberg orders, multicast market data, HA failover | Volume/demand justifies |
| M7 | Calibrated liquidity add-ons; broader portfolio-margin offsets; live-data close-out calibration | Data sufficiency + independent validation |
| M8 | Multi-jurisdiction; retail access if regulatory path found | Governance/regulatory approval |
