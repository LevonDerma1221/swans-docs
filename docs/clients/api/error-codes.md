# Error codes

## Pre-trade rejection codes (FIX tag 20001, REST `code`)

| Code | Name | Meaning |
|---|---|---|
| 1001 | MEMBER_DISABLED | Member not enabled for trading |
| 1002 | ACCOUNT_DISABLED | Account not enabled |
| 1003 | INSTRUMENT_NOT_TRADING | Contract not in `trading` state |
| 1004 | OUTSIDE_HOURS | Outside trading hours or after last trading time |
| 1005 | MISSING_RTS24 | Required RTS 24 fields absent |
| 1006 | DUPLICATE_CLORDID | ClOrdID matches an open order |
| 1007 | OTR_EXCEEDED | Order-to-trade ratio limit exceeded |
| 1010 | PB_KILL | Prime broker kill switch active |
| 1020 | PRICE_OUT_OF_RANGE | Price outside [0.005, 0.995] or not on tick |
| 1021 | QTY_BELOW_MIN | |
| 1022 | QTY_ABOVE_MAX | Contract or PB max order size |
| 1030 | PRICE_BAND | Outside static price band |
| 1040 | POSITION_LIMIT | Would breach position limit (including pending trades) |
| 1050 | PB_GROSS_LIMIT | |
| 1051 | PB_NET_LIMIT | |
| 1052 | PB_DAILY_LOSS | |
| 1060 | MARGIN_BUDGET | Incremental IM exceeds available budget |
| 1070 | SELF_MATCH | Cancelled by self-trade prevention (incoming-cancel mode) |
| 1080 | POST_ONLY_CROSS | Post-only order would cross |
| 1081 | REDUCE_ONLY_INCREASE | Reduce-only order would increase exposure |
| 1090 | FOK_UNFILLABLE | |
| 1099 | RISK_UNAVAILABLE | Risk check could not be evaluated; retry |

## API error codes

| Code | HTTP | Meaning |
|---|---|---|
| INVALID_REQUEST | 400 | Malformed parameters |
| UNAUTHORIZED | 401 | Missing or invalid token |
| FORBIDDEN | 403 | Scope or account not permitted |
| INSTRUMENT_NOT_FOUND | 404 | |
| ORDER_NOT_FOUND | 404 | |
| RATE_LIMITED | 429 | |
| INTERNAL | 500 | |

## Session-level (FIX)

`BusinessMessageReject` (j) with `BusinessRejectReason` (380): 20099 throttle exceeded; standard values otherwise.
