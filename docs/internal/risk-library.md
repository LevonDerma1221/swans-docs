# Core analytics library (`libswansrisk`)

One library; the venue services, margin engine and risk engine call it. No service dependencies. Numerics Eigen 3.4; RNG PCG64; no exceptions in hot paths. Production core in C++20; calibration and research in Python.

```
swans/risk/
  instruments/  BinaryEvent, MeasuredEvent, Family (standalone, nested, ME, cluster), Hedge
  marks/        candidates, logit combiner, filter, family projection            (V6 §11)
  models/       IPricingModel, ICalibrator, IMarginModel, IHazardModel           ← Mehdi's implementations
  factors/      latent loading file loader with Σ B² + w² = 1 assertion           (V6 §10.1)
                factor identification: global tier F^G from cross-family liquid instruments;
                family-local tier F^g from within-family data only
  simulate/     latent diffusion, marked jump-to-resolution, cluster resolution, MPOR paths   (V6 §9, §13)
                common/family/idiosyncratic dependence (hierarchical latent model)
  margin/       structural max loss over admissible states, terminal VaR/ES, MPOR VaR/ES,
                jump (sign-sensitive), add-ons (liq, conc, oracle, model, APC), event ramp, cap   (V6 §5, §12–15)
                margin call lifecycle (DRAFT → ISSUED → SATISFIED → FAILED → DEFAULT_ESCALATION)
  collateral/   eligibility, haircuts, collateral equity, resource comparison, withdrawal capacity
  fees/         v3 fee components, z-score, smoothstep, attribution
  measures/     factor exposures, scenarios, stress-resource consumption
  io/           loaders, PB schedule writer, price-risk file writer, SFTP batch file writer
```

## Two-engine architecture

The library implements two risk engines that share the same factor state and instrument valuation maps:

| Engine | Purpose | Authority |
|---|---|---|
| **MPOR engine** | Simulates portfolio close-out loss over the margin period of risk `[t, t+h]` | Authoritative for clearing IM (VaR/ES) |
| **Terminal engine** | Draws terminal joint law of factors through copula; evaluates to-resolution loss | Challenger/diagnostics, stress, structural validation |

**Shared-state invariant.** Both engines MUST consume the same instantiation of `Z = (F^G, F^g, ε)` and the same instrument-to-value maps `V(t, Z)`. They differ only in how they use Z — the MPOR engine evolves factors over the close-out interval; the terminal engine draws their terminal joint law. A run that sources the two engines from different factor states is rejected.

**Cross-engine divergence.** On every official run, the engine computes both readings and records divergence per account:

```
D_a = |R_MPOR_a - R_terminal_a| / max(R_MPOR_a, R_terminal_a, ε)
```

`D_a` above a governance threshold routes to the risk desk and contributes to `A_model`. Persistent divergence is a model-governance trigger.

## Core interfaces

