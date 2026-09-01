# Open decisions

| # | Decision | Owner | Needed by |
|---|---|---|---|
| 1 | MTF (CLOB) vs OTF (RFQ with discretion) as primary protocol | Levon + counsel | Before M2 |
| 2 | Contract payout structure: binary-only at launch, or also measured/tiered? Payoff function pluggability in ContractSpec | Levon + Mehdi | M1 |
| 3 | Who sets reference prices at each VM window: SWANS only, or PB can override? | Mehdi | Term sheet |
| 4 | CFI/FISN codes and ISIN allocation route | Levon + counsel | End M1 |
| 5 | Package types to list and their leg-allocation rule for leg views | Mehdi | M1 |
| 6 | RTS 25 precision class, RTS 26 thresholds, RTS 24 retention | Counsel | M2 |
| 7 | Transparency mode per instrument under the waiver regime | Counsel | M3 |
| 8 | Does SWANS transaction-report on behalf of members | Counsel | M3 |
| 9 | Production matching and FIX engine: in-house vs Eqlipse/commercial | Lyes | After M2 measurements |
| 10 | Fee schedule parameters | Mehdi | M2 |
| 11 | Offset trigger thresholds per schema (netting layer 3 scope) | Mehdi | Before first PB meeting |
| 12 | BMR: registered administrator vs third-party determinations only | Counsel | Before PB term sheet |
| 13 | Capacity targets and failover design (assumptions in capacity.md and ha-failover.md) | Lyes | Before M2 |
| 14 | Margin compute budget: scenario counts, run times, reduced set for pre-trade incremental | Mehdi + Lyes | Before M2 |
| 15 | Latent loading file format and production parameter tables for the PB parameter file | Mehdi | M1 |
| 16 | Fee anti-gaming aggregation key (affiliate group definition) and window | Mehdi + compliance | M2 |
| 17 | Opening/closing auction: whether to run one, given V6 puts auction prints at the top of the mark hierarchy (24/7 model may not need one) | Levon + Mehdi | Before M2 |
| 18 | Two-engine divergence threshold for model-risk add-on (A_model) | Mehdi + CRO | M1 |
| 19 | MPOR base horizon per product class and regulatory mapping (CFTC §39.13 vs FCA/EMIR) | Mehdi + counsel | M1 |
| 20 | Cross-margin group scope for phase 1: same-contract + structural only, or first common-driver group? | Mehdi + risk committee | Before M2 |
| 21 | Collateral eligibility policy: cash-only at launch, or also securities with haircuts? | Levon + CRO | M2 |
| 22 | Margin call due-time profile: how long before escalation (PB-managed vs full-collateral)? | Levon + Mehdi | M2 |
| 23 | Technology stack for clearing-layer services: Aeron/SBE (current) vs Kafka/Protobuf (risk engine spec v3.0 recommendation) | Lyes + Mehdi | Before M2 |
| 24 | Anti-procyclicality buffer methodology and parameters | Mehdi | M1 |
| 25 | Legal authority map: SWANS as venue+calc agent, or future DCO registration path? | Levon + counsel | Strategic |
