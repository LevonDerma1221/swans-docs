# System architecture (internal)

**Version 3.0.** Full-collateral as day-one default + optional PB-managed mode. 24/7 continuous trading. CCP clearing is a future upgrade path.

## Build / buy

| Component | Decision | Why |
|---|---|---|
| Reference data, contract engine, trades, positions, marks, risk engine, margin and fees, settlement, pre-trade risk, collateral service | **Build** | Core product; what PBs and members evaluate |
| PB adapter | **Build** | FIX drop copy + margin schedule delivery; thin layer |
| Matching engine, FIX gateway, market data publisher | Build in phase 1 (simplified); evaluate commercial FIX engine for production | Binary book is small; in-house lets us test end-to-end now |
| Surveillance | Buy (SMARTS or equivalent); we supply feeds | FCA expects a recognised system |
| Transaction reporting submission | Buy (ARM); we build the extract | Plumbing |
| Trading UI | Build later, thin, on the REST/WS API | Members use their own OMS |

Full diagrams: [End-to-end views](../diagrams.md).

## Services

```
 Members ──FIX──▶ Gateway ──▶ [Engine shard: PreTrade → Matching] ──┬──▶ MarketData ──▶ Members
                                          ▲                          ├──▶ Trades ──▶ PB adapter
                                          │ margin budgets           │        │
                                          │ balance snapshots        │   Collateral service
                                     MarginService ◀── RiskEngine ◀── Positions ◀────┘
                                          │               ▲
          PB files, margin schedule ◀─────┘        ReferenceData ──▶ all
                                                          ▲
                                                  Settlement ──▶ Trades / Positions / PB adapter / Collateral
                                                  Reporting & surveillance extracts ◀── all
                                                  PB API ──▶ PreTrade limits
                                                  REST/WS API ──▶ read models
```

Pre-trade risk runs as a **stage inside the engine shard process** (no IPC hop), as a separate module. It checks both PB limits/margin budget (PB-managed accounts) and available balance (full-collateral accounts).

## Collateral service

Manages member balances. See [Collateral service](services/collateral.md).

- Tracks deposits, locked collateral, available balance, settlement payouts, withdrawals
- Pushes balance snapshots to the engine pre-trade stage via `collateral.balances`
- On trade: locks max loss from both sides
- On settlement: distributes payout, releases locks
- No variation margin — locks stay fixed between trade and settlement

## Clearing layer

**Full collateral (launch).** The collateral service handles everything: balance checks, locks, settlement payouts. No external dependency. No VM — locks stay fixed until settlement.

**Future margin mode.** How to offer capital efficiency (margin/leverage) is an open design question. A PB adapter or CCP adapter could be added without changing the core system.

## Trading hours

**24/7 continuous trading.** The matching engine runs without interruption. There is no opening or closing.

- **Settlement determination:** happens when the source publishes, regardless of time. Two-officer process operates during business hours; automated sources can trigger at any time.
- **Maintenance:** rolling hot-deploy; no planned downtime windows. If maintenance requires a halt, 24-hour notice to members.
- **Reference price:** filtered fair mark (not "daily settle").

## Deployment

- **Production:** LD4 (Equinix Slough). Two racks, dedicated switches, member cross-connects. Engine and gateway hosts pinned, isolated cores, kernel bypass optional later.
- **DR:** second site, warm standby, journal shipping; RTO 2 hours **[confirm RTS 7]**.
- **Prototype:** one VM per service or Docker Compose on a single host.

## Time

`int64_t` nanoseconds since Unix epoch, UTC, everywhere. Production clocks disciplined by PTP; RTS 25 traceability to UTC. Prototype uses NTP.

## Contract economics

- Price in [0.005, 0.995], ticks of 0.005, integer `price_ticks in [1, 199]`; payout 100 units; minimum notional $100; currencies GBP/USD/EUR, one per contract.
- **PB-managed accounts:** futures-style margining. No premium at trade time; VM at each window; IM from the margin service.
- **Full-collateral accounts:** max loss locked at trade time. VM transfers at each window. No margin engine dependency for pre-trade.
- One book per contract in yes-space.
- **Packages are separate products** with their own bounded payoff. Leg decomposition exists only inside SWANS's risk and position views.
- **Payout structure is not final.** The binary yes/no format, payout amount and settlement kind may evolve. The `ContractSpec` and margin engine treat the payoff as a pluggable function. See [Open decisions #2](open-decisions.md).

## Non-goals for phase 1

Auctions, iceberg orders, HA failover, hardware timestamps, multicast market data, real ARM submission, CCP adapter.
