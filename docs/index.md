# SWANS Event Exchange — Technical Architecture

SWANS is a trading venue for standardised, cash-settled event contracts on corporate and macro catalysts. This document is the **working technical specification** — the team reviews and iterates on it before we build.

**Status:** draft architecture. Items marked **[confirm]** need a decision — see [Open decisions](internal/open-decisions.md).

## System at a glance

```
Members ──FIX──▶ Gateway ──▶ Engine (pre-trade + matching) ──▶ Market data ──▶ Members
                                        │
                                        ├──▶ Trades ──▶ Collateral service (full-collateral mode)
                                        ├──▶ Trades ──▶ PB adapter (optional, PB-managed mode)
                                        └──▶ Drop copy

Margin engine    ──▶ balance checks / locks (full-collateral)
                 ──▶ PB margin schedule (PB-managed)
Settlement       ──▶ collateral payout or PB settlement value
```

## Clearing model

Two modes, configurable per account:

| Mode | How it works | Who holds cash |
|---|---|---|
| **Full collateral** | Members deposit cash. SWANS locks max loss per trade. VM every 8 hours. No PB needed. | SWANS (via custodian) |
| **PB-managed** | Bilateral with prime brokers. SWANS is calculation agent. PB holds collateral under CSA. | Prime broker |

**Full collateral is the day-one default.** PB mode adds capital efficiency once PBs are onboarded. CCP clearing is a future upgrade path.

## Trading hours

**24/7 continuous trading.** No close. VM windows every 8 hours (00:00, 08:00, 16:00 UTC). Settlement determination happens when sources publish.

## Contract economics

- Price in [0.005, 0.995], tick = 0.005, 200 price levels
- Payout: 100 units (GBP, USD or EUR), minimum notional $100
- One order book per contract in yes-space

**Note:** the contract structure (payout function, amount, settlement kind, funding mechanics) is not final and may change. The architecture treats the payoff as a pluggable function — see [Open decisions #2](internal/open-decisions.md).

## Architecture sections

| Section | What it covers |
|---|---|
| [End-to-end views](diagrams.md) | Visual diagrams of the full system |
| [System architecture](internal/architecture.md) | Components, build/buy, deployment |
| [Contracts](clients/contracts.md) | Contract specs, schemas, families |
| [Clearing and collateral](clients/clearing.md) | Full-collateral and PB-managed modes |
| [Margin](clients/margin.md) | Margin methodology and outputs |
| [Fees](clients/fees.md) | Fee structure and components |
| [Settlement](clients/settlement.md) | Determination process and sources |
| [Trading rules](clients/trading-rules.md) | Order types, limits, halts |
| [Build plan](internal/build-plan.md) | Milestones and deliverables |
| [Open decisions](internal/open-decisions.md) | What we still need to decide |
