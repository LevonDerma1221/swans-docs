# Trades service

**Owns:** the legal record of every trade, its state machine, drop copies, busts.

**State machine (launch — full collateral):** `Matched → Settled`; any → `Busted` (operator, with reason). Trades are final on match — collateral is already locked.

**State machine (future — when PB/margin offered):** `Matched → NotifiedPB → PBAccepted → Settled`; any → `Busted`. On PB reject: `ExecutionReport` ExecType H to both sides, trade voided, positions reverted.

```cpp
class IPBAdapter {
public:
    virtual ~IPBAdapter() = default;
    virtual void notifyTrade(const Trade&, std::function<void(PBResponse)>) = 0;
    virtual void publishMarginSchedule(const MarginSchedule&) = 0;
    virtual void publishSettlement(const SettlementEvent&) = 0;
};
```
Phase 1 implements `PBSimulator` (configurable latency/failure) for testing the future PB flow.

**Drop copy:** `TradeCaptureReport` per state change; files per member.

**Storage:** append-only journal + PostgreSQL projection.
