# SWANS Event Exchange — Technical Architecture

SWANS is a trading venue for standardised, cash-settled event contracts on corporate and macro catalysts. This document is the **working technical specification** — the team reviews and iterates on it before we build.

**Status:** draft architecture. Items marked **[confirm]** need a decision — see [Open decisions](internal/open-decisions.md).

## System at a glance

```
Members ──FIX──▶ Gateway ──▶ Engine (pre-trade + matching) ──▶ Market data ──▶ Members
                                        │
                                        ├──▶ Trades ──▶ Collateral service (lock max loss)
                                        └──▶ Settlement ──▶ Collateral service (distribute payout)

Margin engine    ──▶ risk analytics, price file (shadow mode at launch)
```

## Clearing model

**Full collateral at launch.** Members deposit cash. SWANS locks max loss at trade time. At settlement, payout is distributed. No variation margin, no margin calls, no adjustments in between. Simple.

How to offer margin (capital efficiency without locking full max loss) is an open design question. See [Clearing](clients/clearing.md) and [Open decisions](internal/open-decisions.md).

## Trading hours

**24/7 continuous trading.** No close. Settlement determination happens when sources publish.

## Contract economics

- Price in [0.005, 0.995], tick = 0.005, 200 price levels
- Payout: 100 units (GBP, USD or EUR), minimum notional $100
- Full collateral: max loss locked at trade time
- One order book per contract in yes-space

**Note:** the contract structure (payout function, amount, settlement kind, funding mechanics) is not final and may change. The architecture treats the payoff as a pluggable function — see [Open decisions #2](internal/open-decisions.md).

## Architecture sections

| Section | What it covers |
|---|---|
| [End-to-end views](diagrams.md) | Visual diagrams of the full system |
| [System architecture](internal/architecture.md) | Components, build/buy, deployment |
| [Contracts](clients/contracts.md) | Contract specs, schemas, families |
| [Clearing and collateral](clients/clearing.md) | Full collateral and future margin |
| [Margin](clients/margin.md) | Margin methodology (shadow mode at launch) |
| [Fees](clients/fees.md) | Fee structure and components |
| [Settlement](clients/settlement.md) | Determination process and sources |
| [Trading rules](clients/trading-rules.md) | Order types, limits, halts |
| [Build plan](internal/build-plan.md) | Milestones and deliverables |
| [Open decisions](internal/open-decisions.md) | What we still need to decide |
