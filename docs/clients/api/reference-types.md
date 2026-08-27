# Reference types

## ContractSpec

| Field | Type | Description |
|---|---|---|
| `symbol` | string | `ISSUER-SCHEMA-CATALYST-EXPIRY` |
| `isin`, `cfi`, `fisn` | string | Identifiers |
| `schema` | string | `S01`…`S10` |
| `settlement_kind` | enum | `binary`, `measured` |
| `currency` | enum | `GBP`, `USD`, `EUR` |
| `payout` | decimal | `100.00` |
| `tick` | decimal | `0.005` |
| `min_qty`, `max_order_qty`, `position_limit` | integer | |
| `issuer_lei`, `issuer_ticker` | string | Empty for macro contracts |
| `catalyst_id` | string | Catalogue key |
| `question` | string | Exact settlement question |
| `source` | object | `{code, uri, tier}` |
| `listing_time`, `last_trading_time`, `expected_resolution`, `settlement_deadline` | timestamp | |
| `fallback_rule` | enum | `settle_zero`, `settle_last`, `void` |
| `package_legs` | array | `[{symbol, ratio}]` |
| `hedge_map` | array | `[{type, identifier, ccp}]` |
| `state` | enum | `pending`, `listed`, `trading`, `halted`, `expired`, `settled`, `cancelled` |
| `transparency_mode` | enum | `realtime`, `delayed`, `top_only` |
| `spec_version` | integer | |

## Order

`order_id, client_order_id, symbol, account, side (buy|sell), price, qty, leaves, cum, avg_price, tif, post_only, reduce_only, status, seq, update_seq, placed_at, updated_at`

## Fill

`trade_id, order_id, client_order_id, symbol, account, side, price, qty, liquidity (maker|taker), fee, clearing_state, tvtic, seq, ts`

## Position

`account, symbol, net_qty, entry_price, mark, unrealised_pnl, realised_pnl, vm_cumulative, legs (for packages), as_of`

## Margin

`account, im, budget, used, available, version, as_of, offsets_applied (array of offset_group_id)`

## Settlement

`symbol, kind (daily|final|fallback), value, method, status, source_uri, evidence_sha256, determined_at, final_at`

## Limits (clearer)

`account, max_order_qty, max_gross_notional, max_net_notional, max_daily_loss, max_swans_im, kill`
