# SWANS Event Exchange — Technical Architecture

SWANS is a trading venue for standardised, cash-settled event contracts (binary and measured) on corporate and macro catalysts. This document is the **working technical specification** — the team reviews and iterates on it before we build.

**Status:** draft architecture. Items marked **[confirm]** need a decision — see [Open decisions](internal/open-decisions.md).

## System at a glance

```
Members ──FIX──▶ Gateway ──▶ Engine (pre-trade + matching) ──▶ Market data ──▶ Members
                                        │
                                        ├──▶ Trades ──▶ PB adapter (margin calls, settlement)
                                        ├──▶ Collateral service (full-collateral mode)
                                        └──▶ Drop copy to PBs

Margin engine    ──▶ PB: IM/VM schedule, price file
Collateral svc   ──▶ balance checks, locks, VM transfers, settlement payouts
Settlement       ──▶ PB or collateral service: final value, payout
```

## Clearing model (launch)

Two modes, configurable per account:

| Mode | How it works | Who holds cash |
|---|---|---|
| **PB-managed** | Bilateral with prime brokers. SWANS is calculation agent. PB holds collateral under CSA. | Prime broker |
| **Full collateral** | Members deposit cash to a CASS client money account. SWANS locks max loss per trade. No PB needed. | SWANS (via custodian) |

Both can run simultaneously. Full collateral is the day-one fallback if no PB is available. PB mode adds capital efficiency once PBs are onboarded.

CCP clearing is a future upgrade path — swap the PB adapter for a CCP adapter.

## Trading hours

**24/7 continuous trading.** No close. VM windows every 8 hours (00:00, 08:00, 16:00 UTC). Settlement determination happens when sources publish (business hours); the market stays open.

## Contract economics

- Price in [0.005, 0.995], tick = 0.005, 200 price levels
- Payout: 100 units (GBP, USD or EUR), minimum notional $100
- Futures-style margining (PB mode) or full collateral (self-managed mode)
- One order book per contract in yes-space

## Architecture sections

| Section | What it covers |
|---|---|
| [End-to-end views](diagrams.md) | Visual diagrams of the full system |
| [System architecture](internal/architecture.md) | Components, build/buy, deployment |
| [Data model](internal/data-model.md) | Core types and structures (C++) |
| [Messaging](internal/messaging.md) | Aeron, SBE, journals, recovery |
| [Services](internal/services/engine.md) | Each service in detail |
| [Collateral service](internal/services/collateral.md) | Balance management, locks, payouts |
| [Margin engine](clients/margin.md) | Margin methodology and outputs |
| [Settlement](clients/settlement.md) | Determination process and sources |
| [Contracts](clients/contracts.md) | Contract specs, schemas, families |
| [Build plan](internal/build-plan.md) | Milestones and deliverables |
| [Open decisions](internal/open-decisions.md) | What we still need to decide |
