# Core analytics library (`libswansrisk`)

One library; the venue services, Product A and Product B call it. No service dependencies. Numerics Eigen 3.4; RNG PCG64; no exceptions in hot paths.

```
swans/risk/
  instruments/  BinaryEvent, MeasuredEvent, Family (standalone, nested, ME, cluster), Hedge
  marks/        candidates, logit combiner, filter, family projection            (V6 §11)
  models/       IPricingModel, ICalibrator, IMarginModel, IHazardModel           ← Mehdi's implementations
  factors/      latent loading file loader with Σ B² + w² = 1 assertion           (V6 §10.1)
  simulate/     latent diffusion, marked jump-to-resolution, cluster resolution, MPOR paths   (V6 §9, §13)
  margin/       structural max loss over admissible states, terminal VaR/ES, MPOR VaR/ES, jump, add-ons, event ramp, cap   (V6 §5, §12–15)
  fees/         v3 fee components, z-score, smoothstep, attribution
  measures/     factor exposures, scenarios, XVA (bilateral only)
  io/           loaders, PB schedule writer, price-risk file writer
```

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
                      double lambda_event; int mpor_days; double fast_bound; std::vector<std::pair<std::string,double>> attribution; };

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

**Golden tests:** Mehdi's Python reference (structural max loss, terminal VaR/ES, concentration add-on, fee lab archetypes) produces golden outputs; C++ must match to tolerance. **Backtests** disclose synthetic price paths wherever venue history is absent.
