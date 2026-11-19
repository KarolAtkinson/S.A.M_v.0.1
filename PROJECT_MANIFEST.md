# S.A.M. Project Manifest

**Project:** S.A.M. v0.1 - AI Trading Terminal  
**Author:** Karol Atkinson  
**Repository:** https://github.com/KarolAtkinson/S.A.M_v.0.1  
**License:** MIT  
**Status:** Active Development (MVP Phase)

---

## 📦 What's Included

### Documentation
- ✅ **README.md** - Comprehensive project overview and architecture
- ✅ **QUICKSTART.md** - 5-minute setup guide
- ✅ **CONTRIBUTING.md** - Contribution guidelines
- ✅ **LICENSE** - MIT License
- ✅ **S.A.M_01/README.md** - Project-specific documentation
- ✅ **S.A.M_01/log.md** - Detailed development log and decisions

### Configuration
- ✅ **.gitignore** - Root-level git ignore rules
- ✅ **S.A.M_01/.gitignore** - Project-level git ignore rules
- ✅ **S.A.M_01/.env.example** - Environment template
- ✅ **S.A.M_01/pyproject.toml** - Project metadata and dependencies

### Project Structure
```
S.A.M_v.0.1/
├── README.md                    # Main documentation
├── QUICKSTART.md               # Quick start guide
├── CONTRIBUTING.md             # Contribution guidelines
├── LICENSE                     # MIT License
├── .gitignore                  # Git ignore rules
└── S.A.M_01/                   # Main project
    ├── README.md               # Project-specific docs
    ├── log.md                  # Development log
    ├── pyproject.toml          # Dependencies
    ├── .env.example            # Configuration template
    ├── .gitignore              # Project-level git rules
    └── data/
        ├── bars/               # Historical OHLCV data
        └── models/hf_ppo/      # ML models
```

---

## 🚀 Getting Started

### Quick Setup (5 minutes)
```bash
cd S.A.M_01
python -m venv .venv
source .venv/bin/activate
pip install -e ".[dev]"
cp .env.example .env
# Edit .env with your Twelve Data API key
pytest  # Verify installation
```

### First Commands
```bash
orbit --help           # View available commands
orbit account          # Check trading account
orbit backtest         # Run strategy backtest
```

### Documentation
- **New to the project?** Start with [QUICKSTART.md](QUICKSTART.md)
- **Want to contribute?** Read [CONTRIBUTING.md](CONTRIBUTING.md)
- **Need full details?** See [README.md](README.md)
- **Project decisions?** Check [S.A.M_01/log.md](S.A.M_01/log.md)

---

## 🏆 Key Features

✨ **Paper Trading Only** - Safe experimentation with no real money  
🤖 **AI-Powered** - Learns from historical patterns using PPO  
📊 **Virtual Exchange** - Realistic order simulation with slippage  
⚡ **Risk Management** - Deterministic risk gateway with audit logging  
🧪 **Thoroughly Tested** - 15+ tests covering core functionality  
🔧 **CLI Interface** - Easy command-line access via Typer  

---

## 📋 Development Status

### ✅ Completed
- Core trading infrastructure
- Virtual Exchange with order types
- Risk gateway and audit logging
- Backtesting engine
- CLI interface
- Comprehensive testing

### 🔄 In Progress
- PPO/Hugging Face agent training
- Multi-agent orchestration
- Advanced training pipeline

### 📅 Planned
- PostgreSQL/Redis integration
- Web-based dashboard
- Docker containerization
- Production deployment support

---

## 📊 Project Statistics

| Metric | Value |
|--------|-------|
| **Language** | Python 3.11+ |
| **Main Package** | `orbit` |
| **Entry Point** | `orbit` (CLI) |
| **Dependencies** | 11 core + pytest |
| **Test Coverage** | Core functionality |
| **Data Storage** | SQLite + Parquet |
| **ML Framework** | Hugging Face |

---

## 🔐 Important Notes

⚠️ **Paper Trading Only** - No live trading support  
⚠️ **Experimental** - For research and learning purposes  
🔒 **API Keys** - Never commit `.env` files  
📊 **Data Quality** - Checks reported, never silently repaired  

---

## 🤝 Contributing

This is an open project welcoming contributions! See [CONTRIBUTING.md](CONTRIBUTING.md) for:
- Bug reporting
- Feature suggestions
- Pull request process
- Development setup

---

## 📞 Contact & Support

- **Repository:** https://github.com/KarolAtkinson/S.A.M_v.0.1
- **Author:** Karol Atkinson
- **Issues:** Report via GitHub Issues
- **Discussions:** Use GitHub Discussions

---

## 📜 License

This project is licensed under the MIT License - see [LICENSE](LICENSE) for details.

---

**Last Updated:** November 19, 2026  
**Version:** 0.1.0 (MVP)  
**Status:** ✅ Project organized and documented
