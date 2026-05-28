# Synthetic VIX Alert — MQL4 Script

A MetaTrader 4 script that computes a **synthetic implied volatility index** from annualized log-return standard deviation across a rolling lookback window, derives a rolling average of that volatility via iterative sub-window computation, and fires spike or dip alerts when the current synthetic VIX reading deviates from the rolling average by more than the configurable relative multiplier thresholds — providing a broker-agnostic, formula-derived volatility measure for any instrument where external VIX data is unavailable.

---

## Overview

The VIX is a forward-looking implied volatility measure derived from options pricing — a data source unavailable in standard MT4 deployments for most instruments. This script bridges that gap by computing historical realized volatility as a proxy for implied volatility using the standard annualized log-return method. For each bar in the lookback window, `ln(closeToday / closeYesterday)` is computed and squared; the squared log returns are averaged and square-rooted to produce the standard deviation of log returns, then annualized by multiplying by `√252` (the conventional trading-day count) and scaled to percentage form. This produces a percentage value that approximates what implied volatility would express — the market's expected annualized price variation. The rolling average is computed by iterating `CalculateSyntheticVIX()` over progressively smaller sub-windows, producing a smoothed baseline. When the current reading exceeds `SpikeThreshold × average` or falls below `DipThreshold × average`, an alert fires — signaling a statistically significant volatility regime shift.

---

## Features

- **Annualized log-return volatility** — `logReturn = MathLog(closeToday / closeYesterday)` per bar; `sumLogReturns += logReturn²`; `syntheticVIX = MathSqrt(sumLogReturns / period) × MathSqrt(252) × 100`; non-positive price guard wraps each computation
- **Rolling VIX average** — `CalculateVIXAverage()` iterates `i = 1` to `period`, calling `CalculateSyntheticVIX(symbol, timeframe, i)` for each sub-window length and averaging the results — produces a smoothed multi-window baseline for relative comparison
- **Relative multiplier threshold detection** — `vixCurrent >= vixAverage × SpikeThreshold` → **Volatility Spike Detected**; `vixCurrent <= vixAverage × DipThreshold` → **Volatility Dip Detected** — both thresholds are multipliers of the rolling average, making the detection adaptive to changing baseline volatility
- **Alert message includes both values** — `AlertVIX()` formats with `"VIX Value: %.2f, Average: %.2f"` for complete relative context
- **Three notification channels:** sound alert, email, and mobile push
- **Lightweight loop** — polls once per minute (`Sleep(60000)`)

---

## How It Works

1. Every minute, `CalculateSyntheticVIX()` computes annualized volatility over `LookbackPeriod` bars
2. `CalculateVIXAverage()` computes rolling average across all sub-window lengths
3. Two relative threshold comparisons evaluated and alerts dispatched

---

## Input Parameters

| Parameter         | Type            | Default     | Description                                                              |
|-------------------|-----------------|-------------|--------------------------------------------------------------------------|
| `TradeSymbol`     | string          | `EURUSD`    | Symbol for analysis                                                      |
| `Timeframe`       | ENUM_TIMEFRAMES | `PERIOD_H1` | Timeframe for volatility calculation                                     |
| `LookbackPeriod`  | int             | `14`        | Lookback period for log return accumulation and rolling average          |
| `SpikeThreshold`  | double          | `2.0`       | Multiplier of rolling average above which a volatility spike is triggered|
| `DipThreshold`    | double          | `0.5`       | Multiplier of rolling average below which a volatility dip is triggered  |
| `EnableAlerts`    | bool            | `true`      | Fire an on-screen/sound alert                                            |
| `EnableEmail`     | bool            | `false`     | Send an email notification                                               |
| `EnablePush`      | bool            | `false`     | Send a mobile push notification                                          |

---

## Alert Message Format

```
Volatility Spike Detected detected on EURUSD (Timeframe: PERIOD_H1)
VIX Value: 18.42, Average: 8.95
```

---

## Installation

1. Copy `Synthetic_VIX_001.mq4` to `MQL4/Scripts/` in your MT4 data folder
2. Compile in MetaEditor (F7)
3. Drag onto any chart from Navigator → Scripts
4. Configure inputs and click **OK**

---

## Requirements

- MetaTrader 4 (`#property strict` compatible build)
- MQL4 compiler (MetaEditor)

---

## License

MIT License

Copyright (c) 2026

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
