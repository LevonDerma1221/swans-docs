# WebSocket API

`wss://api.swanseventexchange.com/v1/ws`. Authenticate with the bearer token in the first message. Send `{"type":"ping"}` at least every 30 s; idle connections close after 60 s.

## Subscribe

```json
{ "type": "subscribe", "channels": [ { "name": "book", "symbols": ["PFE-FDA-NDA2201-26NOV"], "depth": 5 } ] }
```

## Channels

| Channel | Scope | Payload |
|---|---|---|
| `book` | per symbol | Snapshot on subscribe, then incremental level updates `{side, price, size}`; `size` 0 removes the level |
| `trades` | per symbol | `{trade_id, price, size, aggressor, ts, seq}` |
| `status` | per symbol or all | Halt, resume, expiry, settlement events |
| `settlement` | all | Daily settlement prices and final settlements |
| `instruments` | all | Reference data changes (new listings, spec versions) |
| `account` | member scope | Order updates, fills, position changes, margin budget changes for the member's accounts |

Every message carries `seq` (global) and `channel_seq`. A gap in `channel_seq` means a missed message: resubscribe.

## Close codes

| Code | Reason |
|---|---|
| 1000 | idle |
| 1001 | going_away (server restart) |
| 4001 | slow_consumer |
| 4002 | rate_limited |
| 4003 | malformed |
