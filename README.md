# 🤖 XGBoost Forex Trading System

A full end-to-end machine learning pipeline for algorithmic forex trading — from data ingestion and model training to live signal generation and trade execution via MetaTrader 5.

---

## 🏗️ System Architecture

```
MT5 Platform
    │
    ▼
CSV Export (OHLCV data)
    │
    ▼
┌─────────────────────────────┐
│      Training Pipeline      │
│  feature_engine.py          │
│  pipeline.py                │
│  main.py (Grid Search)      │
│  Optuna Hyperparameter Opt. │
│  Walk-Forward Validation    │
│  Blind Test Evaluation      │
│  SHAP Financial Analysis    │
└────────────┬────────────────┘
             │  Exports trained model
             ▼
┌─────────────────────────────┐
│       API Server            │
│  api_server.py (Flask)      │
│  Receives OHLCV via webhook │
│  Returns BUY/SELL signals   │
└────────────┬────────────────┘
             │  Signal JSON
             ▼
┌─────────────────────────────┐
│     MetaTrader 5 EA         │
│  apixforexbotv10.mq5        │
│  Executes orders            │
│  Manages positions          │
└────────────┬────────────────┘
             │  Trade data
             ▼
┌─────────────────────────────┐
│     Performance Dashboard   │
│  report_dashboard.py        │
│  report_api.py              │
│  Real-time MT5 monitoring   │
│  Model management (add/rm)  │
└─────────────────────────────┘
```

---

## ✨ Features

### Training Pipeline
- **XGBoost classifier** with multi-class output (BUY / SELL / HOLD)
- **Grid search** across look-forward windows, timesteps, target modes (ATR / PCT), and learning rates
- **Walk-forward validation** with configurable rolling window (default: 30 days)
- **Blind test evaluation** on held-out recent data (default: 25%)
- **Optuna hyperparameter optimization** (100 trials, Pareto frontier export)
- **SHAP financial impact analysis** — identifies which features drive wins vs losses
- **Parallel execution** with joblib (4 cores)
- **Hall of Fame export** — saves top models ranked by Return, Sharpe, and Drawdown

### Feature Engine
- 60+ configurable feature toggles across categories:
  - Price Action & Microstructure
  - Volume & Volatility (ATR, GK, Parkinson, Yang-Zhang)
  - Momentum & Oscillators (RSI, MACD, ADX, Hull MA)
  - Smart Money Concepts (FVG, OB, BOS/CHoCH)
  - Multi-timeframe context (higher timeframe overlays)
  - Session & Time-based features
  - Entropy & Hurst Exponent

### Signal Server
- Flask REST API receiving live OHLCV data via webhook
- Loads trained XGBoost models dynamically
- Returns real-time BUY/SELL/HOLD signals with probability scores
- Model hot-swap without server restart

### MetaTrader 5 Expert Advisor
- Connects to Python signal server via HTTP
- Executes market orders based on ML signals
- Configurable risk management: Fixed Lot, Fixed Amount, Compound %, ATR-based
- Trailing stop, cooldown logic, spread filtering

### Performance Dashboard
- Real-time connection to MT5 demo/live accounts
- Displays per-model metrics: Return, Sharpe, Drawdown, Win Rate, Profit Factor
- Add/remove active models from the trading pool
- Monitor idle vs active model states

---

## 🛠️ Tech Stack

| Layer | Technology |
|---|---|
| ML Model | XGBoost, Scikit-learn |
| Hyperparameter Optimization | Optuna |
| Feature Importance | SHAP |
| Data Processing | Pandas, NumPy |
| API Server | Flask |
| Visualization | Matplotlib, Seaborn |
| EA / Broker Integration | MQL5 (MetaTrader 5) |
| Parallelism | Joblib |

---

## 📁 Project Structure

```
├── main.py                  # Grid search orchestrator
├── pipeline.py              # XGBoost training pipeline
├── feature_engine.py        # 60+ feature implementations
├── data_processing.py       # Data ingestion and preprocessing
├── backtesting.py           # Backtest engine with risk management
├── api_server.py            # Flask signal server
├── report_dashboard.py      # Performance dashboard
├── report_api.py            # Dashboard API endpoints
├── database.py              # Model registry
├── modeling.py              # Model utilities
├── model_manager.py         # Model lifecycle management
├── gerar_modelos.py         # Batch model generation
├── training/                # Model export and storage
├── models/                  # Saved trained models
├── dados/                   # Historical OHLCV data
├── apixforexbotv10.mq5      # MT5 Expert Advisor (signal consumer)
├── Gerenciador V10.mq5      # MT5 trade manager EA
├── AutoMaestro.mq5          # MT5 automation controller
└── requirements.txt
```

---

## ⚙️ Configuration

All training parameters are controlled via `main.py` config classes:

```python
class Config:
    class grid_search:
        look_forward_options = [5, 10, 15, 20]   # Prediction horizon (candles)
        timesteps_options = [15]                   # Sequence window
        target_mode_options = ['PCT']              # ATR or PCT target
        pct_multiplier_options = [0.003, 0.005]   # Target move size
        learning_rate_options = [0.01]
        n_estimators_options = [5000]
        max_depth_options = [4]

    class risk_management:
        params = {
            'RISK_MODE': 'FIXED_LOT',
            'RISK_FIXED_LOT': 0.01,
            'SPREAD_POINTS': 400,
            'LEVERAGE': 2000,
        }
```

---

## 🚀 Usage

**1. Export CSV data from MT5**
```bash
# Use MT5's built-in history export for your symbol (e.g., XAUUSD, M5)
```

**2. Run training grid search**
```bash
python main.py -f XAUUSD5.csv -t 1h
```

**3. Start the signal server**
```bash
python api_server.py
```

**4. Attach EA to MT5 chart**
```
Load apixforexbotv10.mq5 on your target symbol/timeframe
Configure server URL in EA parameters
```

---

## 📊 Example Output

```
Trial #  Config                           | Val %   Blind %  | Val Sharpe  Bld Sharpe | Status
#0       TS(1h)_LF10_TS15_PCT0.005        | 18.42%   12.31%  |    1.823       1.541   | ✅ BOM
#1       TS(1h)_LF15_TS15_PCT0.003        |  9.17%    6.88%  |    1.201       1.089   | ✅ BOM
#2       TS(1h)_LF20_TS15_PCT0.008        | 22.10%   -3.44%  |    2.101      -0.312   | 📉 DEGRADOU
```

---

## ⚠️ Disclaimer

This project is for educational and research purposes only. Algorithmic trading involves significant financial risk. Past backtest performance does not guarantee future results. Use on live accounts at your own risk.

---

## 👤 Author

**Hilton Paz** — [github.com/hiltontrip](https://github.com/hiltontrip) · [linkedin.com/in/hilton-júnior-b1544927](https://linkedin.com/in/hilton-júnior-b1544927)
