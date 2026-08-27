# Reconciliation

Positions, trades, margin and fees are reconciled every business day between SWANS, the CCP and clearing members. Members can reconcile against the same files.

## Files (available from 18:00 London, T)

| File | Content | Recipients |
|---|---|---|
| `trades_T.csv` | Every trade with clearing state, TVTIC, CCP reference, fee components | Members (own), clearing members (clients) |
| `positions_T.csv` | Net and pending by account and contract; family and leg view | Members, clearing members |
| `marks_T.csv` | Filtered fair mark, daily settlement price, method | All |
| `vm_T.csv` | Variation margin per account | Members, clearing members |
| `margin_T.csv` | SWANS IM per account with attribution, budget, used | Members, clearing members |
| `fees_T.csv` | Fees per fill with attribution; reversals for voided trades | Members, clearing members |
| `ccp_recon_T.csv` | Venue position vs CCP position per clearing member account, difference, status | Clearing members |

## Process

1. 17:00: daily settlement price and marks fixed; margin run.
2. 17:30: CCP end-of-day trade and position report ingested; compared to venue trades with state `cleared` and to positions.
3. Breaks classified: missing trade (submitted but not in CCP report), extra trade, quantity or price mismatch, account mapping mismatch.
4. 18:00: files published with break flags. Clearing members receive their breaks with a proposed resolution.
5. Breaks are resolved before the next day's open; unresolved breaks escalate to the CCP and the clearing member's operations desk, and the affected account may be restricted to risk-reducing orders until resolved.

## Cash

SWANS holds no cash. VM, settlement and fees are collected by clearing members and the CCP; the fee file supports the clearing member's client invoicing. SWANS invoices clearing members monthly for venue fees.
