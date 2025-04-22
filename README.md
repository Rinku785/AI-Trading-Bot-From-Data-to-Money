# **AI-Powered Crypto Trading Bot | Data To Money**

| Metric            | Value      |
|-------------------|-----------|
| initial_balance   | 10000.0000|
| final_balance     | 13509.9919|
| total_return_pct  | 35.0999   |
| total_trades      | 519.0000  |
| win_rate          | 0.6859    |
| avg_profit        | 6.7630    |
| max_drawdown      | -28.7804  |
| Period            | 1 month   |

[![Watch the video](https://raw.githubusercontent.com/FatherMonkey916/AI-Trading-Bot-From-Data-to-Money/main/video_file.png)](https://raw.githubusercontent.com/FatherMonkey916/AI-Trading-Bot-From-Data-to-Money/main/ai_trading_bot_pnl_0.01_daily_winrate_0.68.mp4)

# **Data To Money** ***Documentation***

## **Overview**
This bot is an automated cryptocurrency trading system designed to operate on the **Bitget exchange**. It fetches real-time market data, applies predictive analytics using a pretrained **neural forecast model**, and executes buy/sell or exit strategies based on intelligent signal generation.

It uses **MongoDB** to store historical data, allowing for real-time inference, tracking, and decision-making. It also includes performance tracking and data export functionalities.

---

## **Features**
- ✅ Real-time candle fetching from Bitget.
- ✅ Uses a neural forecasting model (`TimeMixer`) for price prediction.
- ✅ Intelligent signal generation based on historical + forecast analysis.
- ✅ Automatic TP/SL management for risk control.
- ✅ Supports both scalping and swing trading.
- ✅ Trade logging and performance metric reporting.
- ✅ Sends webhook alerts (optional) for executed trades.
- ✅ CLI command (`"hey bot"`) to export current performance snapshot.

---

## **Architecture**

### **1. Market Data Handling**
- **Source**: Bitget's spot market endpoint (`/api/v2/spot/market/history-candles`).
- **Granularity**: `?min` intervals.
- **Stored in**: MongoDB collection (default: `bitget10K.ETHUSDT_??MA_timeseries`).
- **History Window**: ??? data points used for modeling, updated every minute.

### **2. Prediction Model**
- **Model Type**: NeuralForecast Model (`TimeMixer`).
- **Input**: Time series of `??-period average close prices (ten_avg)`.
- **Output**: 30-point future forecast used to derive predictive trends.

---

## **Core Components**

### **Class: `GoldCollector`**
#### Initialization
```python
runbot = GoldCollector(token="ETHUSDT")
```

#### Main Parameters
| Name | Description |
|------|-------------|
| `lookback` | Historical points for context (???) |
| `forward` | Prediction length (??) |
| `hold` | Window for recent comparison (??) |
| `tp_percent` / `sl_percent` | Take-profit / stop-loss thresholds |
| `volume_multiplier` | Trade volume filter to confirm breakout |
| `position_size` | Capital allocation per trade (as % of balance) |
| `initial_balance` | Starting virtual capital (default $10,000) |

---

## **Trading Logic**

### **Signal Generation (`generate_signal`)**
- **Historical Status**:
  - "up" if current avg < all of last ??
  - "down" if current avg > all of last ??
  - "hold" otherwise

- **Prediction Status**:
  - Analyzed over 6 intervals (?, ??, ??, ??, ??, ?? points)
  - Encoded as `++++++` (bullish), `------` (bearish), or mix.

- **Volume Check**:
  - Must exceed `volume_avg * volume_multiplier` to trigger trade.

- **Trade Decision**:
  - **BUY** if hist = up & pred = `++++++` or `+++++-`
  - **SELL** if hist = down & pred = `------` or `-----+`

---

## **Trade Execution**

### **Long/Short Buckets**
- Trades are grouped and tracked in `long_bucket` or `short_bucket` lists.
- When enough time has passed since their entry, a collective **exit** (`exit_buy` or `exit_sell`) is triggered with computed TP/SL levels.

### **Exits (`check_exits`)**
- Exit trades when:
  - Price hits either SL or TP.
  - OR trade duration exceeds 30 minutes.

### **Capital Management**
- Every trade reduces available capital.
- Profits from closed trades are added back to the balance.

---

## **Logging & Notifications**

### **Webhook Payloads**
Payloads can be optionally sent to a webhook for real-time updates:
- **Entry Notification (`send_payload1`)**
- **Exit Notification (`send_payload2`)**

Webhook structure follows the ChartPrime format.

---

## **Backtest / Performance Metrics**

Trigger `"hey bot"` in console to:
- Export all trades to CSV.
- Export performance summary.

### **Metrics Calculated**
| Metric | Description |
|--------|-------------|
| `total_return_pct` | % change in balance |
| `win_rate` | % of profitable trades |
| `avg_profit` | Average PnL per trade |
| `max_drawdown` | Maximum single-trade loss |

---

## **File Export**
- Trades: `{token}_trades_{timestamp}.csv`
- Metrics: `{token}_metrics_{timestamp}.csv`

---

## **Execution Flow**

1. `GoldCollector.run_gold_collector()` runs every 60 seconds:
   - Fetch historical + latest candle
   - Run signal & prediction
   - Check exits and update trades

2. `schedule_gold_collector()` runs in a daemon thread.

3. `listen_for_input()` listens for `hey bot` to save data snapshot.

---

## **Sample Trade Output**
```json
{
  "trade_id": 42,
  "ticker": "ETHUSDT",
  "entry_time": "2025-04-19T12:01:00Z",
  "action": "buy",
  "entry_price": 3185.42,
  "size": 1000.0,
  "sl": null,
  "tp": null,
  "status": "open",
  "hist_status": "up",
  "pred_status": "++++++"
}
```

---

## **Future Improvements**
- Add live order placement with exchange API (Bitget)
- UI dashboard for real-time visualization
- Use dynamic risk allocation (Kelly criterion / volatility targeting)
- Integrate multi-token support (e.g., BTC, SOL, etc.)
- Add unit tests & logging module
