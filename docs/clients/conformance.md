# Conformance

Every member certifies each interface before production access. The suite runs against the certification environment and is scripted; a SWANS engineer reviews the log.

## Order entry (FIX)

1. Logon, heartbeat, test request, resend request, sequence reset.
2. New order: accepted, rested, filled, partially filled.
3. Cancel and cancel/replace: accepted, rejected (unknown order, already filled).
4. IOC and FOK behaviour.
5. Post-only rejection on cross; reduce-only enforcement.
6. Self-trade prevention.
7. Rejections: price band, quantity limit, position limit, clearing-member limit, margin budget, kill switch, contract halted, outside hours. Verify tag 20001 codes.
8. Cancel on disconnect.
9. Trade cancel (ExecType H) after CCP reject.
10. RTS 24 fields present on every order.

## Drop copy (clearing members)

1. `TradeCaptureReport` received for each client fill with parties, account and clearing references.
2. End-of-day trade and position files reconcile to drop copies.

## Clearer API (clearing members)

1. Set and read limits; verify enforcement pre-trade.
2. Kill switch: new orders rejected, resting orders cancelled, within 1 second.
3. Margin budget: verify rejection at the boundary.

## Market data

1. Snapshot then incremental, with sequence continuity.
2. Recovery after a forced gap.
3. Status messages: halt, resume, expiry, settlement.

## RFQ (if applicable)

1. QuoteRequest fan-out, Quote, QuoteResponse, expiry of the window.
