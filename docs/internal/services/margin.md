# Margin service

**At launch, the margin engine runs in shadow mode** — computing risk analytics, collecting data, and validating models, but not driving collateral decisions. Full collateral locks max loss per trade with no VM or margin calls.

**When margin is offered** (via PB integration, CCP, or another mechanism), the margin engine becomes production: it drives IM, VM, margin calls, and collateral locks.

## Components

| V6 component | Where it lives |
|---|---|
| Fair probability filter | Marks service |
| Probability, resolution-intensity, Monte Carlo engine | `libswansrisk` (Mehdi's models) |
| Margin calculator | This service |
| VM engine | This service (future: when margin is offered) |
| Stress-resource control | This service |
| Model governance | This service + governance |
| Participant simulator | This service (`POST /margin/simulate`) |

## Computation

Per account, per run:

1. Load positions, families, filtered marks, hedge positions
2. Structural max loss `L_gross` by family over admissible states
3. MPOR horizon with add-ons: `h = h_base + h_liq + h_conc + h_ops + h_oracle`
4. Monte Carlo over MPOR: latent-factor diffusion, marked jumps, correlated resolution, close-out slippage
5. Core: `IM_core = max(VaR_99%, ES_97.5%, IM_jump, floor)`
6. Add-ons: liquidity, concentration, oracle, model risk, event ramp, APC
7. `IM = min(L_gross, IM_core + add-ons)`, then event-ramp blend
8. Market-maker relief flags evaluated; withdrawn automatically on breach
9. Two-engine comparison: terminal engine runs in parallel; divergence recorded under aligned configurations
10. Outputs with full attribution

See [Margin](../../clients/margin.md) for the full methodology description and formulas.

## Creditable resources and margin call (future: when margin is offered)

Single authoritative resource ledger — no double counting. Settled VM, confirmed collateral and finalized cash each entered exactly once.

```
E_a = C_credit - D_other          (available equity)
R_a = IM + VM_due + Buffer        (required resources)
MC_a = [R_a - E_a]+               (margin call)
```

## Margin call lifecycle (future: when margin is offered)

| State | Meaning |
|---|---|
| DRAFT | Candidate call; awaiting release |
| ISSUED | Firm obligation; due-time active |
| ACKNOWLEDGED | Member receipt confirmed |
| PARTIALLY_SATISFIED | Part of amount covered |
| SATISFIED | Full obligation covered |
| DISPUTED | Member raised dispute |
| FAILED | Payment/custody attempt failed |
| DEFAULT_ESCALATION | Deadline reached default-management state |
| CANCELLED | Authorized replacement/reversal |

## Gross customer margin

Customer-origin IM is gross: sum of individual-customer IM with no inter-customer netting. House accounts may be margined net where permitted.

## Netting layers

| Layer | Mechanism | Status |
|---|---|---|
| 1. Same-contract | Aggregate trades into net position per margin unit | Deterministic |
| 2. Structural family | Evaluate over admissible terminal states | Exact payoff logic |
| 3. Spread/portfolio margining | Jointly simulate related positions in approved groups | Requires conceptual basis + evidence |

Inter-DCO cross-margining is a separate architecture and regulatory workstream.

## Fee engine (v3)

Per-fill: `f = max(f_min, B + R(1-rhoU) - D - M)` with components from book liquidity, time to event, concentration, unwind fraction, and maker quality. Anti-gaming checks run on aggregate account activity per window.

## Budget (future: when margin is offered)

`budget_available = max_swans_im - IM_current - reserved_open_orders`. Fast bound for pre-trade: `qty * payout * max(p, 1-p)`.

## Outputs

| Output | Consumer | Frequency |
|---|---|---|
| IM + fast bound | Shadow mode analytics (launch); pre-trade (when margin offered) | Every run |
| Margin schedule with attribution | Future: PBs or margin counterparty | On trigger |
| Price and risk file | Members, future counterparties | Periodic |
| Backtest and sensitivity report | FCA | Monthly |
| Two-engine divergence | Risk desk | Every official run |
