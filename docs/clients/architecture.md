# Architecture

SWANS is a trading venue. It matches orders, computes margin, determines settlement, and publishes data. It is never a counterparty, never holds funds, and never guarantees a trade.

## Launch clearing model: bilateral with prime brokers

```
Member OMS ──FIX──▶ SWANS gateway ──▶ pre-trade risk ──▶ matching engine ──▶ market data ──▶ members
                                                                │
                                                                ├──▶ PB adapter (trade notification, margin calls)
                                                                │        PB holds collateral, enforces margin
                                                                └──▶ drop copy to PBs
```

Each member has a relationship with a prime broker. The PB holds collateral in a segregated account under a CSA. SWANS acts as **calculation agent** — it computes IM and VM numbers and sends them to the PB. The PB collects margin from the member.

| Role | Who | What they do |
|---|---|---|
| Trading member | Funds, dealers, market makers | Trades on SWANS |
| Prime broker | Banks with PB capability | Holds collateral, enforces margin, settles |
| SWANS | The venue | Matches, computes margin, determines settlement |

## Future: CCP upgrade path

The PB adapter can be replaced by a CCP adapter without changing the core system. When a CCP is available:
- Trades go to CCP instead of PB for novation
- CCP holds default fund, provides guarantee
- Clearing members replace PBs in the chain

The margin engine, settlement service, and all other components remain identical.

## What SWANS does

- Operates the order book, matches trades, publishes market data
- Runs pre-trade risk on every order: PB-set limits and margin budget per account
- Sends trade notifications and margin instructions to the PB
- Determines settlement outcomes against documented sources
- Computes and publishes margin numbers: PB margin schedule, price and risk file
- Reports to FCA and runs surveillance

## What SWANS does not do

- Hold collateral (PB does)
- Set clearing margin (PB does, using SWANS numbers as calculation agent)
- Guarantee settlement (PB bears counterparty risk)

## Sequence numbers

Every state change carries a global, monotonically increasing `seq`. It appears in FIX execution reports (tag 20010), REST envelopes and WebSocket messages, so a client can order events across channels.

## Time

All timestamps are UTC. FIX `TransactTime` (60) and `SendingTime` (52) are microsecond precision; REST and WebSocket timestamps are nanoseconds since the Unix epoch, serialised as strings.

See the [end-to-end views](../diagrams.md) for the full system map and trade lifecycle.
