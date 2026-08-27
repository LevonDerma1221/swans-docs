# System architecture (internal)

**Version 1.0.** Supersedes v0.1 and v0.2; incorporates Margin Framework v6, Fee Model v3, the contract engine, families, and the stress-test fixes (packages as CCP products, GCM-set margin budget, pending positions, STP in the engine, stale-budget fallback, synthetic backtest disclosure, volatility halts, RTS 24 fields).

## Build / buy

| Component | Decision | Why |
|---|---|---|
| Reference data, contract engine, trades, positions, marks, risk engine, margin and fees, settlement, pre-trade risk | **Build** | The product; hard to outsource; what the CCP, GCMs and clients evaluate |
| Matching engine, FIX gateway, market data publisher | Build in phase 1 (simplified); evaluate Eqlipse / commercial FIX engine for production | Binary book is small; in-house lets us test end-to-end now; conformance is the real cost |
| Surveillance | Buy (SMARTS or equivalent); we supply feeds | FCA expects a recognised system |
| Transaction reporting submission | Buy (ARM); we build the extract | Plumbing |
| Trading UI | Build later, thin, on the REST/WS API | Members use their own OMS |

Full diagrams: [End-to-end views](../diagrams.md).

## Services

```
 Members ──FIX──▶ Gateway ──▶ [Engine shard: PreTrade → Matching → STP] ──┬──▶ MarketData ──▶ Members
                                          ▲                                 ├──▶ Trades ──▶ CCP adapter
                                          │ margin budgets                  │        │
                                     MarginService ◀── RiskEngine ◀── Positions ◀────┘
                                          │               ▲
              GCM files, CCP params ◀─────┘        ReferenceData ──▶ all
                                                          ▲
                                                  Settlement ──▶ Trades / Positions / CCP
                                                  Reporting & surveillance extracts ◀── all
                                                  Clearer API ──▶ PreTrade limits
                                                  REST/WS API ──▶ read models
```

Pre-trade risk runs as a **stage inside the engine shard process** (no IPC hop), as a separate module. STP submission is initiated by the trades service on `TradeExecuted`.

## Deployment

- **Production:** LD4 (Equinix Slough). Two racks, dedicated switches, member cross-connects. Engine and gateway hosts pinned, isolated cores, kernel bypass optional later.
- **DR:** second site, warm standby, journal shipping; RTO 2 hours **[confirm RTS 7]**.
- **Prototype:** one VM per service or Docker Compose on a single host.

## Time

`int64_t` nanoseconds since Unix epoch, UTC, everywhere. Production clocks disciplined by PTP; RTS 25 traceability to UTC (≤ 100 µs or ≤ 1 ms depending on classification **[confirm]**). Prototype uses NTP.

## Contract economics (locked)

- Price in [0.005, 0.995], ticks of 0.005, integer `price_ticks ∈ [1, 199]`; payout 100 units; currencies GBP/USD/EUR, one per contract.
- **Futures-style margining.** No premium at trade time; daily VM; IM from the margin service. **The first question to the CCP is whether it will clear binaries futures-style; several CCPs only clear binaries premium-style.**
- One book per contract in yes-space.
- **Packages are separate products** at the CCP with their own bounded payoff. Leg decomposition exists only inside SWANS's risk and position views.

## Non-goals for phase 1

Auctions, iceberg orders, HA failover, hardware timestamps, multicast market data, real ARM submission, OTF discretion workflow (interface reserved).
