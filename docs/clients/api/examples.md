# Examples

Every REST endpoint with a request and a response; FIX message logs for the order-entry cycle. Values are from the simulation environment.

## REST

### `GET /v1/time`
```bash
curl -s https://api-sim.swanseventexchange.com/v1/time
```
```json
{ "status": "success", "data": { "server_time": "1756220400123456789" }, "seq": "1849221", "server_time": "1756220400123456789" }
```

### `GET /v1/schemas`
```json
{ "status": "success", "data": [ { "code": "S01", "name": "REGCERT-DECISION", "template": "Will <issuer> <action> <subject> by <date>?",
  "family_type": "standalone", "sources": [ { "code": "FDA", "tier": 1 }, { "code": "EMA", "tier": 1 } ],
  "fallback_rules": [ "settle_zero", "void" ], "version": 4 } ], "seq": "1849221", "server_time": "…" }
```

### `GET /v1/instruments/{symbol}`
See [Quickstart](../quickstart.md) for the full `ContractSpec`. Unknown symbol:
```json
{ "status": "error", "data": { "code": "INSTRUMENT_NOT_FOUND", "message": "PFE-FDA-NDA2202-26NOV" }, "server_time": "…" }
```

### `GET /v1/book/{symbol}?depth=5`
```json
{ "status": "success", "data": { "symbol": "PFE-FDA-NDA2201-26NOV",
  "bids": [ { "price": "0.695", "size": "400" }, { "price": "0.690", "size": "1200" } ],
  "asks": [ { "price": "0.705", "size": "350" }, { "price": "0.710", "size": "900" } ],
  "last": { "price": "0.700", "size": "200", "aggressor": "sell", "ts": "1756220399000000000" } },
  "seq": "1849220", "server_time": "…" }
```

### `GET /v1/trades/{symbol}?limit=2`
```json
{ "status": "success", "data": { "items": [
  { "trade_id": "5550012", "price": "0.700", "size": "200", "aggressor": "sell", "ts": "1756220399000000000", "seq": "1849218" },
  { "trade_id": "5550011", "price": "0.700", "size": "1800", "aggressor": "sell", "ts": "1756220398500000000", "seq": "1849217" } ],
  "next_cursor": "eyJzZXEiOjE4NDkyMTd9" }, "seq": "1849221", "server_time": "…" }
```

### `GET /v1/accounts/{account}/positions`
```json
{ "status": "success", "data": [ { "account": "ACC-1001", "symbol": "PFE-FDA-NDA2201-26NOV", "net_qty": "-2000", "pending_qty": "0",
  "entry_price": "0.700", "mark": "0.698", "unrealised_pnl": "400.00", "realised_pnl": "0.00", "vm_cumulative": "0.00",
  "family": { "id": "0", "type": "standalone" }, "as_of": "1756220400000000000" } ], "seq": "1849221", "server_time": "…" }
```

### `GET /v1/accounts/{account}/fills?limit=1`
```json
{ "status": "success", "data": { "items": [ { "trade_id": "5550012", "order_id": "98765432", "client_order_id": "42",
  "symbol": "PFE-FDA-NDA2201-26NOV", "account": "ACC-1001", "side": "sell", "price": "0.700", "qty": "200", "liquidity": "taker",
  "fee": { "bps": "31.4", "amount": "62.80", "components": { "base": "20.0", "certainty_pressure": "0.0", "event_window": "6.2",
  "liquidity_stress": "5.2", "concentration": "0.0", "contrarian_discount": "-4.0", "maker_rebate": "0.0", "unwind_relief": "0.0", "floor_applied": false } },
  "clearing_state": "cleared", "tvtic": "SWANS5550012", "seq": "1849218", "ts": "1756220399000000000" } ] }, "seq": "1849221", "server_time": "…" }
```

### `POST /v1/accounts/{account}/margin/simulate`
```bash
curl -s -X POST -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
  https://api-sim.swanseventexchange.com/v1/accounts/ACC-1001/margin/simulate \
  -d '{ "positions": [ { "symbol": "PFE-FDA-NDA2201-26NOV", "qty": "-2000" }, { "symbol": "PFE-FDA-NDA2201-LAUNCH-27Q1", "qty": "1000" } ],
        "hedges": [ { "type": "single_stock_option", "identifier": "PFE 26NOV 25P", "qty": "500" } ] }'
```
```json
{ "status": "success", "data": { "im": "112400.00", "structural_max_loss": "140000.00", "attribution": { "var_mpor": "61000.00",
  "es_mpor": "70500.00", "jump": "108000.00", "a_liq": "2600.00", "a_conc": "0.00", "a_model": "1800.00", "a_event": "0.00" },
  "offsets_applied": [], "offset_candidates": [ { "group": "PFE-NDA2201", "stage": "1", "status": "not_enabled" } ],
  "hedge_report": { "single_stock_option": { "net_delta_equiv": "-0.31", "note": "reported only; netting is your prime broker's decision" } },
  "fast_bound": "160000.00" }, "seq": "1849221", "server_time": "…" }
```

