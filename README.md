# AlphaByProcess — Backtesting Framework

A modular, CLI-driven backtesting engine for systematic trading strategies. Built from scratch in Python — no QuantLib, no Backtrader, no abstraction layers hiding the math.

---

## What It Does

Loads price data, generates trading signals, simulates execution with transaction costs, and outputs a full suite of performance metrics — Sharpe, Sortino, Calmar, max drawdown, win rate, profit factor, and per-trade statistics.

Supports 7 strategies out of the box, runnable individually or all at once with a side-by-side comparison table.

---

## Stack

- **Python** — pandas, NumPy, yfinance
- **Data** — yfinance (default), Binance public API, or local CSV
- **No external backtest libraries** — engine, signal logic, and performance math all written from scratch

---

## Strategies

| Key | Strategy | Type |
|---|---|---|
| `sma` | SMA Crossover | Trend-following |
| `ema` | EMA Crossover | Trend-following |
| `macd` | MACD | Trend-following |
| `rsi` | RSI Mean Reversion | Mean reversion |
| `bollinger` | Bollinger Band Reversion | Mean reversion |
| `atr` | ATR Breakout | Volatility breakout |
| `vwap` | VWAP Reversion | Mean reversion |

All strategies use a 1-bar position shift to prevent lookahead bias. State-machine logic (where applicable) handles entries and exits correctly across holding periods.

---

## Usage

```bash
# Single strategy, default params
python run.py --ticker BTC-USD --strategy rsi

# Run all strategies with comparison table
python run.py --ticker AAPL --strategy all --ohlcv

# Override strategy parameters
python run.py --ticker SPY --strategy macd --params fast=8 slow=21 signal_period=5

# Custom date range, transaction cost, save results to CSV
python run.py --ticker ETH-USD --strategy ema \
  --start 2021-01-01 --end 2023-12-31 \
  --cost 0.002 --save
```

**Flags**

| Flag | Description | Default |
|---|---|---|
| `--ticker` | Ticker symbol (e.g. `BTC-USD`, `AAPL`) | required |
| `--strategy` | Strategy key or `all` | required |
| `--start` | Start date | `2018-01-01` |
| `--end` | End date | today |
| `--cost` | Transaction cost per trade (fraction) | `0.001` |
| `--ohlcv` | Load full OHLCV (required for ATR, VWAP) | off |
| `--source` | `yfinance` / `binance` / `csv` | `yfinance` |
| `--params` | Override strategy params as `key=value` pairs | — |
| `--save` | Export results to CSV | off |

---

## Project Structure

```
ALGO-TRA.../
├── run.py                        # CLI entry point
├── README.md
├── backtests/
│   ├── engine.py                 # Vectorized backtest execution
│   └── strategies/
│       ├── sma_crossover.py
│       ├── ema_crossover.py
│       ├── macd.py
│       ├── rsi_mean_reversion.py
│       ├── bollinger_reversion.py
│       ├── atr_breakout.py
│       └── vwap_reversion.py
└── utils/
    ├── data_loader.py            # Multi-source data loader
    └── performance.py            # Metrics calculation + trade extraction
```

---

## Performance Output (example)

```
────────────────────────────────────────
  BTC-USD  |  RSI
────────────────────────────────────────

  Returns
    Total Return               41.23%
    Annualized Return          12.80%
    Annualized Vol             18.45%

  Risk-Adjusted
    Sharpe Ratio               0.6941
    Sortino Ratio              0.9102
    Calmar Ratio               0.5213

  Drawdown
    Max Drawdown              -24.57%

  Trades
    Num Trades                      9
    Win Rate                   77.78%
    Avg Win                     6.31%
    Avg Loss                  -11.42%
    Profit Factor              0.5527
    Avg Hold Days             28.0000
```

---

## Design Decisions

**Why no Backtrader/Zipline?** The goal was to understand the mechanics — signal generation, execution simulation, and performance attribution — without a framework abstracting it away. Every formula here (Wilder's RSI, ATR, rolling VWAP, Sortino) is implemented and visible.

**Why state machines for signal logic?** Vectorized signals are fast but can't model holding periods or exit conditions cleanly. The strategies that need it (Bollinger, VWAP, RSI, ATR) use explicit state machines — slower on large universes but correct.

**Lookahead prevention** — every strategy shifts position by 1 bar: signal fires on close of day N, trade executes at open of day N+1.

