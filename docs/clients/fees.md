# Fees

Fees are charged per fill, per side, in basis points of face notional (contracts × payout), and are attributed by component so that every participant can see what they paid for. The fee is a market-design control, not a flat toll: liquidity-creating and risk-reducing trades pay less; certainty-amplifying, fragile-timing and concentrating trades pay more. Parameters are published in reference data and changed only through governance with notice.

## Structure (v3)

For fill *j* at price *p* with side *s* (+1 buy, −1 sell), define the side-aware certainty score `z = s·(2p − 1)`: positive when the trade pushes the market toward a boundary, negative when it leans against consensus.

```
f_j  = max( f_min,  B_j + R_j·(1 − ρ·U_j) − D_j − M_j )      in bps
R_j  = CP_j + EW_j + LC_j + CC_j
Fee  = N_j / 10,000 × f_j
```

| Component | Definition | What it prices |
|---|---|---|
| Base `B` | `b₀ + b₁·φ(z)`, small side-aware tilt | Venue, clearing connectivity, surveillance, reporting |
| Certainty pressure `CP` | `α_cp · h(λ_cp·z)`, smoothstep, zero for z ≤ 0 | Pushing an already directional contract toward 0 or 1 |
| Event window `EW` | `α_ew · exp(−τ/T_ew) · (1 + η_ew·h(λ_cp·z))` | Trading near resolution, more so when also certainty-pushing |
| Liquidity stress `LC` | `α_lc · (1 − L)`, L = book liquidity quality | Consuming thin or deteriorating liquidity |
| Concentration `CC` | `Σ_k ζ_k · ((c_k − c̄_k)⁺ / c̄_k)²` | Clustered exposure by family |
| Contrarian discount `D` | `α_d · h(−λ_d·z)`, capped | Absorbing unpopular risk |
| Maker rebate `M` | `α_m · Q_maker`, durable-quote quality score | Displayed, durable, useful liquidity |
| Unwind relief `ρ·U` | `U` = fraction of the fill that reduces existing exposure, after netting across related contracts and accounts | Applied to the risk block only |
| Floor `f_min` | | Prevents negative or gamed fees |

## Illustrative values (research calibration, not production)

Average fee by archetype in the fee lab: passive market maker ≈ 12 bps, mean-reversion ≈ 30 bps, momentum ≈ 36 bps, noise trader ≈ 33 bps, event-window chaser ≈ 45 bps. The ordering is the design intent.

## Attribution

Every fill carries its component breakdown in the drop copy, the fill record (`fee_components`) and the end-of-day fee file: base, certainty pressure, event window, liquidity stress, concentration, contrarian discount, maker rebate, unwind relief, floor applied.

## Anti-gaming rules (system requirements)

Order splitting does not reduce the fee: components are evaluated on the account's aggregate activity in the fill's window. Maker rebates require durable presence measured against quote uptime, spread, displayed size and realised adverse selection, not fleeting quotes. Unwind relief is computed after netting across related contracts, accounts and affiliates. The contrarian discount is capped and protected by the floor. Nested, mutually exclusive and correlated contracts are monitored as families. Related legal entities cannot bypass family-level concentration.

## Regulatory design note

Fee structures on a UK trading venue must be transparent, fair and non-discriminatory and must not create incentives to place, modify or cancel orders in a way that contributes to disorderly trading. The v3 components are published formulas applied identically to all members; the venue's compliance review of this structure against those requirements is part of the authorisation application **[counsel]**.

## Liquidity provider scheme

Designated liquidity providers with quoting obligations (presence, spread, size across a contract set) receive the maker rebate at the enhanced tier and the market-maker margin treatment described in [Margin](margin.md). Obligations and tiers are published in reference data.

## Other fees

Membership (annual), connectivity (per session, per cross-connect), market data (per subscriber, per level), erroneous-trade review (per request). Clearing fees are set by the CCP and your clearing member.
