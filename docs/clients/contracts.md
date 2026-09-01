# Contracts

## Definition

A SWANS event contract is a cash-settled derivative whose settlement value depends on the outcome of a defined catalyst, determined against a named official source. Two settlement kinds:

| Kind | Settlement value |
|---|---|
| Binary | 1 if the settlement question is answered "yes", 0 otherwise |
| Measured | A value in [0, 1] mapped from a published figure (e.g. a reported KPI within a defined range) |

Every contract has a **payout** of 100 units of its settlement currency (GBP, USD or EUR). A binary settling at 1 pays 100; at 0 pays 0. A measured contract settling at 0.62 pays 62.

## Price convention

Prices are quoted between 0 and 1 in ticks of 0.005: valid prices are 0.005, 0.010, …, 0.995. A price of 0.70 on a binary means the market implies roughly a 70% chance of "yes".

There is one order book per contract, in "yes" space. Buying at p is economically the same as selling "no" at 1 − p. If you are long 5 contracts and sell 8, you are short 3.

## Positions and exposure

Positive quantity is long; negative is short. Maximum loss per contract:

- Long at p: p × 100 (contract settles at 0)
- Short at p: (1 − p) × 100 (contract settles at 1)

**PB-managed accounts:** margined futures-style — no premium at trade time; VM at each 8-hour window; IM from the margin engine. **Full-collateral accounts:** max loss locked at trade time; VM transfers at each window. See [Margin](margin.md).

**Note:** the contract payout structure (binary vs measured, payout amount, settlement kind) may evolve. The architecture is designed to support alternative payout functions beyond binary yes/no — the `ContractSpec` and margin engine treat the payoff as a pluggable function of the settlement value.

## Symbols and identifiers

Symbol format: `ISSUER-SCHEMA-CATALYST-EXPIRY`, e.g. `PFE-FDA-NDA2201-26NOV`. Every contract also has an ISIN, a CFI code **[confirm]** and a FISN. Reference data exposes all of them; FIX accepts symbol (55) or ISIN (48 with 22=4).

## Schemas

Contracts are generated from ten schema families (S-01 to S-10) covering regulatory approvals, earnings and KPI thresholds, litigation outcomes, M&A completion, index inclusion, macro prints, central bank decisions and others. Each schema defines the question template, the settlement source hierarchy, the fallback rules and the expiry logic. The schema policy document is the authority on how a contract in that family settles.

## Contract specification fields

| Field | Description |
|---|---|
| `symbol`, `isin`, `cfi`, `fisn` | Identifiers |
| `schema` | S-01 … S-10 |
| `settlement_kind` | `binary` or `measured` |
| `currency`, `payout` | Settlement currency; payout is 100 |
| `tick`, `min_qty`, `max_order_qty`, `position_limit` | Trading parameters |
| `issuer_lei`, `issuer_ticker` | For single-name contracts |
| `catalyst_id`, `question` | Catalogue key and the exact settlement question |
| `source` | Settlement source code, canonical URI, tier |
| `listing_time`, `last_trading_time` | Trading window; trading stops at or before the event |
| `expected_resolution`, `settlement_deadline`, `fallback_rule` | Settlement timing and fallback |
| `package_legs`, `package_ratios` | Populated for packages |
| `hedge_map` | Listed instruments (single-stock options, equity, futures) the contract is designed to hedge against, with the CCP they clear at |
| `state` | `pending`, `listed`, `trading`, `halted`, `expired`, `settled`, `cancelled` |
| `transparency_mode` | Pre-trade transparency treatment under the waiver regime **[confirm]** |

## Families

Contracts belong to families (`standalone`, `nested`, `mutually_exclusive`, `cluster`) defined by their schema. Family membership is reference data and drives mark projection, admissible-state margin and settlement consistency. See [Schemas](schemas.md).

## Packages

A spread, ladder or "yes on A / no on B" combination is listed as its own contract with its own book, symbol and ISIN. Its settlement value is a defined function of its legs' outcomes and its payout is bounded. Packages are margined as single products with their own bounded payoff, which is what makes their margin lower than the sum of the legs. SWANS decomposes packages into legs in the position and risk feeds for your own risk view.

## Lifecycle

`listed → trading → expired (last trading time) → settled`, with `halted` possible during trading (volatility halt, operator halt, source event). A contract may be `cancelled` before listing, or voided under the rulebook if the catalyst becomes undeterminable before any trade.
