# Governance: committees, models and risk

## Committees

**New Product Committee.** Head of product (chair), head of risk, compliance, one independent. Meets weekly. Approves every listing against the schema checklist. Minutes retained; conflicts declared.

**Settlement Committee.** Chief compliance officer (chair), head of product, independent domain expert, general counsel adviser. Convened on dispute or fallback; decides within the dispute window; no member may hold a position in the contract.

**Risk Committee.** Head of risk (chair), CTO, CEO, one independent with clearing experience. Approves margin and fee parameters, model changes, offset-stage changes, liquidity-provider relief criteria. Reviews backtests monthly and validation annually.

## Model governance

Every model (pricing, hazard, margin, fee, marks) has: unique `model_id`, semantic version, named owner, approved scope, parameter schema, validation status, effective date, code commit hash, and rollback version. All production models are registered with full lineage (portfolio snapshot, mark snapshot, model version, parameter set, build hash, etc.).

**Two-engine governance.** The MPOR engine is the clearing-IM authority. The terminal engine is authoritative for diagnostics, structural validation and stress. Cross-engine divergence is diagnostic only under aligned comparison (the comparable-run rule). Persistent divergence is a model-governance trigger.

**Backtesting.** Performed at account/product/portfolio levels. Breaches store realized P&L, IM, MPOR, snapshot and root-cause classification. Model changes require independent validation and before/after impact analysis. Margin overrides are recorded separately, preserving the original calculated result.

## Contract engine governance

LLM components are used only for candidate surfacing and scoring; every output passes deterministic validation against the schema definition; no contract lists without NPC approval.

## Fee governance

Fee parameters are versioned reference data; anti-gaming monitoring reports monthly to the risk committee.

## Conflicts and product governance

Target market: professional clients and eligible counterparties only. Each schema carries its hedging rationale and harm analysis.
