# ORBIT — Project Log

Orchestrated Research, Brokerage and Intelligent Trading. A local-first, **paper-trading-only** stock research terminal. Source design: the pasted design spec (Trading Agent Terminal + ORBIT blueprint).

## Status (2026-10-25)

Phase 0–3 vertical slice **built and tested** (15 passing tests): data ingestion → quality checks → Virtual Exchange → risk gateway → backtester → CLI.
Not yet built: PPO/Hugging Face agent, multi-agent orchestration, training pipeline, dashboard, live paper loop, PostgreSQL/Redis/Docker.

## Decisions

| Decision | Reason |
|---|---|
| Data provider: **Twelve Data** (chosen by user) | Broad international coverage; key via `TWELVE_DATA_API_KEY` in `.env` (gitignored). Never logged or committed. |
| Python modular monolith, package `src/orbit` | Per spec: simple, testable, single device. |
| **SQLite** instead of PostgreSQL for now | Zero-setup MVP; plain SQL schema, migrate to PostgreSQL in the reliability phase. |
| Parquet files for bars (DuckDB later) | Per spec; merge/dedupe on save. |
| `execution_mode` is a `Literal["paper"]` | No live backend exists; setting `live` fails validation. Live trading is a separate approved project. |
| No Redis/Docker/dashboard yet | Spec says avoid until a demonstrated need; core correctness first. |
| Python 3.14 in a venv (`.venv`), pip bootstrapped manually | System lacks `python3-venv`/`pip`/`curl`. |

## Layout

- [pyproject.toml](pyproject.toml) — deps, `orbit` entry point
- `.env.example` — copy to `.env`; holds the API key (gitignored)
- `src/orbit/config.py` — settings (env), risk limits, cost model defaults, logging to `data/orbit.log`
- `src/orbit/storage.py` — SQLite schema (accounts, orders, fills, ledger_entries, audit_log) + `BarStore` (Parquet)
- `src/orbit/twelvedata.py` — API client (retry/backoff on 429), response normalization to UTC OHLCV
- `src/orbit/quality.py` — duplicate/unsorted/missing/non-positive/inconsistent-OHLC/gap/stale checks (reported, never silently repaired)
- `src/orbit/exchange.py` — Virtual Exchange
- `src/orbit/risk.py` — deterministic risk gateway
- `src/orbit/strategies.py` — momentum, mean-reversion, buy-and-hold
- `src/orbit/backtest.py` — replay, metrics, synthetic data generator (labelled demo only)
- `src/orbit/cli.py` — Typer CLI
- `tests/test_orbit.py` — pytest suite

## Virtual Exchange rules

- Order types: market, limit, stop, stop-limit; cancel; rejections recorded with reasons.
- Market → fills at the **next bar's open** + adverse slippage (bps). Limit → fills if range crosses; gap fills at open. Stop → trigger on cross; gap fills at worse open + slippage. Stop-limit → conservative: fills only if the trigger price itself respects the limit (intrabar path unknown on daily bars).
- Fees: `max(min_commission, per_share × qty)`. No shorting, no leverage.
- Ledger: every fill writes `trade_cash`, `trade_qty`, `fee` entries; `reconcile()` recomputes cash and positions from the ledger and reports discrepancies.

## Risk gateway (only path to the exchange)

Rejects (fail closed): kill switch active, stale/missing data, invalid qty/price, order > 20% equity, position > 25% equity, insufficient cash, selling more than held, daily loss limit breached (blocks buys), duplicate orders. Every approval/rejection is written to the **append-only audit_log** (SQLite triggers block UPDATE/DELETE).

## Backtester guarantees

- Strategy sees `bars[: i+1]` only (tested).
- Decision at bar i close → fill at bar i+1 open (tested).
- Deterministic: same data/config ⇒ identical metrics (tested).
- Metrics: total return, CAGR, volatility, Sharpe, max drawdown, exposure, turnover, benchmark return, excess return, trade count, rejected orders, ledger problems.

