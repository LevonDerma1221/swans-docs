# Operations runbooks

Each runbook: trigger, owner, steps, communications, evidence.

| Runbook | Trigger | Key steps |
|---|---|---|
| **Start of day** | 07:00 London | Verify journals replayed and epochs consistent; reference data snapshot published; CCP link heartbeat; GCM limits loaded; margin budgets pushed; market data snapshot; open at 08:00 |
| **End of day** | 17:00–18:30 | Fix marks and daily settle; margin run; CCP report ingest; reconciliation; files published; RTS 22/24 extracts; surveillance EOD feed; backup verification |
| **Instrument halt / resume** | Volatility trigger, source event, surveillance alert, operator | Halt with reason code; `SecurityStatus` broadcast; review; resume with 60-second notice; journal all steps |
| **Trade bust** | Member request within 15 min or operator | Review vs erroneous-trade policy; decision by two staff; `ExecType=H` with reason; CCP notified; positions reverted; fee reversal; member notice |
| **Settlement determination** | Source publication | Officer 1 proposes with evidence hash; Officer 2 confirms; dispute window; committee if disputed; publish; CCP notified |
| **CCP link down** | STP failures or heartbeat loss | Halt new orders on cleared contracts after 30 s; existing matched trades queued; escalate to CCP; resume when link restored and queue drained; if beyond 15 min, member notice and FCA notification |
| **Margin service down** | No budget update for 30 s | Pre-trade continues on fast bound with limits scaled 0.5; alert risk desk; restore; recompute; restore limits |
| **Member disconnect storm** | >10% sessions drop within 10 s | Check gateway health; cancel-on-disconnect applied; member notice; if venue-side, treat as incident |
| **Failover** | Heartbeat loss | Automatic per HA design; ops verifies new epoch; member notice with last accepted `SeqNum` |
| **DR activation** | Site loss | Declared by ops lead; FCA and members notified; resting orders cancelled; resume per DR procedure |
| **Kill switch received** | Clearer API | Verify execution within 1 s; confirm to clearer; monitor for unkill |
| **Source failure at deadline** | Settlement deadline passes without publication | Apply schema fallback; officer confirmation; flag as fallback; member notice |
| **Parameter change** | Governance approval | Version reference data; notify CCP/GCMs/members per change-management; apply at announced time |

Incident classification: P1 (trading impacted), P2 (post-trade impacted), P3 (degraded). P1 requires FCA notification per the venue's obligations **[confirm]** and a written post-incident report within 5 business days.
