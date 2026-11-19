# S.A.M. v0.1 - AI Trading Terminal

**S.A.M.** (Strategic Algorithmic Manager) is an experimental, **paper-trading-only** AI-powered stock research terminal that learns from historical trading patterns to provide intelligent trading insights.

## 🎯 Overview

S.A.M. combines machine learning with rigorous financial risk management to create a local-first trading research platform. It features:

- **Paper Trading Only** - Safe experimentation with no real money at risk
- **AI-Powered Analysis** - Learn from historical trading patterns using PPO (Proximal Policy Optimization)
- **Risk Management** - Deterministic risk gateway with position and loss limits
- **Virtual Exchange** - Realistic order simulation with slippage and fees
- **Backtesting Engine** - Test strategies against historical data
- **CLI Interface** - Easy-to-use command-line access

## 📋 Project Status

**Current Phase:** Vertical slice (MVP) - Core trading infrastructure **complete and tested**

### ✅ Completed (Phase 0-3)
- Data ingestion from Twelve Data API
- Data quality checks and validation
- Virtual Exchange with market, limit, stop, and stop-limit orders
- Deterministic risk gateway
- Backtesting engine with realistic market simulation
- CLI interface with Typer
- 15+ passing tests covering core functionality

### 🚀 In Progress
- PPO/Hugging Face agent training
- Multi-agent orchestration
- Advanced training pipeline

### 📅 Future Roadmap
- PostgreSQL/Redis integration
- Docker containerization
- Web-based dashboard
- Live paper trading loop
- Production deployment

## 🛠️ Project Structure

```
S.A.M_v.0.1/
├── S.A.M_01/                           # Main project directory
│   ├── data/
│   │   ├── bars/                       # Historical OHLCV data (Parquet files)
│   │   ├── models/
│   │   │   └── hf_ppo/                 # Hugging Face PPO model artifacts
│   │   ├── orbit.db                    # SQLite database (accounts, orders, fills)
│   │   └── orbit.log                   # Application logs
│   ├── pyproject.toml                  # Python project configuration & dependencies
│   ├── .env.example                    # Environment variables template
│   ├── .env                            # Local environment (gitignored)
│   ├── .gitignore                      # Git ignore rules
│   └── log.md                          # Project development log
├── README.md                           # This file
└── .git/                              # Git repository

```

## 💻 Getting Started

### Prerequisites

- Python 3.11+
- pip/setuptools
- Twelve Data API key (free tier available at https://twelvedata.com)

### Installation

1. **Clone the repository:**
   ```bash
   git clone https://github.com/KarolAtkinson/S.A.M_v.0.1.git
   cd S.A.M_v.0.1/S.A.M_01
   ```

2. **Create a virtual environment:**
   ```bash
   python -m venv .venv
   source .venv/bin/activate  # On Windows: .venv\Scripts\activate
   ```

3. **Install dependencies:**
   ```bash
   pip install -e ".[dev]"
   ```

4. **Configure environment:**
   ```bash
   cp .env.example .env
   # Edit .env and add your Twelve Data API key
   # TWELVE_DATA_API_KEY=your_key_here
   ```

### Quick Start

```bash
# View available commands
orbit --help

# Run backtesting
orbit backtest --symbol AAPL --strategy momentum

# Check account status
orbit account
```

## 🏗️ Architecture

### Core Components

| Component | Purpose | Location |
|-----------|---------|----------|
| **Config** | Settings, API keys, risk limits, logging | `config.py` |
| **Storage** | SQLite schema + Parquet bar store | `storage.py` |
| **API Client** | Twelve Data integration with retry logic | `twelvedata.py` |
| **Quality** | Data validation (duplicates, gaps, stale data) | `quality.py` |
| **Exchange** | Virtual exchange with order simulation | `exchange.py` |
| **Risk Gateway** | Deterministic risk checks (audit logged) | `risk.py` |
| **Strategies** | Momentum, mean-reversion, buy-and-hold | `strategies.py` |
| **Backtest** | Replay engine and performance metrics | `backtest.py` |
| **CLI** | Command-line interface | `cli.py` |

### Database Schema

**SQLite tables:**
- `accounts` - Trading account state (cash, positions)
- `orders` - Order history with status
- `fills` - Executed order fills with prices
- `ledger_entries` - Full trading ledger (trade_cash, trade_qty, fees)
- `audit_log` - Append-only audit trail (risk decisions)

### Virtual Exchange Rules

- **Order Types**: Market, Limit, Stop, Stop-Limit
- **Fills**: Market orders fill at next bar's open + slippage; Limit orders fill if range crosses
- **Fees**: `max(min_commission, per_share × qty)` - no shorting, no leverage
- **Ledger**: Every fill creates ledger entries; `reconcile()` validates consistency

### Risk Gateway

Deterministic rejection of orders that:
- Exceed 20% of equity per order
- Result in positions > 25% of equity
- Cause insufficient cash
- Breach daily loss limits
- Duplicate previous orders
- Use stale or missing data

All decisions are recorded in the **audit_log** (write-only via triggers).

## 🧪 Testing

```bash
# Run all tests
pytest

# Run with coverage
pytest --cov=src

# Run specific test
pytest tests/test_orbit.py::test_virtual_exchange
```

**Test Coverage:**
- Data quality checks
- Virtual exchange fills and rejections
- Risk gateway decisions
- Backtest isolation (strategy sees only past data)
- Order lifecycle (open → fill → cancel)

## 📊 Data & Models

### Historical Data
- **Source**: Twelve Data API
- **Storage**: Parquet files (bars/) for efficient querying
- **Quality**: Duplicate/gap/stale checks (reported, never silently repaired)
- **Symbols**: AAPL, MSFT, NVDA, SAP (pre-loaded for testing)

### ML Models
- **Framework**: Hugging Face Transformers + Reinforcement Learning
- **Algorithm**: PPO (Proximal Policy Optimization)
- **Model Artifacts**: Stored in `data/models/hf_ppo/`
  - `final_model.zip` - Trained model weights
  - `config.json` - Model configuration
  - `scaler.pkl` - Feature normalization

## 🔧 Configuration

Edit `.env` to customize:

```bash
# Twelve Data API key (required)
TWELVE_DATA_API_KEY=your_api_key

# Data directory
ORBIT_DATA_DIR=data

# Optional: Risk limits, fees, logging level (defined in config.py)
```

## 📝 Development Log

See [S.A.M_01/log.md](S.A.M_01/log.md) for:
- Project decisions and reasoning
- Development timeline
- Technical trade-offs
- Known limitations and future work

## 🚨 Important Notes

- ⚠️ **Paper Trading Only** - This terminal does NOT support live trading. All transactions are simulated.
- ⚠️ **Experimental** - S.A.M. is a research project. Use for learning and backtesting, not production trading.
- 🔒 **API Key Security** - `.env` is gitignored; never commit API keys.
- 📊 **Data Limitations** - Quality checks report issues but never silently repair data.

## 🤝 Contributing

This is an experimental project. Feedback and improvements are welcome!

## 📄 License

Free to use for educational and research purposes.

## 👨‍💻 Author

Created by **Karol Atkinson**

---

**Last Updated:** November 19, 2026  
**Version:** 0.1.0 (MVP)
