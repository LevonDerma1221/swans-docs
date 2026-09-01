# Clearing and collateral

## Launch model: bilateral with prime brokers

Every trade on SWANS is notified to the member's prime broker after matching. The PB holds collateral, enforces margin, and settles. SWANS is the calculation agent — it computes the numbers, the PB executes.

```
Fund A ──CSA──▶ Prime Broker A ◀── margin schedule ── SWANS margin engine
Fund B ──CSA──▶ Prime Broker B ◀── margin schedule ── SWANS margin engine
```

SWANS is not in the credit chain. If a member defaults, the PB manages the close-out under its CSA with the member.

## Roles

| Role | Who | Obligations |
|---|---|---|
| Trading member | Funds, dealers, market makers | Trades; posts collateral to its PB |
| Prime broker | Banks | Holds collateral in segregated accounts; enforces IM/VM calls; settles |
| Self-clearing member | Market makers with PB capability | Posts margin directly, no intermediary |
| SWANS | Venue + calculation agent | Matches, computes margin, determines settlement |

## Before trading

1. Sign a CSA with a prime broker, naming SWANS margin engine as calculation agent.
2. PB registers the member's account on SWANS and sets, through the PB API: max order size, gross and net notional limits, daily loss limit, margin budget, and holds a kill switch.
3. SWANS enables trading rights on the account.

## Trade flow

1. **Match.** `ExecutionReport` (ExecType F) to both sides with `TrdMatchID` (880) and venue transaction ID.
2. **PB notification.** Trade details sent to both sides' PBs via the PB adapter (FIX drop copy).
3. **Margin update.** Margin engine recomputes IM/VM for affected accounts, sends updated schedule to PBs.
4. **PB margin call.** PB collects margin from member if required under the CSA.

## Drop copies and files

PBs receive real-time `TradeCaptureReport` messages for their clients' fills and, at each VM window: trade file, position file, margin schedule per client, and price/risk file.

## Settlement flow

1. SWANS determines settlement value (two-officer process, dispute window).
2. Final settlement value sent to PBs.
3. PB executes cash settlement with member: pays out or collects based on final value.
4. Margin released.

## Full-collateral mode

For members without a PB, or as a day-one fallback, SWANS offers a fully collateralised mode:

| Aspect | How it works |
|---|---|
| **Cash custody** | Member deposits to a CASS client money account held by SWANS via a regulated custodian |
| **Pre-trade** | Balance check: `available >= max_loss` (no margin engine dependency) |
| **On trade** | Max loss locked from both buyer and seller; total locked = payout * qty |
| **VM windows** | Every 8 hours (00:00, 08:00, 16:00 UTC): mark-to-market transfers between accounts, locks adjusted |
| **Settlement** | Payout distributed from locked funds; locks released |
| **Margin call** | If balance cannot cover adjusted lock after VM, risk-increasing orders blocked until deposit or position reduction |
| **Withdrawal** | Via REST API; only `available` balance can be withdrawn |

Both modes can run simultaneously on the same venue. See [Collateral service](../internal/services/collateral.md) for implementation details.

## Default handling

**PB-managed accounts:** handled under the PB's CSA with the member. SWANS suspends trading rights on accounts of a defaulting member on notice from the PB.

**Full-collateral accounts:** max loss is always locked, so there is no credit exposure. If a member cannot meet a margin call after a VM window, risk-increasing orders are blocked but existing positions remain. SWANS may close out positions under the rulebook if the shortfall persists.

## Future: CCP clearing

When a CCP partner is secured, trades will be submitted for novation. The CCP will face clearing members (replacing PBs in the chain), hold a default fund, and provide a guarantee. The SWANS architecture supports this as a swap of the PB adapter for a CCP adapter — no changes to the core matching, margin, or settlement systems.
