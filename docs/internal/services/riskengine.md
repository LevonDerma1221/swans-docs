# Risk engine service

Wraps `libswansrisk`. Loads reference data; builds `MarketSnapshot` (venue mids, VM-window marks, hedge prices, option-implied event moves, factor levels); pulls portfolios; runs:

- **Intraday:** recalibration every 60 s or on quote moves beyond threshold; pushes per-account IM and fast bounds to the engine's pre-trade stage via `risk.budget` with a version.
- **VM window:** at each VM window (00:00, 08:00, 16:00 UTC), full risk report per account; inputs to the margin service.
- **Backtest mode:** replays catalyst outcomes and (synthetic, option-implied) price paths to produce margin reports. **The backtest is partly synthetic because no venue price history exists; every report states this.**
- **Model plug-in:** Mehdi's models implement `IPricingModel`, `ICalibrator`, `IMarginModel` (see [Core analytics library](../risk-library.md)); the service never contains model logic.

## Two-engine architecture

The risk engine service runs both engines on every official risk run:

| Engine | What it computes | Authority |
|---|---|---|
| **MPOR engine** | Close-out loss over `[t, t+h_a]` — VaR, ES, jump, add-ons | Authoritative for clearing IM |
| **Terminal engine** | Terminal-outcome tail loss through factor copula | Challenger, diagnostics, structural validation |

Both engines consume the same economic factor taxonomy, calibration snapshot and instrument-to-value mappings (common economic-state contract), but may carry different path-specific or terminal-specific state variables. Their divergence `D_a,χ` is recorded per account under aligned comparison configurations and fed into the model-risk add-on when above threshold. See [Core analytics library](../risk-library.md) for the consistency contract.

## Recalculation triggers

| Trigger | Action |
|---|---|
| Accepted/corrected/cancelled trade | Incremental account calculation immediately; full if threshold exceeded |
| Mark change above trigger | Recalculate affected accounts |
| Event hazard / status change | Recalculate all accounts holding the family |
| Collateral price/haircut change | Revalue affected pools and test deficit (full-collateral mode) |
| Model/policy parameter release | Controlled bulk rerun with old/new comparison |
| VM window | Full risk refresh, VM, reporting snapshot |

## Stress and challenger models

Each official account run includes the policy stress set: probability shocks, correlation increases, family near-comonotonicity, immediate YES/NO resolution, liquidity-depth collapse, mark fallback, oracle delay/dispute and member-specific close-out delay. Challenger models may use alternative copulas, factor decompositions or probability dynamics. Primary/challenger differences feed the model-risk overlay.

## Stress-resource consumption

For member/account `a` and stress scenario `s`:

```
S_a,s = [L_stress_a,s - IM_a - VM_collected_a - GF_a]⁺
ρ_a = max_s S_a,s / (R_total + ε)
```

Thresholds are governance variables. The engine supports Normal → Watch → Tightening → Stress states and records the scenario that determines `ρ_a`. Above thresholds: restrict risk-increasing orders, then require full collateral on new trades.

## Outputs

| Output | Consumer | Frequency |
|---|---|---|
| `{account: IM, fast_bound, version, ts}` | Engine pre-trade stage | Every run and on fill |
| Margin result with full attribution | Margin service | Every run |
| Two-engine divergence `D_a,χ` (aligned comparisons only) | Risk desk, model governance | Every official run |
| Stress-resource `ρ_a` and top scenarios | Risk desk | Every official run |
