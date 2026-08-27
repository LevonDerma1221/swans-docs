# Rate limits, throttles and close codes

| Interface | Limit | On breach | Persistent breach |
|---|---|---|---|
| FIX order entry | 200 msg/s per session (configurable per member; liquidity providers up to 2,000) | `BusinessMessageReject` (j), 380=20099; message dropped | Session logout after 3 warnings in 60 s; re-logon permitted after 60 s |
| FIX order-to-trade ratio | 500:1 per account per day **[confirm: RTS 9 requires the venue to set a ratio; the value is a SWANS policy parameter]** | Warning at 80%; new orders rejected with 1007 `OTR_EXCEEDED` above | Compliance review |
| REST | 100 req/s per member; 10 req/s on `margin/simulate` | HTTP 429, `Retry-After` | Key suspended for 5 min after 10 breaches in 60 s |
| WebSocket | 20 subscribe/unsubscribe per second; 50 symbols per channel per connection; 10 connections per member | `error` frame, code `RATE_LIMITED` | Close 4002; reconnect refused for 5 min |
| Clearer API | 20 req/s | HTTP 429 | none (control plane) |
| Market data | 1 snapshot request per symbol per second | dropped | none |

## WebSocket close codes

| Code | Reason | Meaning |
|---|---|---|
| 1000 | idle | No frames for 60 s |
| 1001 | going_away | Server restart; reconnect with snapshot |
| 1009 | | Frame over 64 KiB |
| 4001 | slow_consumer | Client stopped reading; buffered updates exceeded 5 MB |
| 4002 | rate_limited | Persistent breach |
| 4003 | malformed | Repeated invalid frames |
| 4004 | auth_expired | Token expired mid-session; reconnect with a fresh token |

## Recovery

After any close, resubscribe; the first message per channel is a snapshot with `seq`. Apply buffered updates with `seq` greater than the snapshot's.
