# Architecture

SWANS is a trading venue. It matches orders, computes margin, determines settlement, and publishes data.

## Clearing model

Two modes, per account:

```
Full collateral (day-one default):
  Member deposits cash ──▶ SWANS locks max loss per trade ──▶ VM every 8 hours ──▶ settlement payout

PB-managed (optional upgrade):
  Member ──CSA──▶ Prime broker ◀── margin schedule ── SWANS (calculation agent)
```

Full collateral works without PB integration. PB-managed mode adds capital efficiency. Both can run simultaneously. See [Clearing](clearing.md).

## What SWANS does

- Operates the order book, matches trades, publishes market data
- Runs pre-trade risk on every order (balance check or PB limits)
- Computes margin and publishes margin schedules
- Determines settlement outcomes against documented sources
- Reports to FCA and runs surveillance

## Sequence numbers

Every state change carries a global, monotonically increasing `seq`. It appears in FIX execution reports, REST and WebSocket, so a client can order events across channels.

## Time

All timestamps are UTC nanoseconds since Unix epoch. Production clocks disciplined by PTP.

See the [end-to-end views](../diagrams.md) for the full system map and trade lifecycle.
