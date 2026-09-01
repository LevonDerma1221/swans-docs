# Position service

**Owns:** net and pending positions per (account, instrument), average price, VM accounting, package leg views, hedge positions for Product A.

- `TradeExecuted` → `pending_qty` updated immediately (so pre-trade sees it). `PBAccepted|Bilateral` → moves to `net_qty`. `PBRejected|Busted` → pending removed.
- Packages are positions in their own right. A **leg view** is derived for risk and client display: for package with legs `(L_i, r_i)`, the leg view adds `r_i × qty` to each `L_i`, price allocation documented per package type.
- VM at each window (00:00, 08:00, 16:00 UTC): `net_qty × (S_t₁ − S_t₀) × payout`; `S` from the marks service (VM window mark).
- Hedge positions uploaded by clients stored separately, never netted into venue positions, consumed by the risk engine only.
- API: `GetPosition`, `GetPortfolio(account) → [Position + LegView + HedgePosition]`, `Snapshot(seq)`.
