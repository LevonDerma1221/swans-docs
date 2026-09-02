# FIX gateway

Phase 1: QuickFIX (C++), one session per member connection, TLS at the load balancer. Production: evaluate commercial engine for certification cost.

**Responsibilities:** session management; syntactic validation; symbol/ISIN → `InstrumentId`, parties → `AccountId`/`ClearerId` from the reference data snapshot; timestamp `received`; forward `OrderRequest` (SBE) to the engine shard; map engine events to `ExecutionReport`; persist every inbound/outbound FIX message to the audit journal with ns timestamps (RTS 24); per-session throttles (200 msg/s default); max open orders per account; cancel-on-disconnect.

**Message set:** FIX 4.4 with custom tags 7928, 20001–20064. RTS 24 fields (20050–20052) mandatory; missing fields rejected at the gateway with `SwansRejectCode` 1005 `MISSING_RTS24`.

**Drop copy sessions:** `TradeCaptureReport` on every state change. Future: separate `SenderCompID` per prime broker when PB integration is added.

**Market data session:** V/W/X; production alternative SBE multicast.
