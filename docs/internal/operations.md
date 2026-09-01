# Operations runbooks

Each runbook: trigger, owner, steps, communications, evidence.

## 24/7 operations model

SWANS runs continuous trading with no open/close. Operations are organised around **VM windows** (00:00, 08:00, 16:00 UTC) rather than start/end of day. Automated monitoring covers unattended hours; on-call staff handle escalations.

| Runbook | Trigger | Key steps |
|---|---|---|
| **VM window** | 00:00, 08:00, 16:00 UTC | Fix marks; compute VM per account; PB-managed: send margin schedule to PBs; full-collateral: execute VM transfers, adjust locks; publish price file; reconciliation snapshot |
| **VM window failure** | Mark or transfer not complete within 5 min of window | Alert on-call; halt risk-increasing orders; complete manually; resume; journal incident |
| **Instrument halt / resume** | Volatility trigger, source event, surveillance alert, operator | Halt with reason code; `SecurityStatus` broadcast; review; resume with 60-second notice; journal all steps |
| **Trade bust** | Member request within 15 min or operator | Review vs erroneous-trade policy; decision by two staff; `ExecType=H` with reason; PBs notified; positions reverted; fee reversal; collateral locks released; member notice |
| **Settlement determination** | Source publication | Officer 1 proposes with evidence hash; Officer 2 confirms; dispute window; committee if disputed; publish; PBs notified; collateral service pays out (full-collateral accounts) |
| **PB adapter down** | Notification failures or heartbeat loss | Halt new orders for PB-managed accounts after 30 s; existing matched trades queued; escalate to PB; resume when link restored and queue drained; if beyond 15 min, member notice and FCA notification |
| **Collateral service down** | Balance update not received for 30 s | Pre-trade rejects all full-collateral orders (stale balance); alert on-call; restore; recompute balances from journal; resume |
| **Margin service down** | No budget update for 30 s | Pre-trade continues on fast bound with limits scaled 0.5; alert risk desk; restore; recompute; restore limits |
| **Member disconnect storm** | >10% sessions drop within 10 s | Check gateway health; cancel-on-disconnect applied; member notice; if venue-side, treat as incident |
| **Failover** | Heartbeat loss | Automatic per HA design; ops verifies new epoch; member notice with last accepted `SeqNum` |
| **DR activation** | Site loss | Declared by ops lead; FCA and members notified; resting orders cancelled; resume per DR procedure |
| **Kill switch received** | PB API | Verify execution within 1 s; confirm to PB; monitor for unkill |
| **Source failure at deadline** | Settlement deadline passes without publication | Apply schema fallback; officer confirmation; flag as fallback; member notice |
| **Parameter change** | Governance approval | Version reference data; notify PBs/members per change-management; apply at announced time |
| **Maintenance / hot deploy** | Planned | Rolling deploy, no downtime; if halt required, 24-hour notice to members |

## Automated monitoring (24/7)

All services emit health metrics. Alerts fire to the on-call rotation via PagerDuty (or equivalent).

| Monitor | Threshold | Action |
|---|---|---|
| Engine heartbeat | Missing for 5 s | Page on-call |
| Gateway session count | Drop >10% in 10 s | Page on-call |
| Journal disk usage | >80% | Alert; if >95%, page on-call |
| VM window completion | Not done within 5 min | Page on-call |
| PB adapter heartbeat | Missing for 30 s | Auto-halt PB-managed orders; page on-call |
| Collateral balance stream | Stale for 30 s | Auto-reject full-collateral orders; page on-call |
| Margin budget stream | Stale for 30 s | Switch to fast bounds; page on-call |
| PostgreSQL replication lag | >10 s | Alert |
| DR journal shipping lag | >60 s | Alert |

## Reconciliation

Runs after each VM window (not just end of day):
- Trade counts: engine journal vs trades service vs PB drop copy
- Position snapshot: position service vs margin engine inputs
- Collateral: locked amounts vs open positions; balance vs bank statement (daily)
- Margin: recomputed from scratch vs incremental; differences flagged

Incident classification: P1 (trading impacted), P2 (post-trade impacted), P3 (degraded). P1 requires FCA notification per the venue's obligations **[confirm]** and a written post-incident report within 5 business days.
