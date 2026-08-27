# Margin service (Margin Framework v6 in the partner-CCP structure)

The V6 methodology was written for SWANS as clearing house. In the partner-CCP structure, the **model** is unchanged and its **outputs** are redirected: the clearing operations (default waterfall, guaranty fund, default engine, recovery tools) belong to the CCP; the venue keeps computation, budgets, files and the bilateral calculation-agent role.

## Components (from V6 §22, mapped)

| V6 component | Where it lives now | Notes |
|---|---|---|
| Fair probability filter | **Venue** (settlement/marks service) | Official filtered mark for VM, IM, price file; family projection |
| Probability model, resolution-intensity model, marked jump simulator, Monte Carlo engine | **Venue** (`libswansrisk`) | Mehdi's models behind `IPricingModel`, `ICalibrator`, `IMarginModel` |
| Margin calculator | **Venue** (this service) | Produces IM, attribution, budgets, schedules, CCP parameter proposal |
| VM engine | **Venue computes; CCP and GCM collect** | Venue publishes VM per account on filtered marks; CCP collects on its settlement price if it sets one **[decision 3]** |
| Collateral engine | **CCP and GCM** | Venue never holds collateral |
| Default engine | **CCP and GCM** | Venue suspends trading rights on notice |
| Stress-resource control | **Venue**, reinterpreted | Account stress loss vs clearing-member limit → restrict risk-increasing orders |
| Model governance | **Venue** | Backtests, sensitivity, validation, challenger models, audit trail; shared with CCP |
| Participant simulator | **Venue** (Product A) | `POST /margin/simulate` |

## Computation

Per account, per run (daily 17:00; intraday on triggers: mark move beyond threshold, event-ramp state change, concentration breach, VM-late signal from GCM):

1. Load positions (net + pending), families, filtered marks `X_fair`, hedge positions (for reporting only).
2. `L_gross` by family over admissible states.
3. MPOR `h_i = max_k (h₀(class_k) + h_liq + h_conc + h_ops + h_oracle)`.
4. Monte Carlo over `[t, t+h_i]`: latent-factor diffusion with idiosyncratic weights as specified in the loading file (assert `Σ B² + w² = 1` at load), marked jumps with policy hazards `λ_policy = m_cat · r(t) / max(θ, 1)`, correlated resolution clusters, close-out slippage, VM timing.
5. `VaR_99`, `ES_97.5`, sign-sensitive `IM_jump`, floor.
6. Add-ons: liquidity (phase 1 policy), concentration (`ζ·L_side·((c−c̄)⁺/c̄)²`), oracle, model risk, event ramp (`λ_event = max(λ_cal, λ_haz)`), APC.
7. `IM = min(L_gross, IM_core + Σ A)`; then `IM_event` blend.
8. Market-maker relief flags evaluated against LP criteria; withdrawn automatically on breach.
9. Outputs with attribution.

## Outputs

| Output | Consumer | Frequency |
|---|---|---|
| `{account: IM, fast_bound, version, ts}` | Engine pre-trade stage | Every run and on fill |
| Client margin schedule with add-on attribution | GCMs | Daily + intraday |
| VM per account on filtered marks | GCMs, members | Daily + intraday |
| CCP parameter proposal: per product IM long/short, scenario P&L arrays, family definitions, offset groups and stage, hazards, liquidity tiers, model version | CCP | Daily |
| Price and risk file: filtered marks, IM, sensitivities, event-ramp state | Prime brokers, members | Daily |
| CSA IM per account pair | Bilateral bridge | Daily |
| Backtest (Kupiec, Christoffersen), sensitivity, trigger report; synthetic-price disclosure | CCP, GCMs, FCA | Monthly |

## Budget
`budget_available = ClearerLimits.max_swans_im − IM_current − reserved_open_orders`, where `reserved_open_orders` is the sum of incremental IM reserved at order acceptance (released on cancel/expiry, converted on fill). Fast bound for pre-trade: `qty × payout × max(p, 1−p)` (a strict upper bound on both structural loss and IM).

## Fee engine (v3)
Same service, same per-fill inputs. Computes `f_j = max(f_min, B + R·(1−ρU) − D − M)` with state from: book liquidity score `L` (depth, spread, cancellation intensity, EWMA), time to event `τ`, account concentration by family `c_k`, pre-fill position for `U` (netted across related contracts and accounts), maker quality score `Q_maker` (uptime, spread, displayed size, realised adverse selection). Emits `fee_components` per fill to trades, drop copy and EOD fee file. Anti-gaming checks run on aggregate account activity per window.

## Governance
Parameter changes and offset-stage changes are versioned reference data approved by the risk committee, notified to the CCP and GCMs with notice periods. Model changes go through validation with challenger comparison before production.
