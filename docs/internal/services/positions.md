# Position service

**Owns:** net positions per (account, instrument), average price, package leg views, hedge positions.

- `TradeExecuted` → `net_qty` updated immediately (so pre-trade sees it). At launch (full collateral), there is no PB acceptance step — trades are final on match.
- Future (when PB/margin offered): `TradeExecuted` → `pending_qty`; `PBAccepted` → moves to `net_qty`; `PBRejected|Busted` → pending removed.
- Packages are positions in their own right. A **leg view** is derived for risk and client display.
- Hedge positions uploaded by clients stored separately, never netted into venue positions, consumed by the risk engine only.
- API: `GetPosition`, `GetPortfolio(account)`, `Snapshot(seq)`.
