# Reconciliation

Positions, trades, margin and fees are reconciled every business day between SWANS and prime brokers. Members can reconcile against the same files.

## Files (available from 18:00 London, T)

| File | Content | Recipients |
|---|---|---|
| `trades_T.csv` | Every trade with clearing state, TVTIC, PB reference, fee components | Members (own), PBs (clients) |
| `positions_T.csv` | Net and pending by account and contract; family and leg view | Members, PBs |
| `marks_T.csv` | Filtered fair mark, daily settlement price, method | All |
| `vm_T.csv` | Variation margin per account | Members, PBs |
| `margin_T.csv` | SWANS IM per account with attribution, budget, used | Members, PBs |
| `fees_T.csv` | Fees per fill with attribution; reversals for voided trades | Members, PBs |
| `pb_recon_T.csv` | Venue position vs PB position per account, difference, status | PBs |

## Process

1. 17:00: daily settlement price and marks fixed; margin run.
2. 17:30: PB end-of-day position reports ingested; compared to venue positions.
3. Breaks classified: missing trade, extra trade, quantity or price mismatch, account mapping mismatch.
4. 18:00: files published with break flags. PBs receive their breaks with a proposed resolution.
5. Breaks are resolved before the next day's open; unresolved breaks escalate to the PB's operations desk, and the affected account may be restricted to risk-reducing orders until resolved.

## Cash

SWANS holds no cash. VM, settlement and fees are collected by PBs; the fee file supports the PB's client invoicing. SWANS invoices PBs monthly for venue fees.
