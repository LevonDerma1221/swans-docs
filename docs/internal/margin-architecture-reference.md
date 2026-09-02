# Collateral, Margin & Liquidation Architecture for a Leveraged Event-Contract Exchange
### Institutional-Grade Design Reference — v1.0

**Scope.** This document catalogues the full design space for collateral infrastructure, margin health computation, liquidation execution, and illiquidity/tail-risk handling on a venue trading leveraged binary (and by extension scalar) event contracts priced in [0, 1]. Each pattern is presented in its DeFi-native form, then annotated with the **institutional adaptation** — the modifications required before the mechanism is credible to regulated professional counterparties (permissioned participation, custody segregation, deterministic recourse, auditability). Entry-level definitions are omitted.

**Governing constraint.** Everything below is downstream of one structural fact unique to event contracts: **settlement is a jump process, not a diffusion**. A share trading at 0.40 can print 1.00 or 0.00 in a single oracle update with no intermediate prices. Liquidation engines that assume they can act *between* the breach of maintenance margin and bankruptcy are borrowing an assumption from continuous markets that does not hold here. Every leverage decision must therefore be sized against jump-to-settlement loss, not path volatility. This is the single most important message for an institutional risk committee, and the honest framing is: **leverage on binaries is a controlled relaxation of full collateralisation, paid for by insurance-fund capital and portfolio offsets — never a free lunch.**

---

## Pillar 1 — Collateral Infrastructure & Asset Layering

### 1.1 Isolated Stablecoin Architecture

**Mechanics.** Per-market (or per-account) ERC-4626 vaults holding a single settlement asset (USDC-native, not bridged). Deposits mint non-transferable share accounting entries; the margin engine reads `convertToAssets(shares)` as free collateral. Isolation means a blown market cannot contaminate another vault — bankruptcy remoteness is enforced at the storage-layout level, not by policy.

- **Capital efficiency:** Lowest. 1:1 backing, zero rehypothecation, idle collateral earns nothing (see 1.2).
- **Structural risks:** Issuer risk on the stablecoin itself (depeg, blacklisting via `blacklist()` on the token contract — an underappreciated operational risk: a blacklisted vault address freezes *all* users); bridge risk if any non-canonical asset is admitted.
- **Implementation notes:** Use exchange-rate (non-rebasing) accounting throughout; rebasing tokens break share-based vault math. Enforce deposit caps per market at launch. All balance mutations must be atomic with position mutations in a single transaction — no two-phase "reserve then trade" flows, which create MEV-exploitable intermediate states.

**Institutional adaptation.** Replace or supplement the on-chain vault with tri-party or segregated custody at a regulated custodian, with the chain (or internal ledger) holding a tokenised claim. Under MiCA, the settlement asset should be an authorised EMT; under a UK venue, client-asset segregation follows CASS. The vault becomes an accounting truth layer; legal title sits with the custodian. This is the pattern institutions already accept from tokenised-collateral pilots at incumbent CCPs.

### 1.2 Yield-Bearing Collateral

**Mechanics.** Admit aUSDC, sDAI/sUSDS, or tokenised T-bill funds as margin, valued at `exchangeRate × balance` via the issuing protocol's rate oracle. The carry (currently the SOFR-adjacent range on T-bill tokens) offsets funding/borrow costs on levered positions — economically identical to interest on initial margin at a CCP.

- **Capital efficiency:** High — collateral works while posted.
- **Structural risks:** (i) *Protocol risk stacking* — the margin system now inherits Aave's oracle, governance, and utilisation risk; a 100%-utilisation event on the underlying pool makes collateral temporarily unredeemable even though it is solvently marked. (ii) *Rate-oracle manipulation* — exchange rates must be read from the canonical accrual function, never from a secondary-market price of the wrapper. (iii) *Redemption latency* — liquidation logic must either unwrap atomically (preferred: flash-redeem inside the liquidation transaction) or apply a liquidity haircut for the unwrap delay.
- **Haircut treatment:** Apply a protocol-risk haircut (2–10%) *on top of* any market haircut, sized to the historical max drawdown of the wrapper's exchange rate and the depth of its instant-redemption route.

