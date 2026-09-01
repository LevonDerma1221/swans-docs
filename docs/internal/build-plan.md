# Roadmap

## What we're building and when

### Phase 1 — Foundations (weeks 1–6)
- Reference data service and contract catalogue
- Trades and position services
- Collateral service (balance tracking, locks, settlement payouts)
- Risk library: first model with both engines running in shadow mode
- Price/risk file output

This phase proves the margin engine works — even though we launch on full collateral, we need the analytics running to demonstrate capability.

### Phase 2 — Trading (weeks 6–12)
- Matching engine with pre-trade balance checks
- FIX gateway (members can connect and trade)
- Market data over FIX and WebSocket
- REST API
- Full end-to-end flow: deposit → order → match → lock → settle → payout

### Phase 3 — Production readiness (weeks 12–20)
- Settlement service (two-officer process, disputes, fallback)
- Marks pipeline
- Regulatory extracts (RTS 22/24)
- Surveillance feed (SMARTS)
- Model governance and backtesting

### After launch
- Margin mode (PB, CCP, blockchain, or alternative) — depends on strategic decision
- Auctions, iceberg orders, multicast market data
- Broader portfolio-margin offsets
- Multi-jurisdiction; retail access if regulatory path found

## Work streams

| Stream | What it delivers |
|---|---|
| Domain model | Member, account, product, trade, position, collateral schemas |
| Connectivity | FIX gateway, REST API |
| Event backbone | Aeron streams, journals, idempotent processing |
| Fair marks | Market ingestion, mark hierarchy, quality checks |
| Quant core | Scenario API, MPOR simulator, terminal engine, VaR/ES/stress |
| Margin policy | IM components, event ramp, netting layers (all shadow mode at launch) |
| Collateral | Locks, settlement payouts, balance checks, deposits, withdrawals |
| Reporting | Member reports, price/risk files, regulatory extracts |
| Governance | Model registry, audit, backtesting, divergence monitoring |

## Testing

1. **Order book tests** — random order streams; verify price-time priority, quantity conservation, replay determinism
2. **FIX conformance** — scripted test suite run against the gateway
3. **Risk golden tests** — Python reference produces golden outputs; C++ must match
4. **Backtest harness** — margin model validated against synthetic price paths
5. **End-to-end flow** — order → fill → collateral lock → settlement → payout
6. **Failure tests** — collateral service down → all orders rejected; gateway reconnect → resend
