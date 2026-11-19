# Contributing to S.A.M.

Thank you for your interest in contributing to S.A.M.! This document provides guidelines and instructions for contributing.

## Code of Conduct

Please be respectful and constructive in all interactions.

## How to Contribute

### Reporting Bugs

1. Check existing issues to avoid duplicates
2. Create a new issue with:
   - Clear title describing the bug
   - Steps to reproduce
   - Expected vs. actual behavior
   - Your environment (Python version, OS, etc.)

### Suggesting Enhancements

1. Describe the enhancement and why it would be useful
2. Provide examples of how it would work
3. Note any potential drawbacks or limitations

### Submitting Pull Requests

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/your-feature`
3. Make your changes with clear commits
4. Add tests for new functionality
5. Run tests locally: `pytest`
6. Push and create a pull request with a descriptive message

## Development Setup

```bash
cd S.A.M_01
python -m venv .venv
source .venv/bin/activate
pip install -e ".[dev]"
```

## Testing

- Write tests in `tests/`
- Run tests: `pytest`
- Ensure tests pass before submitting PR
- Aim for meaningful test coverage

## Code Style

- Follow PEP 8
- Use type hints where appropriate
- Comment non-obvious logic
- Keep functions focused and testable

## Areas for Contribution

### High Priority
- PostgreSQL/Redis integration
- Dashboard web interface
- Improved PPO training pipeline
- Enhanced backtesting metrics

### Nice to Have
- Additional strategies
- More data sources
- Performance optimizations
- Documentation improvements

## Questions?

Open an issue or contact the maintainers.

---

Happy contributing! 🚀
