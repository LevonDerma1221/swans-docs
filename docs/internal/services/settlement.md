# Settlement service

**Final settlement:** Proposed (officer: source URI, hash, timestamp, value) → Determined (second officer; system enforces two distinct IDs) → dispute window (per schema, default 24 h; disputes with evidence; committee decision) → Final (published; positions cash-settled; trades → Settled) → or Fallback at the deadline (per schema rule).

All steps journaled with evidence hashes. Publishes `SettlementEvent` on `settlement`; drives trades, positions, market data status, and collateral service (payout distribution).
