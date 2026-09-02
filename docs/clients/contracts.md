# Contracts

## Definition

A SWANS event contract is a cash-settled derivative whose settlement value depends on the outcome of a defined catalyst, determined against a named official source.

**The contract structure is not final.** The payout function (binary, measured, or other), payout amount, settlement kind, and funding mechanics may evolve as we finalise the product design. The architecture treats the payoff as a pluggable function — `ContractSpec` and the margin engine are designed to support alternative structures. See [Open decisions #2](../internal/open-decisions.md).

## Current design (subject to change)

| Kind | Settlement value |
|---|---|
| Binary | 1 if the settlement question is answered "yes", 0 otherwise |
| Measured | A value in [0, 1] mapped from a published figure |

Contract size: 100. Prices quoted between 0.01 and 0.99 in ticks of 0.01. Currencies: GBP, USD or EUR.

One order book per contract, in "yes" space. Buying at p is economically the same as selling "no" at 1 - p.

## Positions and exposure

Positive quantity is long; negative is short. Maximum loss per contract:

- Long at p: p × 100 (contract settles at 0)
- Short at p: (1 − p) × 100 (contract settles at 1)

Max loss is locked at trade time and released at settlement. See [Clearing](clearing.md).

## Symbols and identifiers

Symbol format: `ISSUER-SCHEMA-CATALYST-EXPIRY`, e.g. `PFE-FDA-NDA2201-26NOV`. Every contract also has an ISIN. Reference data exposes all identifiers.

## Schemas

Contracts are generated from schema families (S-01 to S-10) covering regulatory approvals, earnings and KPI thresholds, litigation outcomes, M&A completion, index inclusion, macro prints, central bank decisions and others. Each schema defines the question template, settlement source hierarchy, fallback rules and expiry logic.

## Families

Contracts belong to families (`standalone`, `nested`, `mutually_exclusive`, `cluster`) defined by their schema. Family membership drives mark projection, admissible-state margin and settlement consistency.

## Packages

A spread, ladder or combination is listed as its own contract with its own book. Its settlement value is a defined function of its legs' outcomes and its payout is bounded.

## Lifecycle

`listed -> trading -> expired -> settled`, with `halted` possible during trading. A contract may be `cancelled` before listing, or voided if the catalyst becomes undeterminable.
