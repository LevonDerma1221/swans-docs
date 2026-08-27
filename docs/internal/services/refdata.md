# Reference data service

**Owns:** instruments, schema policies, members, accounts, clearers, limits, calendars, settlement sources, hedge map, fee schedules, offset groups and triggers.

**Storage:** PostgreSQL 16; every change a versioned row. Snapshot published on startup and on change (`RefDataSnapshot{seq, instruments[], accounts[], limits[], fees[], offset_groups[]}`), incremental `RefDataUpdate{seq, type, payload}`.

**API (gRPC):** `GetInstrument(id|symbol|isin)`, `ListInstruments(filter)`, `GetSchemaPolicy(schema)`, `GetAccount`, `GetClearerLimits`, `GetFeeSchedule(group)`, `GetOffsetGroups()`.

**Generator `refdata_gen`:** reads the catalogue CSV (S-01…S-10, 447 contracts) and emits: SQL load; FIX `SecurityList` file for conformance; CCP product-spec CSV. Validation: unique symbol/ISIN; `last_trading_time < expected_resolution ≤ settlement_deadline`; tier-1 source present; schema policy exists; package legs listed and unexpired; hedge identifiers resolve.

**Hedge map:** `hedge_map(instrument_id, type, identifier, ccp)` used by risk, Product A, and the CCP cross-margin negotiation.

**Offset groups:** `offset_group(id, instruments[], offset_pct, enabled, triggers{min_settled, max_breach_rate, min_months, corr_window})`; enabling is a versioned reference-data change made by the margin service on trigger evaluation and approved by the risk committee.
