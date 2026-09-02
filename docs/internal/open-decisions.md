# Open questions

## Before we start building

| # | Question | When |
|---|---|---|
| 2 | Do we launch with binary contracts only, or also measured/tiered? | Phase 1 |
| 4 | How do we get ISIN codes? CFI/FISN allocation route? | Phase 1 |
| 5 | Which package types do we list and how do we allocate legs? | Phase 1 |
| 15 | What format for the latent loading file and parameter tables? | Phase 1 |
| 18 | What divergence threshold triggers a model-risk review between the two engines? | Phase 1 |
| 19 | What close-out horizon per product class? (CFTC vs FCA/EMIR rules) | Phase 1 |
| 24 | How do we handle anti-procyclicality in the margin buffer? | Phase 1 |

## Before trading goes live

| # | Question | When |
|---|---|---|
| 1 | MTF (central order book) vs OTF (RFQ with discretion) — which model? | Phase 2 |
| 6 | Clock precision, trade reporting thresholds, record retention periods (RTS 24/25/26) | Phase 2 |
| 9 | Build matching engine in-house or buy commercial? | After Phase 2 measurements |
| 10 | Fee schedule — what do we charge? | Phase 2 |
| 13 | Performance targets and failover design | Phase 2 |
| 14 | How many scenarios for margin? Run time budget? | Phase 2 |
| 16 | How do we define affiliate groups for fee anti-gaming? | Phase 2 |
| 17 | Do we need opening/closing auctions given 24/7 trading? | Phase 2 |
| 20 | Cross-margin scope: same-contract only, or broader? | Phase 2 |
| 21 | Cash-only collateral at launch, or also securities? | Phase 2 |
| 22 | How long before a margin call escalates? | Phase 2 |
| 23 | Tech stack: Aeron/SBE or Kafka/Protobuf? | Phase 2 |

## After launch

| # | Question | When |
|---|---|---|
| 7 | Transparency mode per instrument (pre/post-trade waivers) | Phase 3 |
| 8 | Does SWANS transaction-report on behalf of members? | Phase 3 |

## Strategic

| # | Question | When |
|---|---|---|
| 3 | Who sets reference prices — SWANS only, or can PB override? | When margin is offered |
| 11 | Netting offset thresholds per schema | Before first PB meeting |
| 12 | Do we need to be a registered benchmark administrator? | Before PB term sheet |
| 25 | Legal structure: venue + calculation agent, or pursue clearing licence? | Strategic |
| 26 | Can retail clients access the venue? Under what structure? | Strategic |
| 27 | How do we offer margin: PB, CCP, blockchain, or something else? | Strategic |

## Decision #27 — How to offer margin

This is the biggest strategic question. We launch on full collateral (max loss locked, no leverage). When we want to offer capital efficiency, we need to decide the mechanism. See the [margin & liquidation architecture reference](margin-architecture-reference.md) for a full design survey.

### What we already have that carries forward

| Capability | Where it lives |
|---|---|
| Jump-to-resolution risk modeling | Risk engine (MPOR + terminal engines) |
| Event ramp (escalate margin near catalysts) | Margin formula: IM blends toward max loss as resolution approaches |
| Portfolio offsets (complement pairs, spreads, families) | Family types + netting layers in margin service |
| Robust mark methodology | Logit combiner, filtered fair mark, staleness checks |
| Forward-compatible data model | ClearingMode, PBLimits, MarginCall, pending_qty all defined |

### What we'd need to build

| Component | What it does | Reference |
|---|---|---|
| **Liquidation engine** | Closes positions when a margined account breaches maintenance margin. Internalised (venue closes against the book), then routes residual to LP auction panel. | Pillar 3 |
| **Default waterfall** | Defaulter's margin → insurance fund → VM gains haircutting. Never touch non-winning third-party principal. | Pillar 3.4 |
| **Insurance fund** | Sized by EVT on jump-loss distribution, Cover-2 standard (survive the two largest concentrated jump losses at 99.9%). Funded by liquidation penalties + fee levy. | Pillar 3.4 |
| **Maintenance margin** | Volatility-adjusted threshold: `MM(p, τ) = max(floor, κ·p(1-p)·f(τ)·J)` — accounts for price level, time to resolution, and event type. | Pillar 2.3 |
| **Backstop LP programme** | Formalised market-maker obligations: quoting duties, mandatory auction participation, event-window presence. Remunerated by fee rebates + liquidation penalty share. | Pillar 4.3 |

### Key insight from the reference

Leverage on event contracts is a **controlled relaxation of full collateralisation**, paid for by insurance-fund capital and portfolio offsets. It is never free. The ex-ante controls (event-window escalation, jump-scenario IM, concentration limits) are the real risk system — the insurance fund is sized for the residual.