**Institutional adaptation.** Institutions will not accept DeFi-lending wrappers as margin; they will accept tokenised money-market funds (BlackRock BUIDL, Franklin BENJI class) and, in a cleared context, the CCP's own eligible-collateral schedule (cash, gilts/OATs/Bunds with standard haircuts). The DeFi pattern and the CCP pattern are functionally identical — interest-bearing IM — which is a useful framing when presenting to allocators.

### 1.3 Cross-Collateralisation / Multi-Asset Vaults

**Mechanics.** A portfolio vault where BTC, ETH, and *other event shares* margin a target position. Free collateral = Σᵢ (balanceᵢ × priceᵢ × (1 − hᵢ)) − initial margin requirement.

**Haircut calculation for volatile collateral.** The defensible construction is:

> hᵢ = 1 − exp(−k · σᵢ · √T_liq) + λᵢ

where σᵢ is a conservative volatility estimate (EWMA with λ ≈ 0.94, floored by a long-window empirical quantile; GARCH(1,1) if you want mean-reversion in the forecast), T_liq is the assumed liquidation horizon **in units that reflect actual on-venue exit time, not a nominal 1 day**, k is the confidence multiplier (2.33–3.09 for 99–99.9%), and λᵢ is a liquidity add-on driven by an Amihud-style impact estimate against realistic liquidation size versus visible depth. Recompute daily; apply anti-procyclicality buffers (through-the-cycle floor or a 25% buffer releasable in stress, mirroring EMIR RTS 153 Art. 28) so haircuts do not spike exactly when clients are weakest.

