# Quickstart

Two paths: FIX for order entry, REST/WebSocket for data. Both against the simulation environment; credentials issued at onboarding.

## 1. Read the catalogue

```bash
curl -s -H "Authorization: Bearer $TOKEN" \
  "https://api-sim.swanseventexchange.com/v1/instruments?state=trading&schema=S01" | jq '.data[0]'
```

```json
{
  "symbol": "PFE-FDA-NDA2201-26NOV",
  "isin": "GB00SWANS0017",
  "schema": "S01",
  "settlement_kind": "binary",
  "currency": "GBP",
  "payout": "100.00",
  "tick": "0.005",
  "min_qty": "1",
  "max_order_qty": "5000",
  "position_limit": "20000",
  "issuer_ticker": "PFE",
  "catalyst_id": "FDA-NDA-2201",
  "question": "Will the FDA approve NDA 2201 (drug X) on or before 30 November 2026?",
  "source": { "code": "FDA", "uri": "https://www.fda.gov/…", "tier": 1 },
  "listing_time": "1756166400000000000",
  "last_trading_time": "1764460800000000000",
  "expected_resolution": "1764547200000000000",
  "settlement_deadline": "1765152000000000000",
  "fallback_rule": "settle_zero",
  "family": { "id": "0", "type": "standalone" },
  "hedge_map": [ { "type": "single_stock_option", "identifier": "PFE", "ccp": "LCHSA" } ],
  "state": "trading",
  "transparency_mode": "realtime",
  "spec_version": 3
}
```

## 2. Subscribe to the book

```python
import asyncio, json, websockets

async def main():
    async with websockets.connect("wss://api-sim.swanseventexchange.com/v1/ws") as ws:
        await ws.send(json.dumps({"type": "auth", "token": TOKEN}))
        await ws.send(json.dumps({"type": "subscribe", "channels": [
            {"name": "book", "symbols": ["PFE-FDA-NDA2201-26NOV"], "depth": 5},
            {"name": "trades", "symbols": ["PFE-FDA-NDA2201-26NOV"]}]}))
        async for raw in ws:
            msg = json.loads(raw)
            print(msg["channel"], msg["seq"], msg.get("data"))
asyncio.run(main())
```

## 3. Send an order over FIX (QuickFIX/Python)

```python
import quickfix as fix, quickfix44 as fix44

def new_order(session_id, cl_ord_id, symbol, side, qty, price, account, clearer_lei, trader):
    m = fix44.NewOrderSingle()
    m.setField(fix.ClOrdID(cl_ord_id)); m.setField(fix.Symbol(symbol))
    m.setField(fix.Side(side)); m.setField(fix.OrderQty(qty)); m.setField(fix.OrdType(fix.OrdType_LIMIT))
    m.setField(fix.Price(price)); m.setField(fix.TimeInForce(fix.TimeInForce_GOOD_TILL_CANCEL))
    m.setField(fix.Account(account)); m.setField(fix.OrderCapacity(fix.OrderCapacity_PRINCIPAL))
    m.setField(fix.TransactTime())
    for role, pid in ((1, MY_LEI), (4, clearer_lei), (12, trader)):
        g = fix44.NewOrderSingle.NoPartyIDs()
        g.setField(fix.PartyID(pid)); g.setField(fix.PartyIDSource('N')); g.setField(fix.PartyRole(role)); m.addGroup(g)
    m.setField(fix.StringField(20050, "DEC-001")); m.setField(fix.StringField(20051, "EXE-001")); m.setField(fix.BoolField(20052, False))
    fix.Session.sendToTarget(m, session_id)
```

Expected: `ExecutionReport` with `ExecType=0` (New) then `ExecType=F` (Trade) if it crosses. Rejections carry tag 20001.

## 4. Check your margin budget

```bash
curl -s -H "Authorization: Bearer $TOKEN" "https://api-sim.swanseventexchange.com/v1/accounts/ACC-1001/margin"
```

```json
{ "status": "success",
  "data": { "account": "ACC-1001", "im": "54000.00", "budget": "250000.00", "used": "54000.00",
            "available": "196000.00", "version": "8812", "as_of": "1756220400000000000",
            "attribution": { "structural_max_loss": "60000.00", "var_mpor": "31000.00", "es_mpor": "36000.00",
                             "jump": "52000.00", "a_liq": "1200.00", "a_conc": "0.00", "a_event": "800.00",
                             "lambda_event": "0.00", "mpor_days": 2 } },
  "seq": "1849221", "server_time": "1756220400123456789" }
```

A sample repository with these scripts, the FIX data dictionary and the conformance scripts is provided at onboarding.
