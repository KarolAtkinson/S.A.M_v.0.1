# Quick Start Guide

Get S.A.M. running in 5 minutes!

## 1. Prerequisites

- Python 3.11+
- Twelve Data API key (free at https://twelvedata.com)

## 2. Installation

```bash
# Navigate to project directory
cd S.A.M_01

# Create virtual environment
python -m venv .venv
source .venv/bin/activate  # Windows: .venv\Scripts\activate

# Install dependencies
pip install -e ".[dev]"
```

## 3. Configuration

```bash
# Copy environment template
cp .env.example .env

# Edit .env and add your API key
nano .env
# Or open with your editor and set:
# TWELVE_DATA_API_KEY=your_key_here
```

## 4. Run Tests

```bash
pytest
```

Should see ~15 tests pass ✅

## 5. Try It Out

```bash
# View CLI options
orbit --help

# Check account status
orbit account

# Run backtest
orbit backtest --symbol AAPL --strategy momentum
```

## 6. Explore

- Review `log.md` for project decisions
- Check tests in `tests/` for usage examples
- Examine `pyproject.toml` for all available commands

## Troubleshooting

| Issue | Solution |
|-------|----------|
| `ModuleNotFoundError: orbit` | Run `pip install -e .` from S.A.M_01 |
| `API key error` | Verify `TWELVE_DATA_API_KEY` in `.env` |
| `pytest not found` | Run `pip install -e ".[dev]"` |
| `Python 3.11 required` | Check `python --version` |

## Next Steps

- Read [README.md](../README.md) for full documentation
- Explore trading strategies in `strategies.py`
- Run backtests with different parameters
- Review risk settings in `config.py`

Happy trading! 🚀
