# Onboarding

## Who can be a member

Membership criteria depend on the regulatory framework. At launch, SWANS targets professional clients and eligible counterparties under UK MiFID. Retail access may be possible under certain regulatory structures and is under evaluation — see [Open decisions](../internal/open-decisions.md).

| Member type | Description |
|---|---|
| Trading member | Trades for own account or for clients. Funds, asset managers, dealers, brokers. |
| Liquidity provider | Trading member with quoting obligations under a market-making scheme. |

## Steps

1. **Application.** Legal entity details, LEI, regulatory status, client classification, beneficial ownership, trading and compliance contacts, intended activity.
2. **Clearing arrangement.** Depends on account mode:
   - **Full collateral:** deposit cash to the SWANS custodial account. No PB needed.
   - **PB-managed:** evidence of a prime brokerage agreement naming SWANS margin engine as calculation agent.
3. **Rulebook adherence.** Signed membership agreement.
4. **Connectivity.** Cross-connect at LD4, or VPN. FIX session credentials, API credentials.
5. **Go-live.** Production credentials issued; trading rights activated.

## Accounts

A member may operate several accounts (e.g. per fund or per desk). Each account is either full-collateral or PB-managed. Orders carry the account ID (FIX tag 1).

## Test environments

Simulation is available on request for development and testing.