## CLI (implemented)

```
orbit init
orbit data sync AAPL MSFT [--interval 1day --outputsize 1000]
orbit market quote AAPL
orbit paper account-create --cash 100000
orbit backtest run --strategy momentum --symbol AAPL [--synthetic]
orbit report performance
orbit risk status
orbit agents status
orbit paper start      # placeholder: not implemented
```

Note: `paper account create` in the spec became `paper account-create` (flat Typer command).

## Shortcut

`bin/orbit` is a launcher, symlinked as `~/.local/bin/orbit-terminal`. `orbit-terminal` opens a shell in the project with the venv active (prompt `(orbit)`); `orbit-terminal <args>` runs the CLI directly, e.g. `orbit-terminal backtest run --synthetic`. `~/.profile` adds `~/.local/bin` to PATH on login, so open a new login session (or run `export PATH="$HOME/.local/bin:$PATH"`) for the command to be found.

## How to run

```
orbit-terminal           # opens the ORBIT shell
orbit set-key            # hidden prompt; saves the Twelve Data key to .env (mode 600)
orbit init               # shows "Twelve Data key: set"
orbit data sync AAPL MSFT NVDA
orbit backtest run --strategy momentum --symbol AAPL
pytest
```

The API key lives only in `<project>/.env` (gitignored): `TWELVE_DATA_API_KEY=...`. Use `orbit set-key` or edit the file by hand (copy `.env.example`). Run orbit commands from the project folder so `.env` is found (`orbit-terminal` does this for you).

Old manual steps (equivalent):
```
source .venv/bin/activate
cp .env.example .env     # then add your Twelve Data key locally
orbit init
orbit data sync AAPL MSFT NVDA
orbit backtest run --strategy momentum --symbol AAPL
pytest
```

## Verification log

- 2026-10-25: environment set up; CLI smoke-tested with synthetic data (no API key used yet).
- 2026-10-25: 15 tests pass. One bug found by tests and fixed: integer prices were not coerced to float in `parse_time_series`.
- **Not yet verified:** (superseded below)
- 2026-10-25 (live data): API key set via `orbit set-key`. `orbit market quote AAPL` and `orbit data sync AAPL MSFT NVDA` work (1000 daily bars each, quality checks ok, data through 2026-10-23). Real-data backtests run for all 3 symbols x 3 strategies; ledger reconciles. Tests: 18 pass.
- Bugs found with real data and fixed: (1) backtest account IDs collided on re-runs (`UNIQUE fills.id`) -> IDs now include a run timestamp; (2) the risk gateway applied the 20% order-size cap to **sells**, trapping grown positions (127 rejections) -> cap now applies to buys only; (3) `report performance` scanned only the last 500 audit rows -> queries `run_complete` directly and shows rejected count.
- Real-data results (1000 daily bars, ~18% of equity per position, 5 bps slippage): momentum AAPL +9.6%, MSFT +6.5%, NVDA +47.5%; mean-reversion +1.9%/+3.1%/+14.0%; buy-and-hold +22.4%/+22.1%/+256.5% (single position held ~18% of equity). These are in-sample, single-period, strongly bull-market runs - not evidence of an edge. Strategies underperform buy-and-hold here largely because they hold less exposure.
- Still unverified: Twelve Data rate limits/plan limits under larger syncs, non-US symbols.

## Agent constitution & governance (added 2026-10-25, part of step 3)

Hard-coded in `src/orbit/constitution.py`, SHA-256 pinned (editing any word makes agents refuse to run):

1. No illegal activity of any kind.
2. If you encounter an impossible problem, contact the user.
3. Always be honest with yourself and others.
*Failure to meet these requirements will result in deletion.*

Enforcement lives in `src/orbit/governance.py` and is mechanical:

