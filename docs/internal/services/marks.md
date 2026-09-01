# Marks service (filtered fair marks)

Produces the official filtered fair mark `X_fair` per contract, continuously intraday and at each VM window snapshot (00:00, 08:00, 16:00 UTC).

**Inputs:** auction prints (if auctions run), executable two-sided depth, recent trades, model mark from the latent probability map, staleness signals, family definitions.

**Hierarchy and combination (V6 §11):** candidates `X_auction`, `X_book` (microprice `(A·Q_B + B·Q_A)/(Q_B + Q_A)`), `X_trade` (time-decayed VWAP), `X_model`, `X_fallback`; combined in logit space with weights by depth, spread, recency, staleness; filtered `ℓ_fair = (1−α)ℓ_fair⁻ + α·ℓ_raw` with `α` high for deep/tight/recent markets; projected onto family constraints (`X_yes + X_no = 1`, buckets sum to one, ladders monotone).

**Outputs:** `X_fair` stream to margin, positions (VM), price file, settlement (reference mark candidate), Product A. Every mark carries its component weights, confidence, staleness and snapshot ID for audit.

**Controls:** stale-price flags; manipulation-resistant by construction (a single aggressive print in a thin book cannot move the mark materially); governed fallback requires two-officer approval.