```cpp
namespace swans::risk {
struct InstrumentRef { enum Kind : uint8_t { Venue, Hedge } kind; uint32_t id; };
struct FamilyDef { uint32_t id; FamilyType type; std::vector<uint32_t> members; std::vector<std::vector<uint8_t>> admissible; };
struct MarketSnapshot { Timestamp as_of; std::unordered_map<uint32_t,double> fair_mark, settle, hedge_price;
                        std::unordered_map<std::string,double> implied_move, factor_level; std::vector<FamilyDef> families; };
struct EventState { double p, hazard1, hazard0, t_to_event_years, event_ramp; };
struct Position { InstrumentRef ref; double qty; };
struct Portfolio { AccountId account; std::vector<Position> positions; };
struct MarginParams { double var_conf=0.99, es_conf=0.975, kappa_es=1.0; int mpor_days_base=2;
                      bool allow_family_netting=true; uint8_t offset_stage=1; double liq_tier_beta, conc_zeta, conc_threshold,
                      model_risk_zeta, apc_theta, oracle_addon; bool market_maker_relief=false; };
struct MarginResult { double im, l_gross, var_mpor, es_mpor, im_jump, floor, a_liq, a_conc, a_oracle, a_model, a_event, a_apc;
                      double lambda_event; int mpor_days; double fast_bound;
                      double terminal_var, terminal_es, divergence_d;    // two-engine outputs
                      std::vector<std::pair<std::string,double>> attribution; };

class IPricingModel { public: virtual ~IPricingModel()=default;
    virtual double price(const ContractSpec&, const EventState&, const MarketSnapshot&) const=0;
    virtual std::vector<double> factorSensitivities(const ContractSpec&, const EventState&, const MarketSnapshot&) const=0; };
class ICalibrator { public: virtual ~ICalibrator()=default;
    virtual std::unordered_map<uint32_t,EventState> calibrate(const std::vector<ContractSpec>&, const MarketSnapshot&) const=0; };
class IHazardModel { public: virtual ~IHazardModel()=default;
    virtual std::pair<double,double> intensities(const ContractSpec&, const EventState&, Timestamp) const=0; };   // Λ¹, Λ⁰
class IMarginModel { public: virtual ~IMarginModel()=default;
    virtual MarginResult margin(const Portfolio&, const MarketSnapshot&, const MarginParams&) const=0;
    virtual double incremental(const Portfolio&, const Position&, const MarketSnapshot&, const MarginParams&) const=0;
    virtual double fastBound(const Position&, const MarketSnapshot&) const=0; };

double structuralMaxLoss(const Portfolio&, const MarketSnapshot&);            // over admissible states, family by family
std::vector<double> projectToFamily(const std::vector<double>& marks, const FamilyDef&);
struct FeeInputs { double p; int side; double notional; double tau; double liquidity_L; std::map<uint32_t,double> conc_by_family;
                   double unwind_U; double maker_Q; };
struct FeeResult { double bps; std::map<std::string,double> components; bool floor_applied; };
FeeResult computeFee(const FeeInputs&, const FeeParams&);
}
```

## Quantitative specification

### Portfolio representation

Signed position matrix `Q ∈ R^{A×K}` where `Q_ak > 0` is long YES, `Q_ak < 0` is short YES. Contract multiplier matrix `D_M = diag(M_1, ..., M_K)`. Marked portfolio value:

```
V_a(t) = Σ_k Q_ak · M_k · X_k(t)
```

### Event-contract dynamics

Bounded diffusion plus marked jumps to resolution:

```
dX_t = μ_t dt + σ(t, X_t, Z_t) · X_t⁻(1-X_t⁻) dW_t + (1-X_t⁻) dN¹_t - X_t⁻ dN⁰_t
```

With marked intensities `Λ¹ = X·λ¹(t, X, Z)`, `Λ⁰ = (1-X)·λ⁰(t, X, Z)` and martingale correction `μ_t = -X(1-X)(λ¹ - λ⁰)`.

### Common, family and idiosyncratic dependence

Hierarchical latent model for event contract k:

```
Z_k = a_k + B^G_k · F^G + B^g(k)_k · F^g(k) + w_k · ε_k
```

**Factor identification rule.** A factor is admitted to the global tier `F^G` only if it is jointly identifiable across families from instruments that several families share (e.g. a rates factor appears in bonds, rate options and macro contracts). Family-local factors are identified only within their own family. Global factors are pinned by liquid traditional instruments; for an event contract only the loadings `B^G_k` must be estimated.

### Simulation requirements

- Common random numbers for before/after hypothetical-trade comparisons
- Random seed, generator algorithm, scenario-set version, path count persisted
- Probability states remain in [0, 1]; legal resolution maps exactly to settlement state
- Structural-family invariants validated before and after scenario transformation
- Parallel execution preserves scenario identity and deterministic replay
- Results include numerical standard-error when Monte Carlo error can affect a margin decision

**Golden tests:** Mehdi's Python reference (structural max loss, terminal VaR/ES, concentration add-on, fee lab archetypes) produces golden outputs; C++ must match to tolerance. **Backtests** disclose synthetic price paths wherever venue history is absent.
