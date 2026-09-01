# Margin service

The V6 methodology computes margin for all account types. In PB-managed mode, SWANS is calculation agent under the CSA. In full-collateral mode, margin drives lock adjustments and VM transfers. The **model** is the same regardless of clearing mode.

## Components

| V6 component | Where it lives | Notes |
|---|---|---|
| Fair probability filter | Marks service | Official filtered mark for VM, IM, price file; family projection |
| Probability model, resolution-intensity model, marked jump simulator, Monte Carlo engine | `libswansrisk` | Mehdi's models behind `IPricingModel`, `ICalibrator`, `IMarginModel` |
| Margin calculator | This service | Produces IM, attribution, budgets, schedules |
| VM engine | This service computes; PB collects or collateral service transfers | VM per account on filtered marks |
| Collateral engine | PB (PB-managed) or collateral service (full-collateral) | |
| Stress-resource control | This service | Account stress loss vs limits → restrict risk-increasing orders |
| Model governance | This service + governance | Backtests, sensitivity, validation, challenger models, audit trail |
| Participant simulator | This service | `POST /margin/simulate` |

## Computation

Per account, per run (at each VM window 00:00/08:00/16:00 UTC; intraday on triggers: mark move beyond threshold, event-ramp state change, concentration breach, VM-late signal):

1. Load positions (net + pending), families, filtered marks `X_fair`, hedge positions (for reporting only).
2. `L_gross` by family over admissible states.
3. MPOR `h_i = max_k (h₀(class_k) + h_liq + h_conc + h_ops + h_oracle)`.
4. Monte Carlo over `[t, t+h_i]`: latent-factor diffusion with idiosyncratic weights as specified in the loading file (assert `Σ B² + w² = 1` at load), marked jumps with policy hazards `λ_policy = m_cat · r(t) / max(θ, 1)`, correlated resolution clusters, close-out slippage, VM timing.
5. `VaR_99`, `ES_97.5`, sign-sensitive `IM_jump`, floor.
6. Add-ons: liquidity (phase 1 policy), concentration (`ζ·L_side·((c−c̄)⁺/c̄)²`), oracle, model risk, event ramp (`λ_event = max(λ_cal, λ_haz)`), APC.
7. `IM = min(L_gross, IM_core + Σ A)`; then `IM_event` blend.
8. Market-maker relief flags evaluated against LP criteria; withdrawn automatically on breach.
9. Two-engine comparison: terminal engine runs in parallel; divergence `D_a,χ` recorded only under aligned comparison configurations.
10. Outputs with full attribution.

## IM formula (from V6, formalized)

```
IM_core = max( VaR_MPOR_99%, κ·ES_MPOR_97.5%, IM_jump, IM_floor )

A = A_liq + A_conc + A_oracle + A_model + A_event + A_APC

IM_hybrid = min( L_gross, IM_core + A )

IM_event = (1 - λ_event) · IM_hybrid + λ_event · L_gross
```

Every component is persisted and reported separately with its policy/version source.

### Sign-sensitive jump-to-resolution

```
L¹_ak = [-Q_ak · M_k · (1 - X_k)]⁺    (loss from YES resolution)
L⁰_ak = [Q_ak · M_k · X_k]⁺           (loss from NO resolution)
```

Resolution probabilities over the MPOR come from the marked-intensity model or conservative policy hazards. Correlated family resolution is simulated jointly.

### Concentration add-on

```
C_ak = |Q_ak| / (OI_k + ε)
A_conc_ak = ζ_k · L_side_ak · ((C_ak - c̄_k)⁺ / c̄_k)²
A_conc_a = Σ_k A_conc_ak
```

### Anti-procyclicality (APC)

Pre-funds part of the gap to a stressed/lookback margin state to reduce procyclical call cliffs. Each amount carries reason code, owner, effective period and approval record.

## Variation margin

At each VM window (00:00, 08:00, 16:00 UTC):

```
ΔV_a = V_a(t₁) - V_a(t₀)
VM_due = [-ΔV_a]⁺
VM_receivable = [ΔV_a]⁺
```

## Creditable resources, requirement and margin call (single-ledger)

The collateral and settlement services maintain a **single authoritative resource ledger**. Settled VM receivables and other finalized cash movements update the creditable balance; they are not added again as a separate resource term (no double counting). Unsettled receivables receive zero credit unless an explicit, legally approved provisional-credit policy applies.

```
C_credit = haircut-adjusted value of all resources creditable to the margin account
           (confirmed collateral + settled VM + finalized cash, each entered exactly once)
D_other  = accrued fees/debits not yet reflected in C_credit

E_a = C_credit - D_other                      (available account equity)
R_a = IM_event + VM_due + Buffer               (required resources)
MC_a = [R_a - E_a]⁺                           (margin call amount)
Excess_a = [E_a - R_a]⁺                       (withdrawable subject to policy)
```

