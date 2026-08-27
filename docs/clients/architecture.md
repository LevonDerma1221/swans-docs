# Architecture

SWANS is a multilateral trading facility. It matches orders between members; it is never a counterparty to a trade, never holds member funds, and never guarantees a trade. Guarantee and margin are provided by the partner central counterparty (CCP) and the clearing members.

```
 Member OMS ──FIX──▶ SWANS gateway ──▶ pre-trade risk ──▶ matching engine ──▶ market data ──▶ members
                                                                │
                                                                ├──▶ straight-through to CCP (seconds)
                                                                │        CCP novates: faces each side's clearing member
                                                                └──▶ drop copy to clearing members
```

See the [end-to-end views](../diagrams.md) for the full system map, the trade lifecycle and the margin flows.

**What SWANS does**

- Operates the order book, matches trades, publishes market data.
- Runs pre-trade risk controls on every order: clearing-member limits and a SWANS margin budget per account.
- Sends every matched trade to the CCP within the straight-through-processing window; on rejection the trade is voided and both parties are notified.
- Acts as settlement authority: determines and publishes the outcome of each contract against its documented source.
- Computes and publishes risk and margin numbers: the CCP parameter file, the clearing-member margin schedule, and a daily price and risk file for prime brokers.
- Reports orders and transactions to the FCA and runs market surveillance.

**What SWANS does not do**

- Hold collateral. Collateral sits with clearing members and the CCP.
- Set clearing margin. The CCP does, under its model with SWANS parameters. Clearing members may charge more.
- Guarantee settlement. The CCP does.

**Sequence numbers.** Every state change on the venue carries a global, monotonically increasing `seq`. It appears in FIX execution reports (tag 20010), REST envelopes and WebSocket messages, so a client can order events across channels.

**Time.** All timestamps are UTC. FIX `TransactTime` (60) and `SendingTime` (52) are microsecond precision; REST and WebSocket timestamps are nanoseconds since the Unix epoch, serialised as strings.
