# PnL and variation margin

SWANS reports realised and unrealised PnL for display and reconciliation. Cash movements are driven by variation margin through your prime broker, not by these fields.

Rules:

1. A position's lifecycle resets when its quantity touches or crosses zero.
2. Average entry price updates only when the position grows in the same direction.
3. Unrealised PnL = `net_qty × (mark − entry) × payout`, where `mark` is the current filtered fair mark (`X_fair`, see [Margin](margin.md)); at 17:00 it equals the daily settlement price.
4. Realised PnL on a closing fill = `qty_closed × (fill_price − entry) × payout`.
5. PnL excludes fees. Over a full lifecycle, realised PnL equals the cumulative variation margin attributable to the position.

Variation margin uses the daily settlement price, not the mark price; intraday unrealised PnL and end-of-day VM can differ.

At final settlement, the last variation margin is computed against the settlement value (0, 1, or the measured value) and the position closes.
