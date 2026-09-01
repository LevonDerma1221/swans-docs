# Operations runbooks

Each runbook: trigger, owner, steps, communications, evidence.

## Operations model

SWANS runs 24/7 continuous trading with no open/close. Automated monitoring covers unattended hours; on-call staff handle escalations.

| Runbook | Trigger | Key steps |
|---|---|---|
| **Marks snapshot** | Periodic (configurable) | Fix marks; publish price file; reconciliation snapshot |
| **Instrument halt / resume** | Volatility trigger, source event, surveillance alert, operator | Halt with reason code; `SecurityStatus` broadcast; review; resume with 60-second notice; journal all steps |
| **Trade bust** | Member request within 15 min or operator | Review vs erroneous-trade policy; decision by two staff; `ExecType=H` with reason; positions reverted; fee reversal; collateral locks released; member notice |
| **Settlement determination** | Source publication | Officer 1 proposes with evidence hash; Officer 2 confirms; dispute window; committee if disputed; publish; collateral service distributes payout |
| **Collateral service down** | Balance update not received for 30 s | Pre-trade rejects all orders (stale balance); alert on-call; restore; recompute balances from journal; resume |
| **Member disconnect storm** | >10% sessions drop within 10 s | Check gateway health; cancel-on-disconnect applied; member notice; if venue-side, treat as incident |
| **Failover** | Heartbeat loss | Automatic per HA design; ops verifies new epoch; member notice with last accepted `SeqNum` |
| **DR activation** | Site loss | Declared by ops lead; FCA and members notified; resting orders cancelled; resume per DR procedure |
| **Source failure at deadline** | Settlement deadline passes without publication | Apply schema fallback; officer confirmation; flag as fallback; member notice |
| **Parameter change** | Governance approval | Version reference data; notify members per policy; apply at announced time |
| **Maintenance / hot deploy** | Planned | Rolling deploy, no downtime; if halt required, 24-hour notice to members |

## Automated monitoring (24/7)

All services emit health metrics. Alerts fire to the on-call rotation.

| Monitor | Threshold | Action |
|---|---|---|
| Engine heartbeat | Missing for 5 s | Page on-call |
| Gateway session count | Drop >10% in 10 s | Page on-call |
| Journal disk usage | >80% | Alert; if >95%, page on-call |
| Collateral balance stream | Stale for 30 s | Auto-reject orders; page on-call |
| PostgreSQL replication lag | >10 s | Alert |
| DR journal shipping lag | >60 s | Alert |

## Reconciliation

- Trade counts: engine journal vs trades service
- Position snapshot: position service vs collateral locks
- Collateral: locked amounts vs open positions; balance vs bank statement (daily)

Incident classification: P1 (trading impacted), P2 (post-trade impacted), P3 (degraded). P1 requires FCA notification and a written post-incident report within 5 business days.
