# System architecture (internal)

**Version 2.0.** Bilateral/PB launch model. CCP clearing is a future upgrade path.

## Build / buy

| Component | Decision | Why |
|---|---|---|
| Reference data, contract engine, trades, positions, marks, risk engine, margin and fees, settlement, pre-trade risk | **Build** | Core product; what PBs and members evaluate |
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
                                     MarginService ◀── RiskEngine ◀── Positions ◀────┘
                                          │               ▲
          PB files, margin schedule ◀─────┘        ReferenceData ──▶ all
                                                          ▲
                                                  Settlement ──▶ Trades / Positions / PB adapter
                                                  Reporting & surveillance extracts ◀── all
                                                  PB API ──▶ PreTrade limits
                                                  REST/WS API ──▶ read models
```

Pre-trade risk runs as a **stage inside the engine shard process** (no IPC hop), as a separate module.

## Clearing layer: PB adapter

The PB adapter is a service that:
- Sends `TradeCaptureReport` (FIX drop copy) to PBs on each match
- Delivers margin schedule updates (IM/VM per account) to PBs
- Delivers daily price and risk file to PBs
- Delivers settlement values to PBs for cash settlement
- Receives PB limit updates (max order size, notional limits, kill switch)

This is architecturally identical to a CCP adapter — same interface, different counterparty. When a CCP is available, swap the adapter.

## Deployment

- **Production:** LD4 (Equinix Slough). Two racks, dedicated switches, member cross-connects. Engine and gateway hosts pinned, isolated cores, kernel bypass optional later.
- **DR:** second site, warm standby, journal shipping; RTO 2 hours **[confirm RTS 7]**.
- **Prototype:** one VM per service or Docker Compose on a single host.

## Time

`int64_t` nanoseconds since Unix epoch, UTC, everywhere. Production clocks disciplined by PTP; RTS 25 traceability to UTC. Prototype uses NTP.

## Contract economics (locked)

- Price in [0.005, 0.995], ticks of 0.005, integer `price_ticks in [1, 199]`; payout 100 units; minimum notional $100; currencies GBP/USD/EUR, one per contract.
- **Futures-style margining.** No premium at trade time; daily VM; IM from the margin service.
- One book per contract in yes-space.
- **Packages are separate products** with their own bounded payoff. Leg decomposition exists only inside SWANS's risk and position views.

## Non-goals for phase 1

Auctions, iceberg orders, HA failover, hardware timestamps, multicast market data, real ARM submission, CCP adapter.
