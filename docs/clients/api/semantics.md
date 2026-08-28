# Order and event semantics

Exact behaviour that integrators depend on.

## Identifiers

- `ClOrdID` (11) must be unique among the account's open orders. Reusing a `ClOrdID` of a closed order is permitted but discouraged; reusing one of an open order is rejected (`OrderCancelReject` 102=0 on replace/cancel, `ExecutionReport` 8 with 20001=1006 `DUPLICATE_CLORDID` on new order).
- `OrderID` (37) is assigned by the venue at acceptance and is stable through replaces.
- `TrdMatchID` (880) identifies a trade; both sides carry the same value. `ExecID` (17) is unique per execution report.

## Lifecycle rules

| Situation | Behaviour |
|---|---|
| Replace on a partially filled order | Applies to `LeavesQty`. New quantity below `CumQty` is treated as cancel. Price change or quantity increase resets time priority; quantity decrease keeps it. |
| Replace arrives after the order fully filled | `OrderCancelReject` with 102=1 (unknown order) and 434=2. |
| Cancel and fill race | Whichever the engine sequences first wins. A cancel that arrives after a fill for the same order cancels only the remaining `LeavesQty`. |
| PB rejects a trade | Both sides receive `ExecutionReport` with `ExecType=H`, `LastQty` equal to the rejected quantity, `CumQty` and `AvgPx` restated. `LeavesQty` is unchanged: the order does not return to the book. |
| Bust | As PB reject, with `ExecRestatementReason` (378) = 8 and `Text` carrying the bust reference. |
| Contract halted | Resting orders remain. New orders rejected with 1003. On resume, continuous trading restarts without an auction unless the rulebook specifies one for the halt type. |
| Contract expires (`last_trading_time`) | All resting orders expired (`ExecType=C`). |
| Session disconnect | If `CancelOnDisconnect` is on, resting orders for that session are cancelled within one second, reported on reconnect. |
| PB kill | New orders rejected with 1010; resting orders cancelled with `ExecType=4`, `Text=KILL`. |
| Post-only would cross | Rejected with 1080. |
| Reduce-only would increase exposure | Quantity is reduced to the amount that does not increase exposure; if that is zero, rejected with 1081. |
| Self-match | Default: resting order cancelled (`ExecType=4`, `Text=STP`), incoming continues. Session option: cancel incoming (1070). |

## Sequence numbers

- `seq` (tag 20010 on FIX; `seq` in REST envelopes and WebSocket messages) is global and strictly increasing across all contracts and channels.
- If an order response carries `seq = o` and a book update carries `seq = w`: `w < o` predates the order; `w = o` is the same state transition; `w > o` follows it.
- Per-channel sequences (`channel_seq`) are contiguous within a channel; global `seq` on a single channel may skip values.
- REST snapshots carry the `seq` as of which they are consistent; apply WebSocket updates with `seq` greater than the snapshot's.

## Cross-channel ordering

Execution reports on a FIX session are delivered in engine order. WebSocket `account` updates for the same events may arrive before or after the FIX report. Reconcile by `seq` and `OrderID`, never by arrival time.

## Fills and fees

Every fill carries `fee_components` (REST/WS) and, on FIX, tags 20070–20078 (base, certainty pressure, event window, liquidity stress, concentration, contrarian discount, maker rebate, unwind relief, floor applied). Fees are final at fill; they are not restated on PB reject (the fee is reversed in the fee file with reference to the voided trade).

## Positions and margin budget

| Event | Position effect | Margin budget effect |
|---|---|---|
| Order accepted | none | `used` includes the order's incremental IM at limit price (open-order reservation) |
| Order cancelled/expired | none | reservation released |
| Fill (matched) | `pending_qty` updated | reservation converts to position IM (next recalculation) |
| PB accepted | `pending_qty` → `net_qty` | none |
| PB rejected / bust | `pending_qty` removed | IM recomputed, budget restored |
| Daily/intraday margin run | none | `im`, `used`, `available` and `version` restated; pushed on the `account` channel and via `MarginReport` |
| PB changes `max_swans_im` | none | `budget` restated; if `available` goes negative, new risk-increasing orders are rejected until it is positive |

## Time in force

`DAY` expires at 17:30 London. `GTC` survives until filled, cancelled, contract expiry or 90 calendar days, whichever is first. `GTD` requires `ExpireTime` in UTC, not beyond `last_trading_time`.
