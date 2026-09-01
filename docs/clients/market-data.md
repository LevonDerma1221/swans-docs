# Market data

## Content

- Top of book and 5 levels of depth, aggregated by price level.
- Last trade (price, size, aggressor side), volume, open interest.
- Reference price (most recent VM window mark). Filtered fair mark at each VM window.
- Contract status changes (halt, resume, expiry, settlement).

## Channels

| Channel | Transport | Notes |
|---|---|---|
| FIX market data | FIX 4.4 on a separate session | Snapshot and incremental |
| WebSocket | JSON; book, trades, status channels | Real-time |
| REST | Snapshots | On demand |

## Sequencing

Every update carries the global `seq` and a per-channel sequence. A gap means a missed message: resubscribe or request a snapshot.

## Transparency

Pre-trade transparency for illiquid derivatives may be subject to waivers; a contract's `transparency_mode` states whether depth is published in real time, delayed, or top-of-book only **[confirm]**.
