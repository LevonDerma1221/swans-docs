# Pre-trade risk (engine stage)

Runs in the engine shard process between decode and match. **Fail-closed except for the margin check**, which degrades (see below).

## Checks, in order
1. Member, account, instrument enabled; `Trading` state; before `last_trading_time`.
2. Clearer kill (`ClearerLimits.kill`).
3. `1 ≤ price ≤ 199`; `qty ≥ min_qty`; `qty ≤ min(instrument.max_order_qty, limits.max_order_qty)`.
4. Static price band vs reference price (most recent VM window mark, or last trade if newer): default 40 ticks.
5. Position limit: `|net_qty + pending_qty + open_same_side_qty + signed_qty| ≤ position_limit`.
6. **Full-collateral accounts:** balance check — `available ≥ max_loss`, where `max_loss = price × payout × qty` (buy) or `(1−price) × payout × qty` (sell). Reject with `INSUFFICIENT_BALANCE` (1055) if insufficient. Uses cached balance snapshot from `collateral.balances` stream.
7. **PB-managed accounts:** Clearer gross/net notional after this order ≤ limits; daily loss ≤ limit. **Margin budget:** `IM(portfolio + open orders + this order at limit price) − IM(portfolio + open orders) ≤ budget_available`, where `budget_available = limits.max_swans_im − IM_current − reserved_open_orders`. Fast upper bound first: `qty × payout × max(p, 1−p)`; precise call to the margin engine only if the bound fails.
8. Post-only / reduce-only semantics.

Self-trade prevention is in the matching stage, not here.

## Degradation

**PB-managed accounts:** Budgets arrive from the margin service with a version and timestamp. If no update for > N seconds (default 30), the stage continues on the fast bound only, with all limits scaled by a configurable factor (default 0.5), and raises an alert. It rejects (`RISK_UNAVAILABLE`) only if the bound itself cannot be computed (no reference data).

**Full-collateral accounts:** Balance snapshots arrive from the collateral service via `collateral.balances`. If no update for > 30 seconds, all full-collateral orders are rejected (`RISK_UNAVAILABLE`) until a fresh snapshot arrives.

## Volatility controls (RTS 7)
Dynamic halt: if last trade moves > 30 ticks vs the price 60 s earlier, halt the instrument for 2 minutes; parameters per schema. Operator halt/resume via the ops API, journaled.

## State held
Reference snapshot; per-account positions (net + pending); per-account open-order aggregates; `ClearerLimits` with version; margin budget cache `{account → {im_current, budget, version, ts}}`; collateral balance cache `{account → {available, locked, version, ts}}`.

## Latency budget
Checks 1–6, 8: p99 ≤ 20 µs. Check 7 with bound: ≤ 30 µs. Precise IM call: ≤ 500 µs (rare).
