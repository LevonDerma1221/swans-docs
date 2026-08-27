# Build plan

| Milestone | Weeks | Deliverable | Depends on |
|---|---|---|---|
| M1 | 1–6 | `refdata_gen` + reference data service on the catalogue; trades and position services on a scripted stream; `libswansrisk` interfaces with Mehdi's first `EsMarginModel`; margin service producing the **CCP-format backtest and parameter file** (synthetic-price disclosure) | Catalogue CSV; Mehdi's methodology docs |
| M2 | 6–12 | Engine with pre-trade stage and margin budgets; QuickFIX gateway; market data over FIX and WebSocket; end-to-end scripted day with the CCP simulator; GCM files; Clearer API | M1 |
| M3 | 12–20 | Settlement service (four-eyes, disputes, fallback); daily settle; VM; RTS 22/24 extracts; SMARTS feed; REST API; Product A/B batch APIs | M2 |
| M4 | post term sheet | Real CCP adapter; RTS 26 timings; Eqlipse/commercial FIX decision; SBE multicast; LD4 deployment; DR; RFQ workflow if OTF | CCP choice; MTF/OTF decision |

M1 first: its output is what the LCH SA and clearing-member meetings need.
