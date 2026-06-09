# OF MNQ 5M v3 — Order Flow Trading System

![Pine Script](https://img.shields.io/badge/Pine%20Script-v6-blue)
![Platform](https://img.shields.io/badge/Platform-TradingView-orange)
![Timeframe](https://img.shields.io/badge/Timeframe-M5-yellow)
![Broker](https://img.shields.io/badge/Broker-Tradovate%20%7C%20Apex-green)

Institutional order flow trading system for **MNQ (Micro E-mini Nasdaq)** futures on M5 timeframe. Combines Volume Profile, CVD (Cumulative Volume Delta), and Delta analysis to identify high-probability entries at key price levels with automated execution via PMT webhook.

---

## Core Concept

Unlike traditional indicator-based systems, this approach reads **institutional order flow** through:

```
Volume Profile → identifies Value Area High (VAH), Value Area Low (VAL), and POC
CVD           → measures net buying vs selling pressure
Delta         → confirms directional commitment at signal bars
```

Two operating modes:
```
TF (Trend Following) → breakout above VAH or below VAL + CVD confirmation
MR (Mean Reversion)  → rejection at VAH/VAL edges (disabled by default on M5)
```

---

## Architecture

```
M5 Bar closes
    └── Volume Profile recalculates (session bucket)
        ├── VAH / VAL / POC levels updated
        ├── CVD divergence check (10-bar lookback)
        ├── Delta consistency filter (last 3 bars)
        └── Signal grading (B / A / A+)
            └── If grade >= minimum → Entry signal
                ├── SL: candle extreme + buffer (structural, not ATR)
                ├── TP1: 1R | TP2: 2R | TP3: 3R
                └── PMT webhook → Tradovate → MNQ execution
```

---

## Signal Quality Grading

Every signal is graded before execution:

| Grade | Criteria | Action |
|-------|----------|--------|
| **A+** | VAH/VAL breakout + strong CVD divergence + delta confirmation | Best setups |
| **A** | VAH/VAL breakout + CVD confirmation | High quality |
| **B** | Basic breakout with minimal confirmation | Standard |

Configure minimum grade via `Grade mínimo para señal` input.

---

## Key Features

### Structural SL (not ATR-based)
```
LONG:  SL = low of signal candle − buffer (default 8pts)
SHORT: SL = high of signal candle + buffer (default 8pts)

Min SL: 20pts (protection for small candles)
Max SL: 55pts (filters excessive volatility candles)
```

### CVD Exit Signal
Real-time CVD degradation monitoring — if CVD drops X% from entry level → exit signal generated. Configurable threshold (default 50%).

### Multiple Exit Modes (Backtester)
11 exit strategies for performance comparison:
- TP at 1R / 2R / 3R (honest — no partial closes)
- Optimized: 0.5R-1R-1.5R-2R-3R ladder
- BE at 0.5R + runner to 3R
- 50% at 0.5R + various runner options

### Session Management
```
Open blackout:    3 bars (15 min) after NY open
Entry window:     18 bars (90 min) from open → closes at 11:00 ET
Max signals:      2 per session
Cooldown:         12 bars (60 min) between signals
Volatility filter: ATR session average > 300pts → no signals
```

### Automatic Bias Detection
```
Daily + Weekly structure score:
  +2/+3 → LONG only (bullish bias)
  -2/-3 → SHORT only (bearish bias)
  Neutral → uses manual direction filter
```

---

## PMT Integration

Dynamic bracket orders sent on each signal:

```json
{
  "symbol": "MNQ1!",
  "data": "buy",
  "quantity": "5",
  "price": "21000",
  "tp": 21060.00,
  "sl": 20940.00,
  "token": "YOUR_PMT_TOKEN",
  "multiple_accounts": [{"token": "...", "account_id": "..."}]
}
```

Position sizing formula:
```
Risk per leg = i_pmt_risk / 2
Quantity     = Risk_per_leg / (SL_pts × $2)
Example: $650 total, SL=30pts → qty = 325 / 60 = 5 contracts per leg
```

---

## Dashboards

| Dashboard | Content |
|-----------|---------|
| **BT / WR** | PnL by exit mode, WR, Quality Score breakdown, session stats |
| **Trade Log** | Last trades with levels hit (0.5R, 1R, 2R, 3R), grade, CVD exit |
| **Volume Profile** | Current + previous session VP with VAH/VAL/POC visualization |
| **Delta Bars** | Color-coded delta strength on each bar |

---

## Setup

### Requirements
- TradingView Pro+ (webhooks)
- PMT (PickMyTrade) connected to Tradovate
- Apex Trader Funding account

### Configuration
```
Symbol:          MNQ1!
PMT Token:       YOUR_PMT_TOKEN
Account ID:      YOUR_ACCOUNT_ID
Risk total USD:  $650 (splits into 2 legs of $325)
Exit mode:       TP en 1R honesto (PMT) — for live
Grade minimum:   A — recommended starting point
TF mode:         ON
MR mode:         OFF (too noisy on M5)
```

### TradingView Alert
```
Condition:   OF MNQ 5M v3 → Any alert() function call
Webhook URL: https://api.pickmytrade.trade/v2/add-trade-data-latest?t=YOUR_ID
Expiry:      Open-ended
```

---

## Comparison with UT Bot v6

| | **OF MNQ 5M v3** | **UT Bot v6** |
|---|---|---|
| Signal source | Volume Profile + CVD + Delta | Trailing stop crossover |
| SL type | Structural (candle extreme) | Structural (min/max pts) |
| Session | NY only (9:30–11:00 ET) | 24h |
| Signals/day | 0–2 | 2–4 |
| Entry confirmation | Order flow | Price action + filters |
| Best regime | Trending with volume | Any trending |

Both systems are compatible and can run simultaneously on the same chart.

---

## Risk Warning

This system is for educational purposes. Past performance does not guarantee future results. Futures trading involves substantial risk of loss.

---

## License

MIT License
