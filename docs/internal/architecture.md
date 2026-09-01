# System architecture (internal)

**Version 4.0.** Full collateral at launch. Margin (via PB, CCP, or alternative) is a future upgrade. 24/7 continuous trading.

## Build / buy

| Component | Decision | Why |
|---|---|---|
| Reference data, contract engine, trades, positions, marks, risk engine, margin and fees, settlement, pre-trade risk, collateral service | **Build** | Core product |
| Matching engine, FIX gateway, market data publisher | Build in phase 1 (simplified); evaluate commercial FIX engine for production | Binary book is small; in-house lets us test end-to-end now |
| Surveillance | Buy (SMARTS or equivalent); we supply feeds | FCA expects a recognised system |
| Transaction reporting submission | Buy (ARM); we build the extract | Plumbing |
| Trading UI | Build later, thin, on the REST/WS API | Members use their own OMS |

Full diagrams: [End-to-end views](../diagrams.md).

## Services

```
Members ──FIX──▶ Gateway ──▶ [Engine: PreTrade → Matching] ──┬──▶ MarketData ──▶ Members
                                       ▲                       ├──▶ Trades
                                       │ balance snapshots     │
                                  Collateral service ◀─────────┘
                                  RiskEngine (shadow mode) ◀── Positions
                                  ReferenceData ──▶ all
                                  Settlement ──▶ Trades / Positions / Collateral
                                  Reporting & surveillance ◀── all
                                  REST/WS API ──▶ read models
```

Pre-trade risk runs as a **stage inside the engine shard process** (no IPC hop). At launch it checks available balance (full collateral). Future: PB limits and margin budget when margin is offered.

## Collateral service

Manages member balances. See [Collateral service](services/collateral.md).

- Tracks deposits, locked collateral, available balance, settlement payouts, withdrawals
- Pushes balance snapshots to the engine pre-trade stage via `collateral.balances`
- On trade: locks max loss from both sides
- On settlement: distributes payout, releases locks
- No variation margin — locks stay fixed between trade and settlement

## Clearing layer

**Full collateral (launch).** The collateral service handles everything: balance checks, locks, settlement payouts. No external dependency. No VM — locks stay fixed until settlement.

**Future margin mode.** How to offer capital efficiency (margin/leverage) is an open design question — PB integration, CCP, blockchain, or SWANS-managed margin. A PB/CCP adapter could be added without changing the core system.

## Trading hours

**24/7 continuous trading.** The matching engine runs without interruption. There is no opening or closing.

- **Settlement determination:** happens when the source publishes, regardless of time. Two-officer process operates during business hours; automated sources can trigger at any time.
- **Maintenance:** rolling hot-deploy; no planned downtime windows. If maintenance requires a halt, 24-hour notice to members.
- **Reference price:** filtered fair mark.

## Deployment

- **Production:** LD4 (Equinix Slough). Two racks, dedicated switches, member cross-connects.
- **DR:** second site, warm standby, journal shipping; RTO 2 hours **[confirm RTS 7]**.
- **Prototype:** one VM per service or Docker Compose on a single host.

## Time

`int64_t` nanoseconds since Unix epoch, UTC, everywhere. Production clocks disciplined by PTP; RTS 25 traceability to UTC.

## Contract economics

- Price in [0.005, 0.995], ticks of 0.005; payout 100 units; currencies GBP/USD/EUR.
- Full collateral: max loss locked at trade time. No margin engine dependency for pre-trade.
- One book per contract in yes-space.
- Packages are separate products with their own bounded payoff.
- **Payout structure is not final.** See [Open decisions #2](open-decisions.md).

## Non-goals for phase 1

Auctions, iceberg orders, HA failover, hardware timestamps, multicast market data, real ARM submission, PB adapter, CCP adapter, margin mode.
