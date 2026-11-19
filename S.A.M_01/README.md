# S.A.M. - Core Project

**S.A.M. (Strategic Algorithmic Manager)** is the main project directory containing:

- **Data Layer**: SQLite database + Parquet bar files
- **ML Models**: PPO agent and training artifacts
- **Configuration**: Environment variables and settings
- **Development Log**: Project decisions and status

## Quick Reference

### Directory Contents

```
├── data/                          # Data storage
│   ├── bars/                      # Historical OHLCV (Parquet)
│   ├── models/hf_ppo/             # PPO model artifacts
│   ├── orbit.db                   # SQLite database
│   └── orbit.log                  # Application log
├── pyproject.toml                 # Dependencies & build config
├── .env.example                   # Environment template
├── .env                           # Local config (gitignored)
├── .gitignore                     # Git rules
└── log.md                         # Development log
```

### Quick Commands

```bash
# Install
pip install -e ".[dev]"

# Test
pytest

# Configure
cp .env.example .env && nano .env

# Run CLI
orbit --help
```

### Key Files

- **[pyproject.toml](pyproject.toml)** - All dependencies and build configuration
- **[.env.example](.env.example)** - Environment variable template
- **[log.md](log.md)** - Detailed project decisions and architecture notes

## Architecture

See [../README.md](../README.md#-architecture) for complete architecture documentation.

## For Development

- **Adding features?** Check [../CONTRIBUTING.md](../CONTRIBUTING.md)
- **Need help?** See [../QUICKSTART.md](../QUICKSTART.md)
- **Full docs?** Read [../README.md](../README.md)

---

For more information, see the root README or CONTRIBUTING guide.