### Clearer: `PUT /v1/clearer/accounts/{account}/limits`
```bash
curl -s -X PUT --cert clearer.pem --key clearer.key -H "Authorization: Bearer $CLEARER_TOKEN" -H "Idempotency-Key: 7f3a…" \
  https://api-sim.swanseventexchange.com/v1/clearer/accounts/ACC-1001/limits \
  -d '{ "max_order_qty": "5000", "max_gross_notional": "5000000.00", "max_net_notional": "2000000.00", "max_daily_loss": "500000.00", "max_swans_im": "250000.00" }'
```
```json
{ "status": "success", "data": { "account": "ACC-1001", "version": "312", "effective_seq": "1849222", "kill": false }, "seq": "1849222", "server_time": "…" }
```

### Clearer: `POST /v1/clearer/accounts/{account}/kill`
```json
{ "status": "success", "data": { "account": "ACC-1001", "kill": true, "orders_cancelled": "7", "effective_seq": "1849230", "latency_ms": "184" }, "seq": "1849230", "server_time": "…" }
```

## WebSocket

Subscribe acknowledgement and first book snapshot:
```json
{ "type": "subscribed", "channel": "book", "symbol": "PFE-FDA-NDA2201-26NOV", "channel_seq": "0", "seq": "1849220",
  "data": { "bids": [ ["0.695","400"], ["0.690","1200"] ], "asks": [ ["0.705","350"], ["0.710","900"] ] } }
{ "type": "update", "channel": "book", "symbol": "PFE-FDA-NDA2201-26NOV", "channel_seq": "1", "seq": "1849223",
  "data": { "side": "bid", "price": "0.695", "size": "0" } }
{ "type": "update", "channel": "account", "account": "ACC-1001", "channel_seq": "17", "seq": "1849218",
  "data": { "kind": "fill", "order_id": "98765432", "trade_id": "5550012", "price": "0.700", "qty": "200" } }
{ "type": "update", "channel": "account", "account": "ACC-1001", "channel_seq": "18", "seq": "1849240",
  "data": { "kind": "margin", "im": "54000.00", "budget": "250000.00", "available": "196000.00", "version": "8812" } }
```

## FIX logs (tag=value, `|` = SOH)

New order, accept, fill:
```
8=FIX.4.4|35=D|49=MEMB01|56=SWANS|34=112|52=20260826-14:00:00.123456|11=42|55=PFE-FDA-NDA2201-26NOV|54=2|38=200|40=2|44=0.700|59=1|1=ACC-1001|528=P|453=3|448=5493001KJTIIGC8Y1R12|447=N|452=1|448=549300…GCM|447=N|452=4|448=TRD-77|447=N|452=12|20050=DEC-001|20051=EXE-001|20052=N|60=20260826-14:00:00.123456|10=…|
8=FIX.4.4|35=8|49=SWANS|56=MEMB01|34=211|37=98765432|11=42|17=E-1|150=0|39=0|55=PFE-FDA-NDA2201-26NOV|54=2|38=200|44=0.700|151=200|14=0|6=0|20010=1849216|60=20260826-14:00:00.123601|10=…|
8=FIX.4.4|35=8|49=SWANS|56=MEMB01|34=212|37=98765432|11=42|17=E-2|150=F|39=2|55=PFE-FDA-NDA2201-26NOV|54=2|31=0.700|32=200|151=0|14=200|6=0.700|880=5550012|20011=SWANS5550012|20010=1849218|20070=20.0|20071=0.0|20072=6.2|20073=5.2|20074=0.0|20075=-4.0|20076=0.0|20077=0.0|20078=N|60=20260826-14:00:00.123655|10=…|
```

Pre-trade rejection (margin budget):
```
8=FIX.4.4|35=8|49=SWANS|56=MEMB01|34=213|37=0|11=43|17=E-3|150=8|39=8|55=PFE-FDA-NDA2201-26NOV|54=2|38=5000|44=0.700|151=0|14=0|103=99|20001=1060|58=MARGIN_BUDGET available=196000.00 required=210000.00|20010=1849219|60=…|10=…|
```

Replace rejected after fill:
```
8=FIX.4.4|35=G|49=MEMB01|56=SWANS|34=113|11=44|41=42|37=98765432|55=PFE-FDA-NDA2201-26NOV|54=2|38=300|40=2|44=0.695|60=…|10=…|
8=FIX.4.4|35=9|49=SWANS|56=MEMB01|34=214|37=98765432|11=44|41=42|39=2|434=2|102=1|58=Order fully filled|10=…|
```

CCP reject (trade cancel):
```
8=FIX.4.4|35=8|49=SWANS|56=MEMB01|34=215|37=98765432|11=42|17=E-4|150=H|39=2|55=PFE-FDA-NDA2201-26NOV|54=2|31=0.700|32=200|151=0|14=0|6=0|880=5550012|378=8|58=CCP_REJECT code=4101|20010=1849260|60=…|10=…|
```

Drop copy to clearing member:
```
8=FIX.4.4|35=AE|49=SWANS|56=GCM01|34=77|571=TR-5550012|487=0|828=0|55=PFE-FDA-NDA2201-26NOV|32=200|31=0.700|75=20260826|60=…|880=5550012|20011=SWANS5550012|20040=3|552=1|54=2|37=98765432|11=42|1=ACC-1001|453=2|448=…|452=1|448=…|452=24|10=…|
```
