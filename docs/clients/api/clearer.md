# Clearer API

For clearing members. REST with the `clearer` scope, mTLS required. Every call is journaled with the caller's identity.

| Endpoint | Purpose |
|---|---|
| `GET /v1/clearer/accounts` | Accounts cleared by this member, with CCP references |
| `PUT /v1/clearer/accounts/{account}/limits` | Set `max_order_qty`, `max_gross_notional`, `max_net_notional`, `max_daily_loss`, `max_swans_im` |
| `GET /v1/clearer/accounts/{account}/limits` | Read limits and current usage |
| `POST /v1/clearer/accounts/{account}/kill` | Kill switch: reject new orders, cancel resting orders, within 1 second. `POST …/unkill` to restore |
| `POST /v1/clearer/kill` | Kill switch for all accounts of this clearing member |
| `GET /v1/clearer/trades?date=` | Trades for cleared accounts with clearing state |
| `GET /v1/clearer/positions?date=` | Positions per account |
| `GET /v1/clearer/margin?date=` | SWANS margin schedule per account |
| `GET /v1/clearer/files/ccp-parameters?date=` | CCP parameter file |

Limits take effect atomically on the next order. Drop copies are delivered over FIX (`TradeCaptureReport`) in real time; files are available after 18:00 London.
