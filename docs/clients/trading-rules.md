# Trading rules

## Order types and time in force

| Order type | Notes |
|---|---|
| Limit | Rests at specified price or better |
| Market | Fills at best available price; unfilled remainder cancelled (behaves as IOC with no price limit) |

| TIF | Behaviour |
|---|---|
| `DAY` | Expires at end of trading day (00:00 UTC) |
| `GTC` | Rests until filled, cancelled, or the contract expires |
| `GTD` | Rests until `ExpireTime` (126) |
| `IOC` | Fills what it can immediately; remainder cancelled |
| `FOK` | Fills entirely or is rejected |

Flags: `post_only` (rejected if it would cross), `reduce_only` (may only reduce the account's net position).

## Matching

Continuous trading, price-time priority. Orders match at the resting order's price. Within a price level, earlier orders fill first. A cancel/replace that changes price or increases quantity loses priority; a quantity decrease keeps it.

Processing is strictly sequential per contract: every order, cancel and replace is assigned a sequence number on arrival and processed in that order. There is no batching.

## Self-trade prevention

If an incoming order would match a resting order carrying the same self-match prevention ID (tag 7928) in the same account group, the resting order is cancelled and the incoming order continues to match. Optional per session: cancel the incoming order instead.

## Tick size

Fixed at 0.01 across the whole range. Price bounds are 0.01 and 0.99 (99 price levels). Contract size is 100 (like NASDAQ event contracts).

## Price bands and volatility controls

- **Static band:** orders more than 20 ticks from the reference price are rejected. Per-contract override possible.
- **Dynamic halt:** if the last trade moves more than 15 ticks from the price 60 seconds earlier, the contract enters a 2-minute halt, then reopens in continuous trading. Parameters per schema **[confirm with FCA expectations under RTS 7]**.
- **Operator halt:** SWANS may halt a contract on a settlement-source event, surveillance alert or market disorder.

## Pre-trade risk

Every order passes, in this order:

1. Member, account and contract enabled; before `last_trading_time`.
2. Quantity within limits; price within bounds.
3. Price band.
4. Position limit (net absolute position after this order within `position_limit`).
5. Balance check: `available >= max_loss` where max loss = price × contract size × qty (buy) or (1 − price) × contract size × qty (sell). Reject with `INSUFFICIENT_BALANCE` if insufficient.
6. Post-only and reduce-only semantics.

Rejections carry a reason code in FIX tag 20001.

## Trading hours

**24/7 continuous trading.** The matching engine runs without interruption. There is no opening or closing session and no weekend halt.

- Individual contracts stop trading at their `last_trading_time`, which is set before the earliest possible publication of the settlement source.
- The reference price is the filtered fair mark.

## RFQ (OTF mode)

If SWANS operates as an OTF for some contracts, those contracts trade by request-for-quote: a member sends a QuoteRequest, designated liquidity providers respond within the RFQ window (default 5 seconds), the requester executes against a quote. Discretion is exercised by SWANS staff under the rulebook, and every step is journaled. RFQ contracts have no public order book. **[decision pending: MTF vs OTF]**

## Position limits and reporting

Per-account net position limits per contract are set in reference data and enforced pre-trade. Members must report positions held for clients on request. Large-position reporting thresholds **[confirm]**.

## Erroneous trades

A member may request review of a trade within 15 minutes of execution. SWANS may bust or adjust trades that are clearly erroneous under the rulebook. Busted trades are reversed and positions restored; PBs are notified.
