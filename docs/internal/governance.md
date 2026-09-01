# Governance: committees, listing, settlement, models and risk

## New Product Committee
Members: head of product (chair), head of risk, compliance, one independent. Meets weekly and on demand. Approves every listing against the schema checklist; nothing lists without a recorded decision. Minutes retained; conflicts declared and recorded.

## Settlement committee
Members: chief compliance officer (chair), head of product, an independent member with domain expertise, general counsel as adviser. Convened on dispute or on fallback; decides within the dispute window; decisions published with reasons; no committee member may hold a position in the contract (attested per case). Appeals under the rulebook to an independent adjudicator.

## Risk committee
Members: head of risk (chair), CTO, CEO, one independent with clearing experience. Approves margin and fee parameters, model changes, offset-stage changes, liquidity-provider relief criteria and any intraday parameter action in stress. Reviews backtests monthly and validation annually. Approves cross-margin group additions and netting layer expansions.

## Model governance

Every model (pricing, hazard, margin, fee, marks filter) has:

| Attribute | Description |
|---|---|
| `model_id` | Unique identifier |
| Semantic version | `major.minor` |
| Owner | Named individual |
| Purpose and approved scope | Which products/accounts it covers |
| Parameter schema | Versioned separately from model code |
| Validation status | Pending, validated, approved, retired |
| Effective date | When it enters production |
| Code commit / build hash | Exact reproducibility |
| Rollback version | Previous version for immediate revert |

### Model registry

All production models are registered. Policy parameters are versioned separately from mathematical model code. Official runs persist the full lineage tuple: portfolio snapshot, mark snapshot, collateral snapshot, model version, parameter set, policy version, scenario set, random seed, path count, software build hash, infrastructure image digest.

### Two-engine governance

The MPOR engine is the clearing-IM authority. The terminal/challenger engine results are compared quantitatively only under aligned horizon/measure/output; otherwise they are complementary diagnostics. The engine never blends MPOR and terminal results into a single figure without an approved, versioned reconciliation rule. Persistent or portfolio-wide divergence is a model-governance trigger.

### Backtesting and validation

- Backtesting is performed at account/product/portfolio levels required by the margin methodology
- Breaches store realized P&L definition, amount, IM, MPOR, portfolio snapshot and root-cause classification
- Coverage tests, sensitivity, stress, parameter stability and challenger comparisons are automated
- Model change promotion requires independent validation evidence and before/after impact analysis
- Margin overrides are recorded as separate policy-adjustment events with actor, reason, approval and expiry, preserving the original calculated result

### Calculation invariants

| Invariant | Control |
|---|---|
| Probability bounds | `0 ≤ X_fair ≤ 1`; structural family constraints satisfied |
| Max-loss cap | Contract-loss IM ≤ remaining structural gross max loss |
| Origin segregation | Customer/house netting follows legally permitted scope |
| Collateral uniqueness | Each lot credited only to entitled account/pool within available amount |
| Replay | Same immutable inputs + deterministic seed/build reproduce result within tolerance |
| Report reconciliation | Totals reconcile to source ledgers and manifest counts |

## Contract engine governance
LLM components are used only for candidate surfacing, scoring and intake decomposition; every output passes deterministic validation against the schema definition; no contract is listed without New Product Committee approval. Prompts are versioned; inputs and outputs logged; member intake data is never used to train or prompt beyond the request.

## Fee governance
Fee parameters are versioned reference data; anti-gaming monitoring reports monthly to the risk committee; the aggregation window for anti-gaming is the trading day, keyed on the affiliate group (member LEI and declared affiliates), recomputed at end of day with adjustments in the fee file.

## Conflicts and product governance
Target market: professional clients and eligible counterparties only. Each schema carries its hedging rationale and harm analysis; product governance records retained per PROD.
