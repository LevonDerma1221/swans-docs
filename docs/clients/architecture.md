# Architecture

SWANS is a trading venue. It matches orders, determines settlement, publishes data, and manages collateral.

## Clearing model

**Full collateral.** Members deposit cash. SWANS locks max loss per trade. At settlement, payout is distributed. No VM, no margin calls, no adjustments between trade and settlement.

```
Member deposits cash ──▶ SWANS locks max loss per trade ──▶ settlement payout
```

How to offer margin (capital efficiency) is an open design question — see [Clearing](clearing.md).

## What SWANS does

- Operates the order book, matches trades, publishes market data
- Runs pre-trade risk on every order (balance check)
- Holds collateral via regulated custodian, locks max loss, distributes payouts
- Determines settlement outcomes against documented sources
- Computes risk analytics and margin numbers (shadow mode at launch, production when margin is offered)
- Reports to FCA and runs surveillance

## Sequence numbers

Every state change carries a global, monotonically increasing `seq`. It appears in FIX execution reports, REST and WebSocket, so a client can order events across channels.

## Time

All timestamps are UTC nanoseconds since Unix epoch. Production clocks disciplined by PTP.

See the [end-to-end views](../diagrams.md) for the full system map and trade lifecycle.
