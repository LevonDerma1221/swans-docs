# Clearing and collateral

## Full collateral (launch model)

Members deposit cash. SWANS locks max loss at trade time. At settlement, payout is distributed and locks released. No variation margin, no margin calls, no adjustments between trade and settlement.

| Step | What happens |
|---|---|
| **Deposit** | Member wires cash to CASS client money account via regulated custodian |
| **Pre-trade** | Balance check: `available >= max_loss` |
| **On trade** | Max loss locked from both buyer and seller |
| **Between trade and settlement** | Nothing. Locks stay fixed. |
| **Settlement** | Payout distributed, locks released |
| **Withdrawal** | Only available (unlocked) balance can be withdrawn |

This is deliberately simple. There is no credit exposure — the full contract value is always locked.

## Default handling

Max loss is always locked, so there is no default risk. If a member wants to exit a position before settlement, they trade out of it on the book.

## Future: capital efficiency

Locking max loss is capital-intensive. We want to explore ways to offer margin (partial collateral with leverage) without necessarily going through a prime broker. This is an open design question — see [Open decisions](../internal/open-decisions.md).

Options under consideration:
- SWANS-managed margin with VM
- Third-party margin provider
- Prime broker integration
- Other structures

## Future: CCP clearing

When a CCP partner is secured, trades can be submitted for novation. The core matching and settlement systems remain identical.
