# Collateral service

**Owns:** member balances, collateral locks, VM transfers, settlement payouts, withdrawals. This is the full-collateral mode — an alternative to PB-held collateral where SWANS manages funds directly via a regulated custodian or FCA CASS client money account.

## Modes

The system supports two collateral modes per account, configured in reference data:

| Mode | Who holds cash | Pre-trade check | Collateral service role |
|---|---|---|---|
| `PBManaged` | Prime broker | Margin budget (from PB) | None — PB handles everything |
| `FullCollateral` | SWANS (via custodian/CASS account) | Balance check: sufficient funds to cover max loss | Active — manages all flows |

Both modes can run simultaneously. Some accounts use a PB; others are fully collateralised.

## Data model

```cpp
struct CollateralAccount {
    AccountId account;
    int64_t balance_minor;           // total deposited
    int64_t locked_minor;            // locked as collateral for open positions and orders
    int64_t available_minor;         // balance - locked (what can be withdrawn or used for new trades)
    int64_t vm_cumulative_minor;     // net VM transferred to/from this account
    Currency ccy;
    Timestamp as_of;
};

struct CollateralLock {
    AccountId account;
    TradeId trade;                   // or OrderId for order reservations
    InstrumentId instrument;
    int64_t amount_minor;            // max loss for this side of the trade
    Timestamp locked_at;
};

struct CollateralTransfer {
    uint64_t transfer_id;
    AccountId from_account;
    AccountId to_account;            // or SETTLEMENT for payouts
    int64_t amount_minor;
    enum Reason : uint8_t { VM, Settlement, Deposit, Withdrawal, FeePayment };
    Timestamp ts;
};
```

## Pre-trade balance check

For `FullCollateral` accounts, the engine's pre-trade stage adds a check before the margin budget check:

```
max_loss = (side == Buy) ? price * payout * qty    // buy: lose if settles at 0
                         : (1 - price) * payout * qty  // sell: lose if settles at 1

if (account.available < max_loss) reject with INSUFFICIENT_BALANCE
```

This runs inside the engine shard (no RPC) using a cached balance snapshot. The collateral service pushes balance updates on the `collateral.balances` stream; the engine caches the latest version.

## On trade

1. Lock `max_loss` from buyer's available balance.
2. Lock `max_loss` from seller's available balance.
3. Total locked per trade = buyer max loss + seller max loss = payout * qty (always exactly the full payout).

## Variation margin (rolling, every 8 hours)

At each VM window (00:00, 08:00, 16:00 UTC):
1. Compute mark-to-market change since last VM window per account.
2. Transfer VM between accounts: winning side's `available` increases, losing side's `available` decreases.
3. Adjust locks: as the mark moves, the max loss changes, so locks are recalculated.
4. If an account's balance cannot cover the adjusted lock, the account enters **margin call** status: risk-increasing orders are blocked until the member deposits more funds or reduces positions.

## On settlement

1. Contract settles at value V (0 or 1 for binary, [0,1] for measured).
2. Buyer receives: V * payout * qty. Seller receives: (1-V) * payout * qty.
3. Locks released. Transfers executed atomically. Collateral freed.

## Deposits and withdrawals

- **Deposit:** member wires cash to the CASS client money account. Bank confirms receipt. Collateral service credits `balance`.
- **Withdrawal:** member requests via REST API. Service checks `available >= withdrawal_amount`. Instructs bank to wire. Debits `balance`.
- Deposits and withdrawals are journaled and reconciled daily against the bank statement.

## Streams

| Stream | Direction | Content |
|---|---|---|
| `collateral.balances` | out | Balance updates per account (available, locked) — consumed by engine pre-trade |
| `collateral.transfers` | out | VM, settlement, deposit, withdrawal events — consumed by reconciliation |
| `trades` | in | Trade events trigger locks |
| `settlement` | in | Settlement events trigger payouts |

## Storage

Append-only journal + PostgreSQL projection (same pattern as all other services).

## API

| Endpoint | Purpose |
|---|---|
| `GET /v1/accounts/{account}/balance` | Current balance, locked, available |
| `POST /v1/accounts/{account}/withdraw` | Request withdrawal |
| `GET /v1/accounts/{account}/transfers?date=` | Transfer history |

Deposits are not API-initiated — they come from bank confirmations.
