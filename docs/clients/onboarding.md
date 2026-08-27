# Onboarding

## Who can be a member

SWANS is professional-only. Members must be professional clients or eligible counterparties under UK MiFID (per se or elective), and satisfy the venue's access criteria: fit and proper, sufficient trading ability and competence, adequate organisational arrangements, sufficient resources for the role. Retail clients cannot access the venue, directly or through direct electronic access.

| Member type | Description |
|---|---|
| Trading member | Trades for own account or for clients (agency). Funds, asset managers, dealers, brokers. |
| Liquidity provider | Trading member with quoting obligations under a market-making scheme. |
| Clearing member | GCM or self-clearing CCP participant; may also be a trading member. |

## Steps

1. **Application.** Legal entity details, LEI, regulatory status, professional-client classification, beneficial ownership, trading and compliance contacts, intended activity, algorithmic trading declaration.
2. **Clearing arrangement.** Evidence of a clearing agreement with a clearing member active in the SWANS service at the CCP, or of direct CCP membership. Your clearing member registers your trading ID with the CCP and sets your limits on SWANS.
3. **Rulebook adherence.** Signed membership agreement and rulebook acknowledgement.
4. **Connectivity.** Cross-connect at LD4, or VPN. FIX session credentials, IP allow-list, TLS certificates. REST/WebSocket API credentials (OAuth2 client credentials).
5. **Conformance.** Pass the certification suite for each interface you will use (order entry, drop copy, market data, RFQ if applicable). See [Conformance](conformance.md).
6. **Go-live.** Production credentials issued; clearing member limits enabled; trading rights activated per instrument group.

## Accounts

A member may operate several accounts (e.g. per fund or per desk). Every account maps to exactly one clearing member and one CCP account reference. Orders carry the account (FIX tag 1) and the parties group (453) identifying the clearing firm.

## Algorithmic trading

Members using algorithms must declare it, supply algorithm identifiers, and populate the RTS 24 fields on every order: investment decision within firm (tag 20050), execution within firm (20051), algorithmic indicator (20052). Members offering direct electronic access to their own clients need the venue's approval and the controls required by RTS 6 **[confirm]**.

## Test environments

Simulation is available on request for development. Certification mirrors production configuration and is where the conformance suite runs.
