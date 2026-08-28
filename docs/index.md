# SWANS Event Exchange — Technical Architecture

SWANS is a trading venue for standardised, cash-settled event contracts (binary and measured) on corporate and macro catalysts. This document is the **working technical specification** — the team reviews and iterates on it before we build.

**Status:** draft architecture. Items marked **[confirm]** need a decision — see [Open decisions](internal/open-decisions.md).

## System at a glance

```
Members ──FIX──▶ Gateway ──▶ Engine (pre-trade + matching) ──▶ Market data ──▶ Members
                                        │
                                        ├──▶ Trades ──▶ PB adapter (margin calls, settlement)
                                        └──▶ Drop copy to PBs

Margin engine ──▶ PB: IM/VM schedule, price file
Settlement    ──▶ PB: final value, payout instruction
```

## Clearing model (launch)

**Bilateral with prime brokers.** No CCP at launch. Each member's PB holds collateral under a CSA. SWANS computes margin (IM, VM) and acts as calculation agent. The PB enforces margin calls and holds funds in segregated accounts. SWANS never holds collateral or guarantees trades.

CCP clearing is a future upgrade path — the architecture is designed so the PB adapter can be swapped for a CCP adapter without changing the core system.

## Contract economics

- Price in [0.005, 0.995], tick = 0.005, 200 price levels
- Payout: 100 units (GBP, USD or EUR), minimum notional $100
- Futures-style margining: no premium at trade, daily VM, IM from margin engine
- One order book per contract in yes-space

## Architecture sections

| Section | What it covers |
|---|---|
| [End-to-end views](diagrams.md) | Visual diagrams of the full system |
| [System architecture](internal/architecture.md) | Components, build/buy, deployment |
| [Data model](internal/data-model.md) | Core types and structures (C++) |
| [Messaging](internal/messaging.md) | Aeron, SBE, journals, recovery |
| [Services](internal/services/engine.md) | Each service in detail |
| [Margin engine](clients/margin.md) | Margin methodology and outputs |
| [Settlement](clients/settlement.md) | Determination process and sources |
| [Contracts](clients/contracts.md) | Contract specs, schemas, families |
| [Build plan](internal/build-plan.md) | Milestones and deliverables |
| [Open decisions](internal/open-decisions.md) | What we still need to decide |