**Event shares as collateral — three specific hazards:**
1. **Bounded payoff cuts both ways.** A YES share at 0.95 looks like near-cash but carries a latent −0.95 jump. Haircut event-share collateral on *jump-to-zero*, not on diffusion vol: effective haircut ≈ p for a long share priced at p, unless the share is part of a recognised offset (below). This is punitive by design.
2. **Wrong-way risk.** Reject or super-haircut collateral whose value is positively correlated with the margined position's loss scenario (e.g., posting "Fed cuts" YES shares against a short rates-event position). Implement as a correlation matrix over event taxonomies with hard eligibility gates, not just haircut adjustments.
3. **Offsets are where real capital efficiency lives.** Recognise deterministic structures — complement pairs (YES + NO of the same market are jointly riskless, see Pillar 4), mutually exclusive outcome sets within one event, and calendar/severity spreads — as margin offsets computed via a SPAN-style scenario grid: revalue the whole portfolio under {every referenced event → 0, → 1, and joint stress combinations} and set IM to the worst-case portfolio loss plus add-ons. This is the correct architecture for cross-margining event books, and it generalises cleanly to scalar-settled contracts (where the scenario grid scans the settlement variable's range rather than {0,1}).

**Institutional adaptation.** The scenario-grid portfolio margin *is* the institutional model — it is how CCP margin engines (SPAN, PRISMA-style VaR) already think. Crypto assets as collateral are optional and jurisdiction-dependent; the load-bearing eligibility list is cash + HQLA. Where a partner CCP clears the venue's contracts, haircut and eligibility schedules are inherited from the CCP's rulebook and the venue's contribution is the **parameter file**: per-contract jump scenarios, offset recognition rules, and concentration limits.

### 1.4 Native LP-Token Collateral

**Mechanics.** Accepting the platform's own AMM LP shares or backstop-vault shares as margin.

- **Valuation:** Never mark LP tokens off spot reserves (single-transaction manipulable via reserve imbalancing). Use fair-value LP pricing: derive reserves from the invariant and an *external* trusted price, i.e. the Alpha-Homora-class `fair LP = 2·√(k·p_fair)` construction for constant-product pools, generalised for LMSR/CLOB hybrid pools.
- **Structural risk — reflexivity:** This is the pattern's disqualifying feature at scale. Platform stress → LP token devaluation → collateral shortfalls → liquidations → deeper platform stress. It is the FTT/FTX loop in miniature. If admitted at all: aggregate cap (<5% of total collateral), haircut ≥ 50%, and automatic ineligibility trigger on drawdown.

**Institutional adaptation.** Do not offer it. Flag it in diligence materials as an explicitly rejected pattern with the reflexivity rationale — rejecting it visibly is worth more with institutional risk teams than any capital efficiency it buys.

---

## Pillar 2 — Margin Health Metrics & Valuation Frameworks

### 2.1 Mark Price: Orderbook Valuation vs. AMM Pricing

Neither raw source is usable alone.

- **CLOB-derived marks:** Top-of-book mid is trivially paintable in thin books (two dust orders move the mid arbitrarily). Use a **depth-weighted impact mid**: the average execution price of a reference notional Q swept against each side, i.e. mark = (impactBid(Q) + impactAsk(Q))/2, with Q sized to typical liquidation clip size. Books thinner than Q on either side fail over to the fallback hierarchy below.
- **AMM-derived marks:** Instantaneous CPMM/LMSR spot is single-transaction manipulable (flash-loan swap → mark moves → adversarial liquidation → swap back). LMSR is preferable for event markets — bounded loss for the sponsor, price is a softmax over outcome inventories, and manipulation cost is explicit in the liquidity parameter b — but its spot output must still be time-filtered before touching margin logic.
- **Production pattern — median-of-sources index:** markPrice = median(impact-mid, TWAP, external/committee reference), with a clamp band (e.g. ±5%) on per-update movement and a staleness flag per source. Margin health is always computed on the index, never on last trade.
- **Boundary handling:** Clamp marks to [ε, 1−ε] pre-settlement so log-space computations and haircut formulas never divide by zero, and so a manipulated 0.999 print cannot mark a short position to forced insolvency before resolution.

### 2.2 TWAP/VWAP Safeguards Against Oracle Manipulation

- **Geometric-mean TWAP** (Uniswap-v3-style accumulator of log-price ticks) over a 15–60 min window makes single-block manipulation ineffective and multi-block manipulation expensive: the attacker must hold the manipulated price across N blocks against arbitrage flow, so manipulation cost scales with window length × pool depth × forgone arb. Publish the manipulation-cost math per market; it is a diligence asset.
- **Multi-block MEV caveat:** A proposer controlling consecutive slots compresses that cost. Mitigations: longer windows for thin markets, cross-venue medianisation, and deviation circuit-breakers (if spot deviates > x% from TWAP, freeze margin-state transitions and liquidations for that market, mark at last-good index).
- **VWAP** is a manipulation-resistance complement only where organic volume exists; in a dying pre-resolution book, VWAP is stale by construction. Weight it dynamically by realised volume and decay its index weight toward zero as volume dries.
- **Flash-loan immunity as an invariant:** No margin-relevant price may be readable and writable within the same transaction context. Enforce by sourcing all marks from previous-block state (checkpointed accumulators), which structurally kills the atomic manipulate-liquidate-revert pattern.

### 2.3 Fixed vs. Volatility-Adjusted (Delta/Gamma-Based) Maintenance Thresholds

- **Fixed maintenance** (e.g. MM = 20% of max loss): simple, transparent, and *wrong at the boundaries*. A short YES at 0.98 has max loss 0.02 — a fixed-fraction MM implies absurd leverage exactly where gamma is at its most violent (a 0.98 → 1.00 move is a 100% loss of max-loss capital, and empirically these are precisely the shares that gap on news).
- **The bounded-payoff frame:** For event contracts, size margin as a fraction of **max loss**, not notional: long at p risks p; short YES at p risks (1 − p). "Leverage" is the ratio max-loss / IM. Full collateralisation is IM = max loss; everything below that is jump exposure.
- **Volatility-adjusted construction:** MM(p, τ) = max( floor, κ · p(1−p) · f(τ) · J(market) ), where p(1−p) proxies instantaneous binary variance, f(τ) escalates as time-to-settlement τ → 0 (jump probability density concentrates), and J is a per-market jump multiplier from the event taxonomy (scheduled binary announcements — FDA decisions, rate announcements, verdicts — carry J ≫ continuous-information markets like season-long outcomes). Add **event-window escalation**: MM ratchets to 100% of max loss automatically T-hours before a scheduled resolution catalyst, converting all positions to fully collateralised through the jump. This single rule eliminates the worst class of liquidation failure (Pillar 3.4) for *scheduled* events; only genuinely unscheduled news retains gap risk.
- **Institutional adaptation:** Present the whole framework as a scenario-based portfolio margin (per 1.3) with the above as the single-position degenerate case. Anti-procyclicality, margin-period-of-risk assumptions, and backtesting exceptions reporting (Basel-style traffic-light) should be documented in a margin methodology paper — institutions will ask for it verbatim, and where a partner CCP sits in the stack, its regulatory margin model is the binding layer with the venue supplying jump scenarios and add-on parameters.

---

## Pillar 3 — Liquidation & Execution Mechanisms

### 3.1 Public Keeper / Liquidator Bots

**Pattern.** Permissionless `liquidate(account)` callable by anyone once healthFactor < 1; caller receives a bounty (fixed % of seized collateral, or fixed fee + %).

- **Pros:** Censorship-resistant, no single point of failure, latency is market-priced via priority fees, zero standing infrastructure cost.
- **Cons / failure modes:** (i) *Correlated no-show* — keepers are profit-rational; in the exact tail scenario where liquidations matter most (mass resolution, gas spike, mempool congestion), bounty < gas + adverse-selection cost and keepers abstain. (ii) *PGA/MEV races* burn value to validators rather than the insurance fund; use MEV-Share/order-flow-auction style bounty auctions to recapture it. (iii) *Over-liquidation griefing* — always implement **partial liquidation** (close only to restore HF to a target buffer, e.g. 1.10) with per-tx close-factor caps; full-position seizure at HF = 0.999 is both value-destructive and a litigation surface. (iv) Bounty must be haircut-aware: seizing event shares as bounty payment transfers jump risk to the keeper, who will demand a wider discount — pay bounties in the settlement asset where possible.

**Institutional adaptation.** Permissionless invocation is a non-starter on a professional venue. Convert to a **whitelisted liquidator panel** (the venue's LPs — the Optiver/IMC/SIG/DRW class — under contractual response-time and pricing obligations), same smart-contract interface, gated by role. This is economically a standing default-management auction panel, which is exactly the shape a CCP default-management process takes; the mapping should be made explicit in institutional materials.

### 3.2 Just-In-Time (JIT) Dutch Auction Models

**Pattern.** On breach, the position enters an auction where the taker's discount improves monotonically with time (price starts near mark × (1 + premium) and decays — MakerDAO Clipper/LIQ-2.0 `cut`/`step` parameterisation, or Ajna-style bonded kicks). Any counterparty executes when the discount clears their reservation price; execution is atomic (take → collateral transfer → debt repayment in one transaction).

- **Pros:** Price discovery replaces an arbitrary fixed penalty; in liquid conditions positions clear near mark, minimising client harm; naturally rations liquidation flow instead of dumping.
- **Cons / failure modes:** (i) *Duration risk is fatal here* — a 20-minute decay curve is an eternity against a resolution jump; the auction can still be decaying when the oracle prints settlement. Mitigate with steep decay (minutes not hours), a terminal fallback (unsold tail routes to 3.3 or the merge path in 4.1), and the event-window escalation from 2.3 so auctions rarely coexist with imminent scheduled jumps. (ii) *Auction sniping/collusion* in thin keeper sets — the whitelisted-panel model needs at least 4–6 independent takers with obligations, or the Dutch clock becomes a gift to the last bidder. (iii) Restartable-auction logic (Maker's `redo`) is mandatory for stale auctions after circuit-breaker freezes.
- **Gas note:** Batch multi-account kicks into a single auction lot per market; per-account auctions during mass events are gas-prohibitive and serialise badly.

### 3.3 Protocol-Owned Liquidation Engine (Internalised)

**Pattern.** An off-chain risk worker operated by the venue monitors health continuously, and on breach force-closes against the book / internalises the position onto a venue-owned book, with on-chain (or ledger-level) authority to transfer the position. Deribit/Binance-style incremental liquidation: close in slices, re-check health between slices, stop as soon as HF recovers.

- **Pros:** Lowest latency (no public mempool), no MEV leakage, no market-dumping externality — the engine can warehouse and work out of risk over hours; deterministic behaviour that can be documented in a rulebook and audited.
- **Cons / failure modes:** (i) *Conflict of interest* — the venue profits when clients are liquidated at bad prices; must be neutralised structurally: liquidation P&L above cost accrues to the insurance fund, never venue revenue, and execution quality versus mark is reported per incident. (ii) *Single point of failure* — the worker down during the event is a total liquidation outage; run redundant workers with an on-chain dead-man's-switch that re-enables the permissionless/panel path if the engine misses heartbeats. (iii) Warehoused risk needs its own limits or the venue becomes a directional event fund by accident.
- **Institutional adaptation.** This is the credible default for a regulated venue, because it maps 1:1 onto CCP default management: **hedge → auction the residual to the member/LP panel → juniorise the defaulter's margin first**. Document it as a default-management waterfall, not a "liquidation engine," and institutions will recognise the shape immediately.

### 3.4 Debt Socialisation & Insurance Funds

**The core problem restated:** an unscheduled real-world event resolves; the oracle prints 1.00; every levered short YES gaps from HF = 1.4 to bankrupt with *zero* intervening prices. No engine of any speed forecloses in time. The loss is real and must land somewhere. The waterfall:

1. **Defaulter's full margin and remaining collateral** (including cross-collateral after haircuts).
2. **Insurance fund.** Funded by liquidation penalties, a bps levy on volume/fees, and seed capital. **Sizing is the institutional question:** run EVT (peaked-over-threshold) on the portfolio's jump-loss distribution — i.e., for each market, (open short-side notional at price p) × (1 − p) summed over plausibly-correlated simultaneous resolutions — and hold the fund at a Cover-2-style standard: survive the two largest concentrated jump losses at 99.9%. Publish the methodology; an unsized fund is marketing, a sized fund is infrastructure.
3. **Auction/ADL layer.** Residual bankrupt positions are force-assigned. Pure ADL (deleveraging profitable opposite-side accounts by ranked P&L×leverage) is standard in perps but commercially toxic to institutions — it converts their winning hedge into a truncated payoff. Prefer **variation-margin haircutting** (pro-rata haircut of the *gains* attributable to the specific resolution event) with full disclosure in the rulebook, which is the recognised CCP end-of-waterfall tool and at least arrives with regulatory pedigree.
4. **Never** touch non-winning third-party principal; loss mutualisation beyond event-gain haircutting is where venues die reputationally.

**Design implication worth stating plainly:** because layers 3–4 are so costly, the *ex ante* controls (event-window margin escalation to full collateralisation for scheduled catalysts, jump-scenario IM for unscheduled ones, concentration limits per market) are the real risk system. The insurance fund is sized for the residual, not the base case.

---

## Pillar 4 — Illiquidity Handling & Structural Tail Risks

### 4.1 Direct Share Conversion (Merge/Redeem Against the Premium Pool)

**Pattern.** In a fully-collateralised outcome-token system (Gnosis CTF-style), YES + NO of the same market are jointly a claim on exactly 1 unit of settlement asset held in the market's premium pool. `mergePositions(YES, NO) → 1.00` is available *unconditionally*, with zero dependence on order-book depth.

**As a liquidation primitive — the key structural insight of this pillar:** a liquidator closing an underwater long YES does not need a YES bid. It needs a **NO ask**: buy the complement at its ask price q, merge atomically with the seized YES, reclaim 1.00 from the pool, net 1 − q against the debt. The liquidation's liquidity requirement is displaced to the complement side of the book, and since (YES bid) and (NO ask) are arbitrage-linked (bid_YES ≈ 1 − ask_NO), the merge path is exactly as good as the best synthetic exit — but it executes **atomically in one transaction** (flash-borrow settlement asset → buy NO → merge → repay), immune to leg risk and to MEV sandwiching of a two-leg exit. Every liquidation router should quote both the direct-sale path and the merge path and take the better; in end-of-life books where one side has evaporated, the merge path is frequently the *only* path. Multi-outcome markets generalise via full-set completion (buy all complements, redeem the complete set).

**Institutional adaptation.** In a cleared/ledger implementation this is simply **compression / offsetting-position netting** — the venue's netting engine cancels opposing positions against the market's settlement escrow. Same primitive, CCP vocabulary.

### 4.2 AMM LP Debt Absorption

**Pattern.** A resident AMM (LMSR strongly preferred for event markets: sponsor loss is bounded by b·ln(n), pricing is inventory-driven, it cannot be drained by one-sided flow the way CPMM can) stands as the forced counterparty of last resort. The liquidation engine has a privileged route to hit the AMM at model price + penalty spread; LPs absorb the inventory and are compensated by the liquidation spread and ongoing fees.

- **Failure modes:** (i) LP capital is exactly as tail-exposed as the positions it absorbs — an AMM that absorbs pre-resolution short inventory is warehousing the jump; b must shrink (liquidity withdrawn) as τ → 0, which is precisely when you want it to grow. Honest design accepts that resident AMMs are a *mid-life* liquidity tool, not a terminal one. (ii) If LP shares are also collateral (1.4), absorption losses trigger reflexive liquidations — another reason to bar that pattern. (iii) Privileged liquidation flow is toxic flow by definition; price it in the LP prospectus or LPs will exit after the first event.

### 4.3 Backstop Liquidity Providers (Synthesised Market-Making Vaults)

**Pattern.** Pre-committed capital vaults with **contractual obligations**, not passive pools: max-spread and min-depth quoting duties per market tier, mandatory participation in liquidation auctions (3.2) up to a committed size, and event-window presence requirements — remunerated by fee rebates, a share of liquidation penalties, and a standing subsidy. Obligations are enforced by slashing/penalty on the committed bond.

- **Pros:** Converts "hope there's a bid" into a contracted service level; auditable; sized to the venue's stress scenarios rather than to yield-chasing flows.
- **Cons:** Standing cost; obligations must have *force-majeure carve-outs narrower than the LPs will ask for* — an obligation that suspends "in disorderly markets" is no obligation at all; negotiate hard caps (obligation holds up to X notional, then converts to auction participation) instead of soft outs.
- **Institutional adaptation.** This is a formal **market-maker programme + default-management-auction obligation**, the standard structure professional LPs (Optiver, IMC, Flow, SIG, DRW, Jane Street tier) already sign at incumbent venues. Presenting backstop vaults in that vocabulary — programme tiers, quoting obligations, DMA commitments — is materially more fundable than "vault APY."

### 4.4 Cross-Cutting Engineering Concerns

- **MEV around resolution:** the single most valuable MEV opportunity on an event venue is front-running the oracle's settlement report (buy YES at 0.94 in the same block the oracle prints 1.00). Mitigations: commit–reveal resolution (commitment lands, trading halts, reveal settles), pre-settlement trading halts triggered by the oracle's commit, private orderflow/encrypted mempool submission for the resolution transaction, and — on a permissioned institutional deployment — the problem largely dissolves because sequencing is venue-controlled and surveilled.
- **Atomicity invariants:** every state transition that couples price, collateral, and debt (trade, liquidation take, merge-liquidation, insurance-fund draw) must be single-transaction atomic; any multi-step flow with observable intermediate states is an MEV surface and an accounting-reconciliation risk.
- **Gas/throughput under mass-resolution load:** batch health checks via multicall over packed storage (health data for an account in ≤ 2 slots: packed uint96 balances, uint32 timestamps); event-driven re-checks (only accounts holding the resolving market) instead of global scans; pre-signed liquidation bundles staged before scheduled catalysts; L2/appchain deployment with a venue-operated sequencer for deterministic inclusion — which is also the natural meeting point with the permissioned-DLT expectations of institutional counterparties.

---

## Summary Matrix — What to Put in Front of Institutions

| Layer | DeFi-native pattern | Institutional presentation |
|---|---|---|
| Collateral | ERC-4626 isolated vaults, tokenised T-bill collateral | Segregated custody + eligible-collateral schedule with EMIR-style haircuts & anti-procyclicality |
| Portfolio margin | Jump-scenario grid over {0,1} per market + offsets | SPAN/VaR-class methodology paper; complement & spread offsets; event-window escalation to full collateralisation |
| Mark price | Median(impact-mid, TWAP, reference), previous-block reads | Documented mark methodology + circuit breakers + surveillance |
| Liquidation | Internalised engine, panel Dutch auctions, merge-path router | Default-management waterfall: hedge → panel auction → defaulter's margin → insurance fund → VM gains haircutting |
| Backstop | Obligated MM vaults | Market-maker programme + default-auction obligations |
| Tail honesty | Insurance fund sized by EVT on jump losses, Cover-2 standard | Published sizing methodology; leverage framed as a priced relaxation of full collateralisation |

The consistent translation rule: every DeFi mechanism above has a cleared-market ancestor. Institutions do not need to be sold novel machinery — they need to recognise machinery they already trust, implemented with better atomicity and transparency. Materials should therefore lead with the waterfall, the margin methodology, and the fund sizing, and treat the on-chain implementation as the execution substrate rather than the headline.
