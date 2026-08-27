# Position service

**Owns:** net and pending positions per (account, instrument), average price, VM accounting, package leg views, hedge positions for Product A.

- `TradeExecuted` → `pending_qty` updated immediately (so pre-trade sees it). `CCPAccepted|Bilateral` → moves to `net_qty`. `CCPRejected|Busted` → pending removed.
- Packages are positions in their own right (as at the CCP). A **leg view** is derived for risk and client display: for package with legs `(L_i, r_i)`, the leg view adds `r_i × qty` to each `L_i`, price allocation documented per package type.
- Daily VM: `net_qty × (S_t − S_{t−1}) × payout`; `S` from the settlement service (CCP-sourced if the CCP sets it).
- Hedge positions uploaded by clients (Product A) stored separately, never netted into venue positions, consumed by the risk engine only.
- API: `GetPosition`, `GetPortfolio(account) → [Position + LegView + HedgePosition]`, `Snapshot(seq)`.
