# SWANS Event Exchange

SWANS is an FCA-regulated, professional-only trading venue for standardised, cash-settled event contracts on corporate and macro catalysts: FDA decisions, earnings thresholds, litigation outcomes, M&A completions, central bank and regulatory rulings. Contracts trade on a central limit order book, are margined futures-style, and clear at a partner central counterparty.

**Visual overview:** [End-to-end views](diagrams.md).

**Status:** pre-authorisation. This documentation describes the venue as it will operate at launch. Anything marked **[confirm]** is pending counsel or counterparty confirmation.

## Quickstart

1. **Membership.** Apply as a trading member (professional clients and eligible counterparties only). See [Onboarding](clients/onboarding.md).
2. **Clearing.** Sign a clearing agreement with a general clearing member (GCM) that is a participant in the SWANS service at the partner CCP, or clear directly if you are a CCP member. See [Clearing](clients/clearing.md).
3. **Connect.** FIX 4.4 for order entry and drop copy; REST and WebSocket for reference data and market data. See [API](clients/api/fix.md).
4. **Certify.** Run the conformance scripts against the test environment. See [Conformance](clients/conformance.md).
5. **Trade.** Contracts are priced 0.005–0.995 in ticks of 0.005 with a 100-unit payout. See [Contracts](clients/contracts.md).

## Where things are

| You are | Start here |
|---|---|
| A fund or dealer connecting to trade | [Quickstart](clients/quickstart.md), [Guarantees](clients/guarantees.md), [Semantics](clients/api/semantics.md), [FIX](clients/api/fix.md), [Margin](clients/margin.md) |
| A clearing member | [Clearing](clients/clearing.md), [Clearer API](clients/api/clearer.md), [Reconciliation](clients/reconciliation.md) |
| A prime broker assessing the product | [Contracts](clients/contracts.md), [Margin](clients/margin.md), [Settlement](clients/settlement.md) |
| Building the venue | [Internal: system architecture](internal/architecture.md), [Contract engine](internal/services/contract-engine.md) |

## Environments

| Environment | Purpose | Location |
|---|---|---|
| Production | Live trading | LD4 (Equinix Slough), FIX cross-connect or VPN |
| Certification | Conformance testing, stable | LD4 |
| Simulation | Member development, may reset | Cloud |

Base URLs and session parameters are issued at onboarding.
