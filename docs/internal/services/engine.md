# Matching engine

Continuous limit order book, price-time priority, one book per instrument, instruments sharded by `id % N`, single-threaded per shard, no locks in the hot path.

## Book
```cpp
struct RestingOrder { OrderId id; AccountId acct; Qty leaves; Timestamp ts; uint32_t smp_id; bool reduce_only; RestingOrder* next; };
struct PriceLevel  { Qty total; RestingOrder* head; RestingOrder* tail; };
struct Book {
    InstrumentId id;
    std::array<PriceLevel,99> bids, asks;        // index = price ticks (0.01 to 0.99)
    PriceTicks best_bid = 0, best_ask = 99;
    uint64_t bid_bitmap[4], ask_bitmap[4];       // non-empty levels, for O(1) next-level scan
    OpenAddressingMap<OrderId, RestingOrder*> by_id;
    ObjectPool<RestingOrder> pool;
};
```

## Algorithm (buy)
```
if FOK and available_at_or_below(price) < qty: reject FOK_UNFILLABLE
if post_only and best_ask <= price: reject POST_ONLY_CROSS
while leaves > 0 and best_ask <= price:
    level = asks[best_ask]
    while leaves > 0 and level.head:
        r = level.head
        if r.smp_id == o.smp_id and same_member: cancel r (or cancel o per session mode); continue
        fill = min(leaves, r.leaves)
        emit Trade(price=best_ask, qty=fill, aggressor=Buy)
        leaves -= fill; r.leaves -= fill; level.total -= fill
        if r.leaves == 0: pop r
    if level.total == 0: advance best_ask via bitmap
if leaves > 0: IOC/FOK → cancel remainder; else rest at bids[price]
```
Replace: price change or qty increase loses priority; qty decrease keeps it.

## Determinism
Every input to a shard gets a `SeqNum` and is journaled before processing; state = f(journal). Outputs carry the input `SeqNum`. Budget/limit versions used by pre-trade are journaled with the input.

## Lifecycle
`Listed → Trading` at `listing_time`; `Trading ↔ Halted` on volatility/operator/source events; `Trading → Expired` at `last_trading_time` (resting orders expired, ExecType C); `Expired → Settled` on final settlement.

## RFQ (OTF contracts)
Not an engine mode. A separate **RFQ workflow service** fans out `QuoteRequest`, collects `Quote`s for the window, applies the requester's `QuoteResponse`, and, where discretion applies, records the operator's action. The resulting trade enters the same trades path.

## Outputs
`OrderAccepted, OrderRejected, OrderCancelled, OrderReplaced, OrderExpired, TradeExecuted, BookUpdate`, each with `SeqNum`, engine ts, instrument.

## Targets
Ingress → `TradeExecuted`: p50 ≤ 5 µs, p99 ≤ 25 µs; ≥ 200k orders/s per shard.
