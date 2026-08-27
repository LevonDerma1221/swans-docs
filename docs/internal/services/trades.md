# Trades service

**Owns:** the legal record of every trade, its clearing state machine, STP to the CCP, drop copies, busts.

**State machine:** `Matched → SentToCCP → {CCPAccepted → Cleared | CCPRejected}`; `Matched → Bilateral` for bilateral accounts; `Cleared|Bilateral → Settled`; any → `Busted` (operator, with reason).

**STP:** on `TradeExecuted`, build and send the CCP submission within the venue window; record accept/reject. Thresholds **[confirm RTS 26]**. On reject: `ExecutionReport` ExecType H to both sides, trade voided, positions reverted (pending → 0).

```cpp
class ICcpAdapter {
public:
    virtual ~ICcpAdapter() = default;
    virtual void submit(const Trade&, std::function<void(CcpResponse)>) = 0;
    virtual void publishProductSpecs(const std::vector<ContractSpec>&) = 0;
    virtual void publishSettlement(const SettlementEvent&) = 0;
    virtual void publishDailySettle(InstrumentId, PriceTicks) = 0;           // if SWANS sets it
    virtual CcpMarginReport fetchEndOfDayMargin(Date) = 0;                    // reconciliation
};
struct CcpResponse { TradeId id; bool accepted; char ccp_ref[32]; uint16_t reject_code; Timestamp ts; };
```
Phase 1 implements `CcpSimulator` (configurable latency/failure). Phase 4 implements the chosen CCP's spec.

**Bilateral adapter:** `IGiveUpAdapter` with the same shape, targeting the prime broker's give-up confirmation flow.

**Drop copy:** `TradeCaptureReport` per clearing-state change; EOD CSV per clearer and per member.

**Storage:** append-only journal + PostgreSQL projection.
