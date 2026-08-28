# Settlement

SWANS is the settlement authority for its contracts. Final settlement is determined against the contract's named source under its schema policy, published, and delivered to the CCP for cash settlement.

## Determination process

1. **Proposed.** A settlement officer records the source publication (canonical URI, retrieval timestamp, SHA-256 of the retrieved document) and the proposed settlement value against the contract's question.
2. **Determined.** A second, independent officer confirms. Two distinct officers are required; the system enforces it.
3. **Dispute window.** Members may dispute within the schema's window (default 24 hours) by submitting evidence. Disputes go to the settlement committee, whose decision is final under the rulebook.
4. **Final.** Settlement is published (REST, WebSocket `settlement` channel, FIX `SecurityStatus`) and delivered to the PB. Positions cash-settle at `value × payout` through the last variation margin.
5. **Fallback.** If the settlement deadline passes without a determinable outcome, the schema's fallback rule applies: settle at 0, settle at the last daily settlement price, or void, as specified in reference data at listing. Fallback settlements are flagged as such.

## Sources

Each schema defines a source hierarchy. Tier 1 is the official primary source (regulator, court, company filing, statistics office, central bank). Secondary sources are used only where the schema policy allows and are flagged. Source documents are retained for the record-keeping period.

## Measured contracts

For measured contracts, the published figure is mapped to [0, 1] by the contract's defined range and rounding rule, then treated like a binary settlement value.

## Timing

Contracts stop trading at `last_trading_time`, set before the earliest publication window of the source. Settlement is targeted within one business day of source publication, subject to the dispute window.

## Corporate events and restatements

Restatements after final settlement do not reopen a settlement. Corporate events affecting single-name contracts before settlement (delisting, acquisition, ticker change) are handled under the schema policy, which may adjust the question or void the contract.
