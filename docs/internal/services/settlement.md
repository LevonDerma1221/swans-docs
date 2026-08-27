# Settlement service

**Daily settlement price** at 17:00 London is the filtered fair mark `X_fair` from the marks service at the snapshot (hierarchy: auction print, executable microprice, decayed VWAP, model mark, governed fallback), projected onto family constraints. If the CCP sets daily settlement prices for cleared contracts, this service ingests and republishes them, reconciles them to `X_fair`, and uses `X_fair` only for bilateral contracts **[decision 3]**.

**Final settlement:** Proposed (officer: source URI, hash, timestamp, value) → Determined (second officer; system enforces two distinct IDs) → dispute window (per schema, default 24 h; disputes with evidence; committee decision) → Final (published; positions cash-settled; trades → Settled; CCP notified) → or Fallback at the deadline (per schema rule).

All steps journaled with evidence hashes. Publishes `SettlementEvent` on `settlement`; drives trades, positions, market data status, CCP adapter.
