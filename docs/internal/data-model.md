# Data model

```cpp
namespace swans {
using InstrumentId = uint32_t;  using MemberId = uint16_t;  using AccountId = uint32_t;
using ClearerId = uint16_t;     using OrderId = uint64_t;   using TradeId = uint64_t;
using SeqNum = uint64_t;        using Timestamp = int64_t;  using PriceTicks = int16_t;  using Qty = int32_t;

enum class SchemaType : uint8_t { S01=1,S02,S03,S04,S05,S06,S07,S08,S09,S10 };
enum class SettlementKind : uint8_t { Binary, Measured };
enum class Currency : uint8_t { GBP, USD, EUR };
enum class InstrumentState : uint8_t { Pending, Listed, Trading, Halted, Expired, Settled, Cancelled };
enum class TransparencyMode : uint8_t { Realtime, Delayed, TopOnly };
enum class FamilyType : uint8_t { Standalone, Nested, MutuallyExclusive, Cluster };
struct Family { uint32_t id; FamilyType type; InstrumentId members[16]; uint8_t n; uint8_t order[16]; uint16_t product_group; };

struct SettlementSource { char code[8]; char uri[256]; uint8_t tier; };
struct HedgeRef { uint8_t type; char identifier[24]; char venue[8]; };   // type: SingleStockOption, Equity, ETF, IndexFuture, RatesFuture

struct ContractSpec {
    InstrumentId id; char symbol[24]; char isin[12]; char cfi[6]; char fisn[35];
    SchemaType schema; SettlementKind settlement_kind; Currency ccy;
    int32_t payout_minor;            // 10000 = 100.00
    PriceTicks tick; Qty min_qty; Qty lot; Qty max_order_qty; Qty position_limit;
    char issuer_lei[20]; char issuer_ticker[12]; char catalyst_id[32]; char question[512];
    SettlementSource source;
    Timestamp listing_time, last_trading_time, expected_resolution, settlement_deadline;
    uint8_t fallback_rule;
    uint32_t family_id; uint16_t product_group;                // family for admissible-state margin and mark projection
    int32_t strike_low_x1e6, strike_high_x1e6;                // measured contracts: linear settlement between strikes
    InstrumentId package_legs[4]; int8_t package_ratios[4];   // packages: own bounded-payoff product
    HedgeRef hedges[4];
    TransparencyMode transparency; InstrumentState state; uint32_t spec_version;
    bool rfq_only;
};

enum class MemberType : uint8_t { Fund, MarketMaker, Dealer, Broker };
enum class ClearingMode : uint8_t { PrimeBroker, SelfClearing };
struct Member  { MemberId id; char lei[20]; char name[64]; MemberType type; bool professional_attested; bool enabled; };
struct Account { AccountId id; MemberId member; ClearerId pb; ClearingMode mode; char pb_account_ref[32]; bool enabled; };

struct PBLimits {                    // set by the prime broker via the PB API
    ClearerId pb; AccountId account;         // account 0 = all accounts of this PB
    int64_t max_gross_notional_minor, max_net_notional_minor, max_daily_loss_minor, max_swans_im_minor;
    Qty max_order_qty; bool kill; uint32_t version;
};

enum class Side : uint8_t { Buy=1, Sell=2 };
enum class TimeInForce : uint8_t { Day=0, GTC=1, IOC=3, FOK=4, GTD=6 };
enum class OrdStatus : uint8_t { New, PartiallyFilled, Filled, Cancelled, Rejected, Expired };
struct Order {
    OrderId id; char cl_ord_id[20]; InstrumentId instrument; AccountId account; Side side;
    TimeInForce tif; PriceTicks price; Qty qty, leaves, cum; Timestamp expire_time, received, accepted;
    OrdStatus status; uint8_t capacity; uint32_t smp_id; bool post_only, reduce_only;
    char inv_decision_id[16]; char exec_within_firm_id[16]; bool algo;    // RTS 24
};

enum class TradeState : uint8_t { Matched, NotifiedPB, PBAccepted, Settled, Busted };
struct Trade {
    TradeId id; InstrumentId instrument; PriceTicks price; Qty qty;
    OrderId buy_order, sell_order; AccountId buy_account, sell_account; ClearerId buy_pb, sell_pb;
    Timestamp matched, notified_pb, pb_response; TradeState state; char pb_trade_ref[32];
    uint8_t aggressor_side; char tvtic[52];
};

struct Position {                     // per (account, instrument); packages kept as packages here
    AccountId account; InstrumentId instrument; Qty net_qty; Qty pending_qty;   // pending = matched, not yet PB-acknowledged
    int64_t avg_price_ticks_x1e6, realised_pnl_minor, vm_cumulative_minor; Timestamp as_of;
};

enum class SettlementStatus : uint8_t { Proposed, Determined, Disputed, Final, Fallback };
struct SettlementEvent {
    InstrumentId id; int32_t value_x1e6; SettlementStatus status;
    Timestamp source_publish_time, determination_time, final_time;
    char evidence_uri[256]; uint8_t evidence_sha256[32]; char determiner_id[16]; char confirmer_id[16];
};
}
```

Persisted in PostgreSQL (versioned rows, `valid_from/valid_to`, nothing deleted) for reference data and projections; journals for the hot path (see [Messaging](messaging.md)).
