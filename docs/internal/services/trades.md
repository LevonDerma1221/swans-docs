# Trades service

**Owns:** the legal record of every trade, its clearing state machine, PB notification, drop copies, busts.

**State machine:** `Matched → NotifiedPB → PBAccepted → Settled`; any → `Busted` (operator, with reason).

**PB notification:** on `TradeExecuted`, send trade details to both sides' PBs via the PB adapter; deliver updated margin schedule. On PB reject (rare — limit breach): `ExecutionReport` ExecType H to both sides, trade voided, positions reverted.

```cpp
class IPBAdapter {
public:
    virtual ~IPBAdapter() = default;
    virtual void notifyTrade(const Trade&, std::function<void(PBResponse)>) = 0;
    virtual void publishMarginSchedule(const MarginSchedule&) = 0;
    virtual void publishSettlement(const SettlementEvent&) = 0;
    virtual void publishDailySettle(InstrumentId, PriceTicks) = 0;
    virtual PBPositionReport fetchEndOfDayPositions(Date) = 0;                   // reconciliation
};
struct PBResponse { TradeId id; bool accepted; char pb_ref[32]; uint16_t reject_code; Timestamp ts; };
```
Phase 1 implements `PBSimulator` (configurable latency/failure). Production implements each PB's specific FIX/API spec.

**Drop copy:** `TradeCaptureReport` per state change; EOD CSV per PB and per member.

**Storage:** append-only journal + PostgreSQL projection.
