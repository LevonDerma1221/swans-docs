# REST API

Base URL issued at onboarding, e.g. `https://api.swanseventexchange.com`. All timestamps are nanoseconds since the Unix epoch as strings; prices are decimal strings with 3 fractional digits; quantities and identifiers are strings.

## Authentication

OAuth2 client credentials. Obtain a bearer token from `/oauth/token`; send `Authorization: Bearer …`. Tokens expire after 1 hour. Reference and market data endpoints are readable by any member; account endpoints are scoped to the member's own accounts; clearer endpoints require the `clearer` scope.

Order entry is FIX only in v1.

## Envelope

```json
{ "status": "success", "data": { … }, "seq": "1849221", "server_time": "1756200000123456789" }
{ "status": "error",   "data": { "code": "INSTRUMENT_NOT_FOUND", "message": "…" }, "server_time": "…" }
```

## Pagination

`limit` (default 100, max 500), `before_cursor`, `at_or_before_seq`. Responses include `next_cursor` when more results exist.

## Reference data

| Endpoint | Returns |
|---|---|
| `GET /v1/time` | Server time |
| `GET /v1/schemas` | Schema policies (S-01…S-10): question templates, source hierarchies, fallback rules |
| `GET /v1/instruments?state=&schema=&issuer=` | Contract specifications ([ContractSpec](reference-types.md#contractspec)) |
| `GET /v1/instruments/{symbol}` | One contract |
| `GET /v1/instruments/{symbol}/hedges` | Hedge map |
| `GET /v1/calendar` | Trading days and hours |

## Market data

| Endpoint | Returns |
|---|---|
| `GET /v1/book/{symbol}?depth=5` | Book snapshot with `seq` |
| `GET /v1/trades/{symbol}` | Recent trades, paginated |
| `GET /v1/candles/{symbol}?interval=1m` | OHLCV |
| `GET /v1/stats/{symbol}` | Last, volume, open interest, reference price, daily settlement |
| `GET /v1/settlements?from=&to=` | Daily settlement prices |
| `GET /v1/resolutions?state=final` | Final settlements with evidence references |

## Account data (member scope)

| Endpoint | Returns |
|---|---|
| `GET /v1/accounts` | Accounts and their clearing mapping |
| `GET /v1/accounts/{account}/orders?state=open` | Orders |
| `GET /v1/accounts/{account}/fills` | Fills, paginated |
| `GET /v1/accounts/{account}/positions` | Net positions, entry, mark, unrealised PnL, package decomposition |
| `GET /v1/accounts/{account}/margin` | SWANS IM, budget, used, available, version |
| `GET /v1/accounts/{account}/vm?date=` | Variation margin history |
| `POST /v1/accounts/{account}/hedges` | Upload hedge positions for risk reporting (Product A) |
| `GET /v1/accounts/{account}/risk` | Portfolio risk report on venue plus uploaded hedge positions |

## Files

| Endpoint | Returns |
|---|---|
| `GET /v1/files/price-risk?date=` | Daily price and risk file (members, prime brokers) |
| `GET /v1/files/ccp-parameters?date=` | CCP parameter file (CCP, clearing members) |
| `GET /v1/files/triggers?month=` | Offset trigger report |

## Rate limits

100 requests/second per member; HTTP 429 with `Retry-After` on excess.
