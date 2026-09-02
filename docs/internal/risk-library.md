# Risk library

One library; the venue services, margin engine and risk engine call it. No service dependencies. Numerics Eigen 3.4; RNG PCG64; no exceptions in hot paths. Production core in C++20; calibration and research in Python.

```
swans/risk/
  instruments/  BinaryEvent, MeasuredEvent, Family, Hedge
  marks/        candidates, logit combiner, filter, family projection
  models/       IPricingModel, ICalibrator, IMarginModel, IHazardModel   <- Mehdi's implementations
  factors/      latent loading file loader
  simulate/     latent diffusion, marked jump-to-resolution, cluster resolution, MPOR paths
  margin/       structural max loss, terminal VaR/ES, MPOR VaR/ES, add-ons, event ramp, margin call lifecycle
  collateral/   eligibility, haircuts, collateral equity, resource comparison
  fees/         v3 fee components, z-score, smoothstep, attribution
  measures/     factor exposures, scenarios, stress-resource consumption
  io/           loaders, PB schedule writer, price-risk file writer, SFTP batch file writer
```

## Two-engine architecture

| Engine | Purpose | Authority |
|---|---|---|
| **MPOR engine** | Close-out loss over the margin period of risk `[t, t+h]` | Authoritative for clearing IM |
| **Terminal engine** | Terminal joint law through factor copula; to-resolution loss | Challenger, diagnostics, stress |

**Common economic-state contract.** Both engines share the same versioned factor taxonomy, calibration snapshot, and instrument-to-value mappings. They may carry different path/terminal-specific state variables (`S_MPOR = S_shared + U_path`; `S_terminal = S_shared + U_terminal`). A run is rejected only for a semantic/version mismatch in the shared contract, not for different dimensions or horizons.

**Cross-engine divergence (comparable-run rule).** Divergence `D_a` is diagnostic only when outputs are aligned on horizon, measure, portfolio snapshot and output definition. A difference between a 5-day close-out loss and a to-resolution terminal loss is expected, not a defect. Persistent divergence under aligned comparison is a model-governance trigger.

## Key interfaces

Models implement `IPricingModel`, `ICalibrator`, `IMarginModel`, `IHazardModel`. The library never contains model logic — Mehdi's implementations plug in. See the risk engine technical specification for full interface definitions and quantitative detail.

## Simulation

Event-contract dynamics: bounded diffusion plus marked jumps to resolution. Hierarchical latent factor model with global, family-local, and idiosyncratic components. Factor identification rule: the conceptual basis for portfolio-margin offsets may include complement/substitute relationships, shared inputs, or common external drivers — not limited to a named global factor.

Requirements: common random numbers for before/after comparisons; deterministic replay from seed + build hash; probability states in [0, 1]; structural-family invariants validated per scenario; Monte Carlo standard error reported when it can affect a margin decision.

## Golden tests

Mehdi's Python reference (structural max loss, terminal VaR/ES, concentration add-on, fee lab archetypes) produces golden outputs; C++ must match to tolerance. Backtests disclose synthetic price paths wherever venue history is absent.
