# Margin

## Launch: full collateral, margin in shadow mode

At launch, full collateral means max loss is locked per trade — no margin engine dependency for pre-trade. The margin engine runs in **shadow mode**: computing risk analytics, collecting data, and validating models, but not driving any collateral decisions.

When SWANS offers margin (capital efficiency), the margin engine becomes production. How that works is an open design question — see [Clearing](clearing.md).

## Methodology overview

**Two engines.** The MPOR engine (close-out loss simulation) is authoritative for clearing IM. A terminal engine runs as a standing challenger. Cross-engine divergence is meaningful only under aligned comparison — a 5-day close-out loss and a to-resolution terminal loss are different by design.

**Three layers, kept separate.** Structural maximum loss (exact over admissible family states); terminal VaR/ES (diagnostic); and production margin: close-out loss over a margin period of risk (MPOR).

**IM formula.**

```
IM_core  = max( VaR_99%, ES_97.5%, IM_jump, IM_floor )
IM       = min( L_gross, IM_core + add-ons )
IM_event = (1 - lambda) * IM + lambda * L_gross
```

Add-ons: liquidity, concentration, oracle, model risk, event ramp, anti-procyclicality. Event ramp smoothly converges IM to max loss as resolution approaches — no collateral cliff.

**Fair marks.** Margin is keyed to a filtered fair mark, not the last print. Hierarchy: auction price, executable microprice, decayed VWAP, model mark. Combined in logit space with weights for depth, spread, recency and staleness.

**Market-maker relief.** Shorter MPOR or lower liquidity add-on for designated LPs meeting objective criteria (quote uptime, spread, depth). Withdrawn automatically on breach.

**Gross customer margin.** Customer-origin IM is gross: sum of individual-customer IM with no inter-customer netting.

## Netting layers

| Layer | Mechanism |
|---|---|
| 1. Same-contract | Aggregate trades into net position per margin unit |
| 2. Structural family | Evaluate over admissible terminal states (exact payoff logic) |
| 3. Spread/portfolio margining | Jointly simulate related positions in approved groups; requires conceptual basis and evidence |

Inter-DCO cross-margining is a separate architecture.

## Launch posture

Full collateral with the margin engine in shadow mode, collecting live data and validating models. When margin is offered, phased rollout: hybrid margin for approved members, then market-maker relief, then broader offsets.

## Margin simulator

`POST /v1/accounts/{account}/margin/simulate` with a hypothetical portfolio returns IM, add-on attribution, event-ramp state and fast bound.

## Files published

| File | Recipient | Frequency |
|---|---|---|
| Price and risk file | Members | Periodic |
| Backtest and sensitivity report | FCA | Monthly |
| Margin schedule with attribution | Future: margin counterparty (PB, CCP, or other) | When margin is offered |
