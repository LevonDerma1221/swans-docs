# Risk engine service

Wraps `libswansrisk`. Loads reference data; builds `MarketSnapshot` (venue mids, daily settles, hedge prices, option-implied event moves, factor levels); pulls portfolios; runs:

- **Intraday:** recalibration every 60 s or on quote moves beyond threshold; pushes per-account IM and fast bounds to the engine's pre-trade stage via `risk.budget` with a version.
- **End of day:** full risk report per account (Product A), inputs to the margin service.
- **Backtest mode:** replays catalyst outcomes and (synthetic, option-implied) price paths to produce CCP-format reports. **The backtest is partly synthetic because no venue price history exists; every report states this.**
- **Model plug-in:** Mehdi's models implement `IPricingModel`, `ICalibrator`, `IMarginModel` (see [Core analytics library](../risk-library.md)); the service never contains model logic.
