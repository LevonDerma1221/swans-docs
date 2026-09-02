# Collateral service

**Owns:** member balances, collateral locks, settlement payouts, withdrawals. SWANS manages funds directly via a regulated custodian or FCA CASS client money account.

## How it works

Full collateral is simple: lock max loss at trade time, release and pay out at settlement. No variation margin, no margin calls, no adjustments in between.

## Pre-trade balance check

The engine's pre-trade stage checks balance before matching:

```
max_loss = (side == Buy) ? price * contract_size * qty
                         : (1 - price) * contract_size * qty

if (account.available < max_loss) reject with INSUFFICIENT_BALANCE
```

Runs inside the engine shard (no RPC) using a cached balance snapshot. The collateral service pushes balance updates on the `collateral.balances` stream.

## On trade

1. Lock `max_loss` from buyer's available balance.
2. Lock `max_loss` from seller's available balance.
3. Total locked per trade = buyer max loss + seller max loss = contract_size * qty (always exactly the full contract value).

## On settlement

1. Contract settles at value V (0 or 1 for binary, [0,1] for measured).
2. Buyer receives: V * contract_size * qty. Seller receives: (1-V) * contract_size * qty.
3. Locks released. Transfers executed atomically.

## Deposits and withdrawals

- **Deposit:** member wires cash to the CASS client money account. Bank confirms receipt. Collateral service credits balance.
- **Withdrawal:** member requests via API. Service checks `available >= amount`. Instructs bank to wire. Debits balance.
- Deposits and withdrawals are journaled and reconciled against the bank statement.

## Data model

```cpp
struct CollateralAccount {
    AccountId account;
    int64_t balance_minor;           // total deposited
    int64_t locked_minor;            // locked for open positions
    int64_t available_minor;         // balance - locked
    Currency ccy;
    Timestamp as_of;
};

struct CollateralLock {
    AccountId account;
    TradeId trade;
    InstrumentId instrument;
    int64_t amount_minor;            // max loss for this side
    Timestamp locked_at;
};
```

## Streams

| Stream | Direction | Content |
|---|---|---|
| `collateral.balances` | out | Balance updates per account — consumed by engine pre-trade |
| `collateral.transfers` | out | Settlement, deposit, withdrawal events — consumed by reconciliation |
| `trades` | in | Trade events trigger locks |
| `settlement` | in | Settlement events trigger payouts |

## Storage

Append-only journal + PostgreSQL projection.

## Future: capital efficiency

Full collateral locks max loss, which is capital-intensive. How to offer margin (partial collateral with leverage) is an open design question — it does not have to involve a prime broker. See [Open decisions](../open-decisions.md).
