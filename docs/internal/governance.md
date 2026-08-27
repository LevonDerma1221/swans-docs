# Governance: committees, listing, settlement and models

## New Product Committee
Members: head of product (chair), head of risk, compliance, one independent. Meets weekly and on demand. Approves every listing against the schema checklist; nothing lists without a recorded decision. Minutes retained; conflicts declared and recorded.

## Settlement committee
Members: chief compliance officer (chair), head of product, an independent member with domain expertise, general counsel as adviser. Convened on dispute or on fallback; decides within the dispute window; decisions published with reasons; no committee member may hold a position in the contract (attested per case). Appeals under the rulebook to an independent adjudicator.

## Risk committee
Members: head of risk (chair), CTO, CEO, one independent with CCP or clearing experience. Approves margin and fee parameters, model changes, offset-stage changes, liquidity-provider relief criteria and any intraday parameter action in stress. Reviews backtests monthly and validation annually.

## Model governance
Every model (pricing, hazard, margin, fee, marks filter) has an owner, a version, validation status, backtest history and a challenger. Changes follow change management; independent validation before production for margin and marks; documentation shared with the CCP and clearing members.

## Contract engine governance
LLM components are used only for candidate surfacing, scoring and intake decomposition; every output passes deterministic validation against the schema definition; no contract is listed without New Product Committee approval. Prompts are versioned; inputs and outputs logged; member intake data is never used to train or prompt beyond the request.

## Fee governance
Fee parameters are versioned reference data; anti-gaming monitoring reports monthly to the risk committee; the aggregation window for anti-gaming is the trading day, keyed on the affiliate group (member LEI and declared affiliates), recomputed at end of day with adjustments in the fee file.

## Conflicts and product governance
Target market: professional clients and eligible counterparties only. Each schema carries its hedging rationale and harm analysis; product governance records retained per PROD.
