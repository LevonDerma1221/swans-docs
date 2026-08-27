# Market data

## Content

- Top of book and 5 levels of depth, aggregated by price level.
- Last trade (price, size, aggressor side), daily volume, open interest (end of day).
- Reference price (last daily settlement, or last trade if more recent) and daily settlement price (filtered fair mark at 17:00).
- Contract status changes (halt, resume, expiry, settlement).

## Channels

| Channel | Transport | Notes |
|---|---|---|
| FIX market data | FIX 4.4 `V`/`W`/`X` on a separate session | Snapshot and incremental |
| WebSocket | `wss://…/v1/ws` | JSON; book, trades, status, settlement channels |
| REST | `GET /v1/book/{symbol}` etc. | Snapshots |
| Multicast (production, later) | SBE over UDP, TCP recovery | For co-located members |

## Sequencing

Every update carries the global `seq` and a per-channel sequence. A gap in the per-channel sequence means a missed message: resubscribe or request a snapshot.

## Transparency

Pre-trade transparency for illiquid derivatives may be subject to waivers; a contract's `transparency_mode` in reference data states whether depth is published in real time, delayed, or top-of-book only **[confirm]**. Post-trade publication follows the deferral regime applicable to the instrument.

## Entitlements

Market data is entitled per member and per level (top of book, depth). Redistribution requires a licence.
