# Reporting and surveillance

- **RTS 24 order records:** nightly CSV from gateway and engine journals; all required fields incl. the RTS 24 identifiers and pre-trade reject codes.
- **RTS 22 transaction reports:** per trade, per side; LEIs, identifiers, price, qty, timestamps, TVTIC, capacity, client identification where SWANS reports on behalf of members **[decision]**. Extract to the ARM.
- **Surveillance feed:** orders, trades, reference data to SMARTS format, real-time and EOD.
- **EMIR:** counterparties report; SWANS provides each side a trade file with UTI `SWANS-<LEI>-<TradeId>` **[confirm format]**.
- **Clock:** ns timestamps, UTC-traceable (RTS 25).
- **Retention:** 7 years **[confirm]**.