**Ledger invariant.** Every cash balance, collateral lot and finalized VM movement has one resource-ledger identity and can contribute to `C_credit` at most once. A completed withdrawal or fee posting that has already reduced the balance is not subtracted again in the margin formula.

A positive `MC_a` creates or updates a margin call (see margin call lifecycle below).

## Margin call lifecycle

| State | Meaning |
|---|---|
| DRAFT | Calculation produced a candidate call; awaiting authorized release |
| ISSUED | Firm obligation delivered to member; due-time clock active |
| ACKNOWLEDGED | Member receipt confirmed; due time remains effective |
| PARTIALLY_SATISFIED | Confirmed cash/collateral covers part of amount |
| SATISFIED | Full obligation covered |
| DISPUTED | Member raised dispute; follows dispute policy |
| FAILED | Payment/custody attempt failed |
| DEFAULT_ESCALATION | Deadline or governance trigger reached default-management state |
| CANCELLED | Authorized replacement/reversal |

Each transition records actor/system, timestamp, reason code, evidence and balance delta.

## Gross customer margin

Customer-origin IM is calculated gross: the sum of individual-customer IM requirements, with no inter-customer netting. Each separate-account customer is represented as its own individual-customer margin unit. House accounts may be margined on a net basis where permitted. The position ledger retains the customer identifier and gross EOD positions required to reproduce the calculation.

## Netting layers (correctly named per CFTC terminology)

| Layer | Mechanism | Status |
|---|---|---|
| 1. Same-contract netting | Aggregate trades into `Q_ak` within one legal margin unit. For customer-origin calculations, aggregation is performed per individual customer before gross summation. | Deterministic bookkeeping |
| 2. Structural family netting | Evaluate mutually exclusive, nested or linked contracts over admissible terminal states `Y_g` | Exact payoff logic |
| 3. Spread/portfolio margining (§39.13(g)(4)) | Jointly simulate related positions in approved portfolio-margin group `G_r`; approved diversification in portfolio loss distribution | Requires conceptual basis + statistical evidence |

The conceptual basis for a portfolio-margin offset may include complement/substitute relationships, input relationships, shared inputs or common external drivers — it is not limited to a named global factor. Each `portfolio_margin_policy` record MUST contain: group ID, covered products/families, conceptual basis, dependency-model version, validation evidence/date, offset caps, effective dates and approval owner.

**Inter-DCO cross-margining** (§39.13(i)) is a separate architecture and regulatory workstream, represented by a distinct `cross_margin_program` entity. It MUST NOT be conflated in code or reporting with the internal spread/portfolio-margin policy above.

## Outputs

| Output | Consumer | Frequency |
|---|---|---|
| `{account: IM, fast_bound, version, ts}` | Engine pre-trade stage | Every run and on fill |
| Margin schedule with add-on attribution | PBs (PB-managed accounts) | VM window + intraday |
| VM per account on filtered marks | PBs, members, collateral service | VM window + intraday |
| Price and risk file: filtered marks, IM, sensitivities, event-ramp state | PBs, members | VM window |
| Backtest (Kupiec, Christoffersen), sensitivity, trigger report | PBs, FCA | Monthly |
| Collateral lock adjustments | Collateral service (full-collateral accounts) | VM window |
| Two-engine divergence report | Risk desk, model governance | Every official run |

## Budget

`budget_available = ClearerLimits.max_swans_im − IM_current − reserved_open_orders`, where `reserved_open_orders` is the sum of incremental IM reserved at order acceptance (released on cancel/expiry, converted on fill). Fast bound for pre-trade: `qty × payout × max(p, 1−p)` (a strict upper bound on both structural loss and IM).

## Fee engine (v3)

Same service, same per-fill inputs. Computes `f_j = max(f_min, B + R·(1−ρU) − D − M)` with state from: book liquidity score `L` (depth, spread, cancellation intensity, EWMA), time to event `τ`, account concentration by family `c_k`, pre-fill position for `U` (netted across related contracts and accounts), maker quality score `Q_maker` (uptime, spread, displayed size, realised adverse selection). Emits `fee_components` per fill to trades, drop copy and fee file. Anti-gaming checks run on aggregate account activity per window.

## Governance

Parameter changes and offset-stage changes are versioned reference data approved by the risk committee, notified to PBs and members with notice periods. Model changes go through validation with challenger comparison before production. Every model has a semantic version, owner, approved scope, validation status, effective date, code commit/build hash and rollback version. See [Model governance](../governance.md).
