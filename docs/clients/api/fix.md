# FIX order entry

## Session

- FIX 4.4, tag=value, with selected FIX 5.0 SP2 fields. TLS 1.3 required.
- One session per member connection; `SenderCompID` and `TargetCompID` issued at onboarding.
- Heartbeat interval 30 s. Standard resend and sequence reset semantics.
- `CancelOnDisconnect` per session (default on for liquidity providers).
- Throttle: 200 messages/second per session by default; excess is rejected with `BusinessMessageReject` (j), `BusinessRejectReason` 20099.

## Message summary

| Msg | Type | Direction | Purpose |
|---|---|---|---|
| Logon | A | both | Session start; `Username` (553), `Password` (554) |
| NewOrderSingle | D | in | Place order |
| OrderCancelRequest | F | in | Cancel |
| OrderCancelReplaceRequest | G | in | Replace |
| ExecutionReport | 8 | out | Order state and fills |
| OrderCancelReject | 9 | out | Cancel/replace rejected |
| SecurityListRequest / SecurityList | x / y | in / out | Contract reference data |
| SecurityStatus | f | out | Halt, resume, expiry, settlement |
| TradeCaptureReport | AE | out | Drop copy to clearing members |
| QuoteRequest / Quote / QuoteResponse / QuoteCancel | R / S / AJ / Z | both | RFQ (OTF contracts) |
| MarketDataRequest / Snapshot / Incremental | V / W / X | market data session | See [Market data](../market-data.md) |
| MarginReport (custom, UM1) | UM1 | out | Account margin budget and usage |

## NewOrderSingle (D)

| Tag | Field | Req | Notes |
|---|---|---|---|
| 11 | ClOrdID | Y | Unique per account among open orders |
| 55 | Symbol | Y* | Or 48/22 with 22=4 (ISIN) |
| 54 | Side | Y | 1 Buy, 2 Sell (yes-space) |
| 38 | OrderQty | Y | Contracts, integer |
| 40 | OrdType | Y | 2 (Limit) only |
| 44 | Price | Y | Decimal, multiple of 0.005, in [0.005, 0.995] |
| 59 | TimeInForce | N | 0 Day (default), 1 GTC, 3 IOC, 4 FOK, 6 GTD |
| 126 | ExpireTime | C | Required for GTD |
| 1 | Account | Y | SWANS account ID |
| 528 | OrderCapacity | Y | A agency, P principal, R riskless principal |
| 453 | NoPartyIDs | Y | Parties group; see below |
| 7928 | SelfMatchPreventionID | N | Same ID on both orders triggers STP |
| 20020 | PostOnly | N | Y/N |
| 20021 | ReduceOnly | N | Y/N |
| 20050 | InvestmentDecisionWithinFirm | Y | RTS 24 |
| 20051 | ExecutionWithinFirm | Y | RTS 24 |
| 20052 | AlgoIndicator | Y | Y/N |
| 60 | TransactTime | Y | UTC, microseconds |

Parties group (453): `PartyID` (448), `PartyIDSource` (447), `PartyRole` (452). Required roles: 1 ExecutingFirm, 4 ClearingFirm, 24 CustomerAccount (CCP account reference), 12 ExecutingTrader.

## ExecutionReport (8)

| Tag | Field | Notes |
|---|---|---|
| 37 | OrderID | Venue order ID |
| 11 / 41 | ClOrdID / OrigClOrdID | |
| 17 | ExecID | Unique per report |
| 150 | ExecType | 0 New, 4 Cancelled, 5 Replaced, 8 Rejected, C Expired, F Trade, H Trade Cancel |
| 39 | OrdStatus | 0 New, 1 Partially filled, 2 Filled, 4 Cancelled, 8 Rejected, C Expired |
| 31 / 32 | LastPx / LastQty | On fills |
| 151 / 14 / 6 | LeavesQty / CumQty / AvgPx | |
| 880 | TrdMatchID | Trade ID, on fills |
| 20011 | TradeVenueTxID | MiFIR TVTIC, on fills |
| 20010 | Seq | Global sequence number |
| 20001 | SwansRejectCode | On rejections; see [Error codes](error-codes.md) |
| 103 / 58 | OrdRejReason / Text | |
| 60 | TransactTime | Engine time |

## OrderCancelReplaceRequest (G)

Same fields as D plus `OrigClOrdID` (41). Price change or quantity increase loses time priority. If the original is already fully filled, the request is rejected with `OrderCancelReject` (9), `CxlRejReason` (102) = 1.

## SecurityList (y)

One entry per contract with: 55, 48, 22=4, 167 SecurityType **[confirm]**, 541 MaturityDate, 15 Currency, 231 ContractMultiplier=100, 969 MinPriceIncrement=0.005, 20002 SwansSchema, 20003 SwansCatalystId, 20004 SettlementKind, 20005 LastTradingTime, 20006 SettlementDeadline, 20007 TransparencyMode, 20008 PackageLegs (repeating group for packages), 20009 HedgeMap (repeating group).

## SecurityStatus (f)

`SecurityTradingStatus` (326): 2 Trading halt, 17 Ready to trade, 18 Not available for trading, 20 Unknown or invalid; custom 20030 SettlementValue and 20031 SettlementStatus on settlement events.

## TradeCaptureReport (AE), drop copy

`TradeReportID` (571), `TradeReportTransType` (487), `TrdType` (828), 55, 32, 31, 75 TradeDate, 60, 880, 20011, 20040 ClearingState (matched, sent, accepted, rejected, cleared, settled, busted); `NoSides` (552) group with 54, 37, 11, 1, 453.

## MarginReport (UM1), custom

Sent on request (`MarginReportRequest`, UM0) and on every change to the account's budget: 1 Account, 20060 MarginBudget, 20061 MarginUsed, 20062 MarginAvailable, 20063 BudgetVersion, 20064 AsOf.

## RFQ

`QuoteRequest` (R) with 131 QuoteReqID, 55, 38, 54 (optional, two-way if absent); `Quote` (S) from liquidity providers with 117 QuoteID, 132/133 BidPx/OfferPx, 134/135 sizes, 62 ValidUntilTime; `QuoteResponse` (AJ) 694 QuoteRespType 1 (hit/lift) from the requester; fills reported as `ExecutionReport`.

## Custom tags

| Tag | Name | Type |
|---|---|---|
| 7928 | SelfMatchPreventionID | int |
| 20001 | SwansRejectCode | int |
| 20002–20009 | Contract fields (schema, catalyst, settlement kind, last trading time, settlement deadline, transparency mode, package legs, hedge map) | various |
| 20010 | Seq | int |
| 20011 | TradeVenueTxID | string |
| 20020 / 20021 | PostOnly / ReduceOnly | bool |
| 20030 / 20031 | SettlementValue / SettlementStatus | decimal / int |
| 20040 | ClearingState | int |
| 20050–20052 | RTS 24 fields | string / bool |
| 20060–20064 | Margin report fields | decimal / int / UTC |
| 20070–20078 | Fee attribution on fills: base, certainty pressure, event window, liquidity stress, concentration, contrarian discount, maker rebate, unwind relief (bps), floor applied (Y/N) | decimal / bool |
