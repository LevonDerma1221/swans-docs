# Regulatory mapping (UK)

| Obligation | Source | Where implemented |
|---|---|---|
| MTF/OTF operation, access criteria, professional-only | MiFID II Art. 18/19/20, 53; FCA MAR 5 | Onboarding, rulebook |
| Pre-trade controls, kill, price collars, volatility halts, throttles | RTS 7 | Pre-trade stage, engine, gateway |
| PB limits and trade notification timings | RTS 26; UK EMIR | PB API, trades service |
| Order record keeping | RTS 24 | Gateway/engine journals, reporting extracts, FIX 20050–20052 |
| Clock synchronisation | RTS 25 | PTP, ns timestamps |
| Transaction reporting | RTS 22 / MiFIR Art. 26 | Reporting service → ARM |
| Pre/post-trade transparency and waivers | MiFIR Art. 8–11, RTS 2 | `transparency_mode`, market data publisher |
| Market abuse surveillance | MAR | SMARTS feeds |
| Algorithmic trading, DEA | RTS 6 / MAR 7A | Onboarding declarations, RTS 24 fields |
| Prudential | IFPR (PMR £150k; fixed overheads) | Finance |
| Margin floors (OTC-classified) | UK EMIR RTS 153 (99.5% / 5 days; offsets on demonstrated correlation) | Margin schedule, offset triggers |
| Benchmarks | UK BMR | Settlement sources policy **[opinion pending]** |
| Instrument classification | MiFID II Annex I C(4)/C(10); Gambling Act perimeter | Contract design **[opinion pending]** |
| Business continuity | RTS 7 (2-hour resumption **[confirm]**) | DR site, journal shipping |

Items marked **[confirm]** or **[opinion pending]** are on the counsel list.
