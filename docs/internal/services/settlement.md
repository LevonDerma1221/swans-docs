# Settlement service

**Reference price** at each VM window (00:00, 08:00, 16:00 UTC) is the filtered fair mark `X_fair` from the marks service at the snapshot (hierarchy: auction print, executable microprice, decayed VWAP, model mark, governed fallback), projected onto family constraints. This mark drives VM and margin calculations.

**Final settlement:** Proposed (officer: source URI, hash, timestamp, value) → Determined (second officer; system enforces two distinct IDs) → dispute window (per schema, default 24 h; disputes with evidence; committee decision) → Final (published; positions cash-settled; trades → Settled; CCP notified) → or Fallback at the deadline (per schema rule).

All steps journaled with evidence hashes. Publishes `SettlementEvent` on `settlement`; drives trades, positions, market data status, PB adapter and collateral service (payout distribution for full-collateral accounts).
