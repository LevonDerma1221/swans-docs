# PB API

For prime brokers. REST with the `pb` scope, mTLS required. Every call is journaled with the caller's identity.

| Endpoint | Purpose |
|---|---|
| `GET /v1/pb/accounts` | Accounts managed by this PB |
| `PUT /v1/pb/accounts/{account}/limits` | Set `max_order_qty`, `max_gross_notional`, `max_net_notional`, `max_daily_loss`, `max_swans_im` |
| `GET /v1/pb/accounts/{account}/limits` | Read limits and current usage |
| `POST /v1/pb/accounts/{account}/kill` | Kill switch: reject new orders, cancel resting orders, within 1 second. `POST .../unkill` to restore |
| `POST /v1/pb/kill` | Kill switch for all accounts of this PB |
| `GET /v1/pb/trades?date=` | Trades for managed accounts with state |
| `GET /v1/pb/positions?date=` | Positions per account |
| `GET /v1/pb/margin?date=` | SWANS margin schedule per account |

Limits take effect atomically on the next order. Drop copies are delivered over FIX (`TradeCaptureReport`) in real time; files are available after 18:00 London.
