# Capacity and performance

**Working assumptions [confirm with Lyes and Levon]:** year one, 40 trading members, 12 liquidity providers, 300 live contracts, 250,000 order messages per day, peak 3,000 messages per second (quote updates around macro releases), 5,000 trades per day, 2,000 accounts. Year three, 5× each.

## Order path
| Metric | Target | Measured by |
|---|---|---|
| Gateway ingress to engine ack (p50 / p99) | 30 µs / 120 µs | Timestamps at gateway ingress and engine output, same host clock |
| Engine matching per order (p50 / p99) | 5 µs / 25 µs | Engine internal |
| Pre-trade stage incl. fast margin bound (p99) | 50 µs | Engine internal |
| Precise incremental IM call (p99) | 500 µs | Margin engine RPC; rare |
| End-to-end order to ExecutionReport (p99, cross-connect) | 250 µs | Member-side capture in certification |
| Sustained throughput per shard | 200,000 msg/s | Load test |
| Shards at launch | 4 | |

## Market data
Top-of-book update to publish (p99) 50 µs; snapshot every 5 s or on request; WebSocket fan-out latency (p99) 5 ms to 500 subscribers.

## STP
Trade matched to collateral lock (p99) 500 ms. Future: trade matched to PB notification when margin mode is offered.

## Margin engine compute budget
V6 is Monte Carlo per account. Daily run: 2,000 accounts × 20,000 scenarios × (positions per account ≤ 50) with vectorised evaluation on a 64-core host: target under 10 minutes; intraday triggered runs on the affected accounts only: under 30 seconds per account batch. Pre-trade uses the fast bound and a cached incremental from the last run; the precise incremental call uses a reduced scenario set (2,000) and a 500 µs budget. Scenario sets, hazard tables and factor loadings are precomputed at each recalibration (every 60 s or on trigger) and shared across accounts.

## Storage
Journals: ~1 KB per input; 250,000/day → ~250 MB/day; 7-year retention → ~700 GB per shard group before compression. PostgreSQL projections sized at 10× journal for indexes and history.

## Load and soak tests
Nightly load test at 3× peak on certification; quarterly 8-hour soak at 1.5× peak with failover injected; results retained for the FCA.