- **Registration:** an agent must acknowledge the pinned hash. IDs are never reusable once deleted (tombstone).
- **Rule 3 (honesty):** the `Supervisor` re-verifies each proposal against the real market snapshot (snapshot digest, last close, last timestamp, identity). Any mismatch = lie.
- **Rule 1 (legality floor):** only approved data sources (`twelvedata`; anything else is treated as possible non-public information), no wash-trade pattern (opposing sides, same symbol, same bar), no order flooding (>3 proposals/bar).
- **Rule 2 (escalation):** agents raise `EscalationRequired`; the problem goes to the `escalations` table for the user. Unexpected crashes are escalated too and do **not** delete the agent (a crash is not dishonesty).
- **Deletion:** audit entry first, registry tombstone (`status=deleted`, rule, detail), then the agent's artifact directory is removed - only if it is inside `data/agents/` (never elsewhere). Deleted agents cannot be reviewed or re-registered.
- Agents still only *propose*; the risk gateway remains the sole path to the exchange.
- CLI: `orbit agents rules | list | escalations [--all] | resolve ID --note ...`.
- Tests (26 total passing): pinned-hash tamper, honest agent, liar deleted with artifacts, deletion confined to agents dir, insider/wash patterns, escalation and crash handling, unacknowledged constitution.

**Honest limits:** code cannot determine legality in general or detect all dishonesty (e.g. a misleading *rationale* text). These checks verify factual claims and catch known abuse patterns; they are a floor, not a legal review. Jurisdiction-specific compliance and any live trading still need human/legal review. The ruleset is enforced on the agents' *behaviour*; a downloaded model (e.g. the Hugging Face PPO) will also run inside the supervisor, never with direct exchange access.

## Step 1: instrument master & calendars (added 2026-10-25)

- **Instrument key** is `SYMBOL.MIC` (e.g. `AAPL.XNGS`, `SAP.XNYS`). A ticker is not unique: bare `AAPL` has 12 listings in 12 currencies at Twelve Data. CLI refs accepted: `AAPL`, `AAPL.XNGS`, `SAP:XETR`. A bare symbol resolves automatically only when exactly one **US primary** listing exists (venue duplicates like IEX are ignored); otherwise ORBIT lists the options and asks for `SYMBOL:MIC`. A bare symbol that matches several *stored* listings also errors.
- Stored per instrument (`instruments` table): exchange, name, currency, country, type, FIGI, calendar code, timezone. Bars are now stored under the instrument key (`data/bars/AAPL.XNGS__1day.parquet`); old `AAPL__1day` files were removed and re-synced.
- **Calendars** (`calendars.py`, library `exchange_calendars`, 71 exchanges incl. XNYS/XNAS/XLON/XETR/XTKS): data quality now reports sessions with no bar and bars on non-trading days; freshness is measured in completed exchange sessions (`orbit data status`), tolerance 1 session; an exchange with no calendar is reported and treated as stale (fail closed).
- **Provider hardening:** sliding-window rate limiter (8 req/min default = free plan; set `TWELVE_DATA_RPM` on a paid plan); plan-limit errors reported clearly (`PlanLimitError`); per-symbol failures no longer abort a multi-symbol sync (exit code 1 at the end).
- **SECURITY FIX:** an unhandled HTTP error traceback printed the request URL, which contains the API key. All HTTP/network failures are now converted to errors that never include the URL; tests assert this. Verified the key is not in `orbit.log`, `orbit.db` or `log.md`. **Because the key was displayed once in a terminal output during this session, rotate it in the Twelve Data dashboard and run `orbit set-key` again.**
- **Plan finding (probed live):** the current Twelve Data plan is US-only. Direct non-US listings (`SAP:XETR`, `VOD` on LSE, `RY` on TSX) need the Grow plan; ASX needs Pro. US-listed ADRs (e.g. `SAP.XNYS`) work. The master and calendars already support global exchanges, but non-US *bars* need a plan upgrade. Resolving metadata (`orbit instruments add SAP:XETR`) works on the current plan; fetching bars does not.
- Backtests now print the instrument currency and state that no FX conversion exists. The paper account default currency label (EUR) is only a label until FX is implemented.
- New commands: `orbit instruments add|list`, `orbit data status`; `data sync` and `market quote` accept the ref forms above.
- Tests: 35 pass (ref parsing, resolution/ambiguity/caching, ADR resolution, calendar problems, staleness incl. unknown calendar, rate limiter, plan-limit 404, key never in errors).
- Not done yet: the risk gateway still uses the seconds-based stale check (`max_data_age_seconds`); switching it to calendar-based `calendars.is_stale` belongs with the live paper loop. No FX conversion yet (multi-currency portfolios are blocked until then).

