# Market data publisher

Subscribes to `BookUpdate` and `TradeExecuted`; publishes top-of-book and 5-level depth incrementals with `seq` and channel sequence; periodic and on-request snapshots; last trade, volume, open interest (EOD), reference price, daily settle, status events. Honors `transparency_mode` per instrument **[confirm waiver regime]**. Transports: FIX V/W/X and WebSocket JSON in phase 1; SBE multicast with TCP recovery in production. Conflation per session.
