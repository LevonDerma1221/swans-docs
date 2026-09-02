# Risk engine service

Wraps `libswansrisk`. Loads reference data; builds market snapshots (venue mids, VM-window marks, hedge prices, factor levels); pulls portfolios; runs both engines on every official risk run.

## Two-engine architecture

| Engine | What it computes | Authority |
|---|---|---|
| **MPOR engine** | Close-out loss over `[t, t+h]` — VaR, ES, jump, add-ons | Authoritative for clearing IM |
| **Terminal engine** | Terminal-outcome tail loss through factor copula | Challenger, diagnostics, stress |

Both consume the same factor taxonomy and calibration snapshot (common economic-state contract). Divergence `D_a` is recorded under aligned comparison and feeds the model-risk add-on when above threshold.

## Run modes

- **Intraday:** recalibration every 60s or on quote moves beyond threshold; pushes IM and fast bounds to the engine pre-trade stage
- **VM window:** full risk report per account at 00:00, 08:00, 16:00 UTC; inputs to the margin service
- **Backtest:** replays outcomes and synthetic price paths. Every report states that backtests are partly synthetic (no venue price history)

## Recalculation triggers

| Trigger | Action |
|---|---|
| Trade accepted/corrected/cancelled | Incremental; full if threshold exceeded |
| Mark change above trigger | Recalculate affected accounts |
| Event hazard/status change | Recalculate all accounts holding the family |
| Collateral price/haircut change | Revalue affected pools |
| Model/policy parameter release | Bulk rerun with old/new comparison |
| VM window | Full risk refresh |

## Stress

Each account run includes the policy stress set: probability shocks, correlation increases, immediate resolution, liquidity collapse, mark fallback, oracle delay, member-specific close-out delay. Stress-resource consumption: `rho_a = max_s S_a,s / (R_total + eps)`. Above thresholds: restrict risk-increasing orders, then require full collateral.

## Outputs

| Output | Consumer | Frequency |
|---|---|---|
| IM + fast bound | Engine pre-trade | Every run |
| Margin result with attribution | Margin service | Every run |
| Two-engine divergence `D_a` | Risk desk, model governance | Every official run |
| Stress-resource `rho_a` and top scenarios | Risk desk | Every official run |