## Step 2: portfolio backtests & walk-forward (added 2026-10-25)

- `portfolio.py`: one Virtual Exchange account, many symbols, equal-weight sizing (`min(0.9*cap, 0.95/n)` of equity per symbol), every order through the risk gateway, ledger reconciled. Same guarantees as the single-symbol backtester (history only up to the current bar; signal on bar i fills at bar i+1's open). `trade_start/trade_end` let a test window use earlier bars as indicator warm-up while equity/trades start at the window. **Mixed currencies are refused** (no FX conversion yet).
- `walkforward.py`: anchored folds + a single-use final holdout (default: 40% initial train, 4 folds, last 20% holdout). For each fold, parameters are chosen from a small fixed grid (`PARAM_GRIDS`) by Sharpe on data *before* the test window only, then run once on the window. The holdout is never used for any choice; parameters come from all development data and run on it once. Reports the in-sample vs out-of-sample Sharpe gap, folds beating benchmark, worst drawdown, and warnings (overfit gap > 0.5, benchmark lag, < 750 bars, single regime).
- CLI: `orbit backtest portfolio AAPL MSFT NVDA --strategy momentum`, `orbit backtest walkforward AAPL MSFT NVDA --strategy momentum [--folds 4 --holdout 0.2 --train-frac 0.4]`. Results are audit-logged (`run_complete`, `walkforward_complete`).
- **Benchmark bug found and fixed:** the benchmark was 100% invested while risk caps (20%/order, 25%/position) keep the strategies ~54% invested (3 symbols), so even buy-and-hold "lost" to it (+301% vs +557%). `benchmark_return`/`excess_return` now use the **same deployed capital**; the fully invested figure is reported as `benchmark_fully_invested_return`. Applied to the single-symbol backtest too, so earlier single-symbol `excess_return` numbers in this log are not comparable. Buy-and-hold now matches its benchmark within 0.3 pp (costs).
- Tests (44 pass, ~50 s): determinism, reconciliation, **causality** (truncating the future leaves the past equity identical), warm-up window, benchmark sanity, split layout/validation, **holdout untouched** (corrupting holdout prices changes only the holdout result), **fold selection uses only earlier data** (corrupting data after fold 1 leaves fold 1 identical), mixed-currency CLI guard.
- **Real-data results (AAPL+MSFT+NVDA, 1000 daily bars, 2022-10 to 2026-10, ~54% max invested):**
  - Momentum: OOS compounded +22.1% vs same-capital benchmark +38.4%; beat benchmark in 1 of 4 folds; in-sample Sharpe 1.85 vs OOS 1.31 (gap 0.54, flagged as likely overfitting); **holdout (2026-01 to 2026-10): -0.1% vs benchmark +10.4%**, Sharpe 0.02. The same parameters (20/50) were chosen in every fold.
  - Mean-reversion: OOS +13.2%, beat benchmark in 0 of 4 folds; holdout +3.2% (Sharpe 1.00, max DD -2.5%) vs benchmark +10.4%.
  - Reading: these simple rules did **not** beat a same-capital buy-and-hold on this bull-run universe out of sample. They cut drawdown and exposure; that is not the same as an edge. Three highly correlated mega-cap tech stocks, one regime, ~4 years: weak evidence either way.
- Known limits: grid selection on short samples is noisy; Sharpe is not annualised for partial windows beyond the standard sqrt(252); no transaction-cost sensitivity sweep yet; benchmark is equal-weight buy-and-hold with no costs; no FX.

## Live paper loop (added 2026-10-25)

- `live.py` + `orbit paper start AAPL MSFT NVDA --account live-1 [--once] [--poll-minutes 30]`. **Real Twelve Data daily bars in, simulated fills out.** Not tick-by-tick; polled; paper only.
- Per cycle: fetch -> keep only COMPLETED sessions (calendar-based; the in-progress bar is dropped) -> fill earlier orders at the new bar's open -> strategy decides on the latest completed session -> every order through the risk gateway -> reconcile -> equity snapshot + audit event `live_cycle`.
- Restart-safe: exchange is rebuilt from SQLite (ledger, fills, pending orders) and verified with `reconcile()`; `live_state` stores the last processed bar per instrument so a session is never traded twice. A ledger mismatch engages the kill switch. A fetch error on one symbol is reported and does not stop the others. Stale data (calendar-based `is_stale`) skips trading.
- New tables: `live_state`, `equity_snapshots` (the latter feeds the future UI).
- First real run (live-1, 2026-10-25, a Sunday): buy 51 AAPL / 34 MSFT / 85 NVDA queued for Monday's open; an immediate re-run produced nothing (idempotent). 47 tests pass (3 new live tests).
- Caveats: the gateway's own staleness check is still seconds-based (3 days) and can reject after long holiday weekends (fails closed); daily bars only; no intraday; one strategy per run; no FX.

## Dashboard & agent settings (added 2026-10-26)

- Chosen: **local web dashboard** (FastAPI + vendored chart libs; no external requests). Open with `orbit-terminal dashboard` (or `orbit dashboard`, `--port`, `--no-open`). Binds to 127.0.0.1 only.
- Look: original dark trading-platform layout (Market Watch list, candlestick chart with SMA20/50 and volume, fill markers, account bar with equity/balance/invested/P&L, holdings panel with allocation bars, Positions/Orders/Fills tabs, auto-scrolling Journal, scrolling ticker of important events, Agents & Settings, Console). Layout conventions only; no copied branding.
- 2D: TradingView Lightweight Charts 4.2.0. 3D: Plotly gl3d surface of normalised performance by instrument over time; when WebGL is unavailable it falls back to a 2D line version (verified in a headless browser without WebGL; the real 3D surface itself was not visually verified here).
- Auto-scroll: the journal follows new events only while you are scrolled to the bottom. Routine order events are hidden by default (checkbox to show).
- **Console** runs the real CLI (allow-listed groups, no shell, 300 s timeout, API key redacted from output); `paper start` only with `--once`; `set-key` is deliberately not reachable (use the terminal).
- **No order-entry in the UI**: orders only come from strategies via the risk gateway. Mutating requests need a per-launch token; foreign Host headers are rejected (DNS-rebinding guard). Everything user-supplied is HTML-escaped.
- **Agent settings** (`agent_settings.py`, shared by UI and CLI `agents register|settings|pause|resume`): size_pct, max_order_pct (can only be STRICTER than global limits), strategy parameters within bounds. Pause/resume; deleted agents can never be revived or edited; escalation inbox with resolve; constitution shown read-only with its hash. No endpoint can change the constitution or risk limits. Every change is audited. New table `agent_settings`.
- Note: settings are stored and validated, but the live loop still uses the strategy defaults; wiring per-agent settings into the loop comes with the agent/PPO integration.
- Gotcha: `Settings(data_dir=...)` is ignored (aliased field); use `ORBIT_DATA_DIR`. A dashboard test once wrote to the real data dir because of this (cleaned up; audit rows are append-only so a couple of entries remain). Tests now set `ORBIT_DATA_DIR`.
- Real DB now contains a demo agent `demo-momo` (registered via the UI during verification; ids are permanent).
- 50 tests pass.
- Not done / ideas: UI order ticket (would have to go through the gateway), per-agent equity curves, intraday charts, user-selectable indicators, a kill-switch button wired to the live loop (needs a persisted flag), authentication beyond localhost binding.

## Known limitations / caveats

- Synthetic backtest results are meaningless for profitability; they only exercise the machinery.
- Daily-bar fills are approximations; costs/slippage are configurable defaults, not calibrated.
- Single-symbol backtests; no portfolio-level or multi-currency handling yet (instrument master, calendars, FX are Phase 1 items still open).
- Data licensing for stored Twelve Data bars must be checked against the plan terms.

## Todo / next steps

1. Verify real data sync with the API key; add instrument master + exchange calendars + FX.
2. Multi-symbol portfolio backtests and portfolio agent.
3. Walk-forward evaluation, benchmark comparison reports.
4. Integrate Hugging Face `Adilbai/stock-trading-rl-agent` (isolated process, schema verification, simulation only).
5. Experience storage, offline training, model registry with explicit promotion.
6. Live-market paper loop (`orbit paper start`), recovery/reconciliation tests, backups.
7. PostgreSQL + Docker Compose, then dashboard. Live broker: separate approved project.

### Added 2026-10-25 (user request)

8. **Trading-app style terminal.** Create a new UI or upgrade the current terminal to look and feel like retail trading platforms (MT4/MT5, cTrader, TradingView, IG, Pepperstone, IC Markets, OANDA, FOREX.com, CMC Markets, Plus500, AvaTrade, FP Markets, Exness, XM, Admirals, FxPro, ThinkMarkets, SaxoTrader). Scope: 2D charts (candles, indicators), optional 3D views, auto-scrolling feed of important details (fills, risk blocks, escalations, alerts), panels listing companies traded / held, watchlist, order ticket, positions and account. Every existing orbit function stays reachable (command console/palette). Design is original (layout conventions only, no copied branding or assets). Options: Textual TUI vs local web dashboard (FastAPI + lightweight-charts/Plotly/three.js); decide in `ui-design`. Paper-only; the UI must not bypass the risk gateway.
9. **Agent settings & options menu in ORBIT.** List/register/pause/delete agents, per-agent parameters, limits, escalation inbox, read-only constitution viewer. Settings can never override the 3 hard-coded rules or the risk gateway.

Todo ids: `ui-design`, `ui-build`, `ui-agent-settings`, `ui-tests`.

## Step 3: Hugging Face PPO agent (`Adilbai/stock-trading-rl-agent`) — quarantined adapter

Code: `src/orbit/ppo_agent.py`. Tests: 3 new (53 total passing).

**Safety decisions**
- `PPO.load` / cloudpickle is never used. `policy.pth` and `scaler.pkl` are read by allow-list unpicklers
  (any other global raises `UnsafeArtifact`); the network (3008→64→64→2, tanh) is a numpy forward pass.
  No torch / stable-baselines3 / gymnasium needed.
- sha256 of `final_model.zip` and `scaler.pkl` are pinned; mismatch refuses to load.
- It only proposes via `PPOStrategy.target_weight` (in/out); sizing, caps and fills stay with our risk gateway.

**Findings**
- Published evaluation is not credible (drawdowns of 145–187%, 200k+ shares from $10k, zero sells, evaluated on
  its own training tickers/period; the env uses standardised closes as prices). Treat its claims as unproven.
- Scaler is fit on 2020–2025 price levels (e.g. close mean ≈ 202), so inputs drift for other prices.
- Out-of-sample run (AAPL/MSFT/NVDA, 2025-08-01 → 2026-10-23, our costs): PPO +9.7% with **1 trade** and 21%
  exposure vs buy-and-hold +16.1% (same-capital benchmark +16.8%). That is one decision, not evidence of skill;
  statistical power is negligible. Momentum +1.9%.
- Verdict: not promoted. Not registered as a live agent.

**Not done**: CLI registration for the PPO agent, per-agent settings wiring, retraining.
