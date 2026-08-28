# Guarantees and non-guarantees

What SWANS enforces, and what it does not. Members should design to this list, not to assumptions.

## SWANS guarantees

1. **Deterministic matching.** Every order, cancel and replace for a contract is processed strictly in arrival sequence; the same input sequence always produces the same book and fills. Priority is price, then time.
2. **Sequenced state.** Every state change carries a global `seq`. If two events carry `seq_a < seq_b`, event a happened first, on any channel.
3. **Acknowledgement before consequence.** No fill is published before the order that produced it was accepted; no cancel is confirmed before it took effect in the book.
4. **Pre-trade limits bind.** An order that would breach a PB limit, a position limit, a price band or the margin budget is rejected before it reaches the book, with a reason code.
5. **Trade notification.** Every matched trade is notified to both sides' PBs via the PB adapter, with margin schedule updates.
6. **Settlement per source.** Contracts settle only on the named source's publication, under the schema's rules, through a two-officer determination with a dispute window. The evidence is retained and available to members.
7. **Equal access.** Rules, fees, limits and data are applied identically to all members in the same category. Fee components are published formulas.
8. **Record and report.** Every order event and message is journaled with nanosecond timestamps and retained for the regulatory period; transactions are reported to the FCA.
9. **Kill within one second.** A PB's kill instruction rejects new orders and cancels resting orders for its clients within one second of receipt.

## SWANS does not guarantee

1. **Liquidity.** A book may be empty. Liquidity-provider schemes impose obligations on designated providers, not on the venue.
2. **Margin levels.** SWANS publishes its margin numbers as calculation agent. What you post is set by your PB under the CSA, and may exceed SWANS's figure.
3. **Settlement timing.** SWANS targets settlement within one business day of source publication, but the source and the dispute window govern the actual date.
4. **Marks as prices.** Filtered fair marks are the official valuation input; they are not executable quotes and may differ from the last trade.
5. **Uptime beyond the published service levels.** Planned maintenance windows and incident procedures are published; the business-continuity objective is resumption within two hours of a declared disruption **[confirm]**.
6. **That a contract will exist to expiry.** Under the rulebook a contract may be voided if its catalyst becomes undeterminable before any trade, or fallback-settled at the deadline.
7. **Message ordering across channels.** Within a FIX session, execution reports are in engine order. Across FIX and WebSocket, use `seq`; SWANS does not guarantee which channel delivers first.
8. **Netting at your prime broker.** This is the PB's decision under its portfolio-margin policy, not promised by SWANS.
