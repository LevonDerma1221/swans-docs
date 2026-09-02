# Pre-trade risk (engine stage)

Runs in the engine shard process between decode and match. **Fail-closed.**

## Checks, in order (launch — full collateral)

1. Member, account, instrument enabled; `Trading` state; before `last_trading_time`.
2. `1 <= price <= 99`; `qty >= min_qty`; `qty <= max_order_qty`.
3. Static price band vs reference price (filtered fair mark, or last trade if newer): default 20 ticks.
4. Position limit: `|net_qty + signed_qty| <= position_limit`.
5. Balance check: `available >= max_loss`, where `max_loss = price * contract_size * qty` (buy) or `(1 - price) * contract_size * qty` (sell). Reject with `INSUFFICIENT_BALANCE` (1055) if insufficient. Uses cached balance snapshot from `collateral.balances` stream.
6. Post-only / reduce-only semantics.

Self-trade prevention is in the matching stage, not here.

## Future checks (when PB/margin offered)

- PB kill switch (`ClearerLimits.kill`)
- PB gross/net notional limits; daily loss limit
- Margin budget: `IM(portfolio + this order) - IM(portfolio) <= budget_available`
- Fast upper bound first: `qty * contract_size * max(p, 1-p)`; precise margin call only if the bound fails

## Degradation

**Balance snapshots** arrive from the collateral service via `collateral.balances`. If no update for > 30 seconds, all orders are rejected (`RISK_UNAVAILABLE`) until a fresh snapshot arrives.

## Volatility controls (RTS 7)

Dynamic halt: if last trade moves > 15 ticks vs the price 60 s earlier, halt the instrument for 2 minutes; parameters per schema. Operator halt/resume via the ops API, journaled.

## State held

Reference snapshot; per-account positions; per-account open-order aggregates; collateral balance cache `{account -> {available, locked, version, ts}}`. Future: `ClearerLimits`, margin budget cache.

## Latency budget

Checks 1-6: p99 <= 20 us.
