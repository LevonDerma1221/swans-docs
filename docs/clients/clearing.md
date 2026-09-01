# Clearing and collateral

## Two modes

SWANS supports two clearing modes, configurable per account. Both can run simultaneously.

| Mode | How it works | Who holds cash |
|---|---|---|
| **Full collateral** | Members deposit cash. SWANS locks max loss per trade. VM transferred every 8 hours. No PB needed. | SWANS (via custodian) |
| **PB-managed** | Bilateral with prime broker. SWANS is calculation agent. PB holds collateral under CSA. | Prime broker |

**Full collateral is the day-one default.** It works without any PB integration — simpler to launch and easier to onboard. PB-managed mode adds capital efficiency for members who have a prime broker relationship.

## Full-collateral mode

| Aspect | How it works |
|---|---|
| **Cash custody** | Member deposits to a CASS client money account held via a regulated custodian |
| **Pre-trade** | Balance check: `available >= max_loss` |
| **On trade** | Max loss locked from both sides; total locked = payout x qty |
| **VM windows** | Every 8 hours (00:00, 08:00, 16:00 UTC): mark-to-market transfers, locks adjusted |
| **Settlement** | Payout distributed from locked funds; locks released |
| **Margin call** | If balance can't cover adjusted lock after VM, risk-increasing orders blocked |
| **Withdrawal** | Only available balance can be withdrawn |

## PB-managed mode (optional)

For members with a PB:

1. Sign a CSA with a prime broker, naming SWANS margin engine as calculation agent.
2. PB sets limits through the PB API: max order size, notional limits, margin budget, kill switch.
3. SWANS sends trade notifications (FIX drop copy) and margin schedules to the PB.
4. PB collects margin from member under the CSA.

SWANS is not in the credit chain. If a member defaults, the PB manages the close-out.

## Settlement flow

1. SWANS determines settlement value (two-officer process, dispute window).
2. **Full-collateral:** collateral service distributes payout and releases locks.
3. **PB-managed:** settlement value sent to PB for cash settlement with member.

## Default handling

**Full-collateral:** max loss is always locked, so there is no credit exposure. If a member can't meet a VM call, risk-increasing orders are blocked but positions remain.

**PB-managed:** handled under the PB's CSA. SWANS suspends trading on notice from the PB.

## Future: CCP clearing

When a CCP partner is secured, the PB adapter can be swapped for a CCP adapter — no changes to the core system.
