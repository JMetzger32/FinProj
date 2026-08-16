# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project status

Phase 1 (data foundation) is built: SQLite schema, S&P 500 universe sampling, and
four ingestion sources (prices, macro, and two independent news feeds). No
features/modeling code yet — `features` table exists in the schema but nothing
populates it. Reference `Technical Specs.pdf` (repo root) for the full phased plan.

## Project intent

Predict stock prices by combining multiple signals/approaches into one system:
- Volatility
- News sentiment
- ARIMA
- A mix model (combining the above)

## Map

- `src/db.py` — `get_connection()` (opens `data/stockproject.db`, gitignored) +
  `init_schema()` (runs `src/schema.sql`, idempotent `CREATE TABLE IF NOT EXISTS`).
  Every script calls both at startup — safe to run any script cold.
- `src/schema.sql` — tables: `sp500_constituents`, `universe` (the sampled
  50-ticker working set, FK to constituents), `prices` (OHLCV, PK
  ticker+date), `features` (unused so far — RSI/MACD/bollinger/OBV/ATR/SMA/EMA
  columns already defined for the next phase), `macro` (fed funds, 10y yield,
  VIX, PK date), `news` (PK `provider`+`ticker`+`url`, so NewsAPI and Finnhub
  rows coexist without colliding).
- `src/config.py` — loads `.env`, exposes `FRED_API_KEY`/`NEWS_API_KEY`/`FINNHUB_API_KEY`;
  raises immediately if any is missing.
- `src/universe/` — `fetch_sp500.py` scrapes the Wikipedia S&P 500 constituent
  table; `sample_universe.py` draws a seeded 50-ticker sample into `universe`.
- `src/ingest/` — `prices.py` (yfinance, batched/retrying, 10y default),
  `macro.py` (FRED, 10y daily backfill), `news.py` (NewsAPI), `finnhub_news.py`
  (Finnhub `/company-news`, second independent source).
- `scripts/` — thin CLI wrappers around the above: `init_db.py`,
  `build_universe.py`, `pull_prices.py`, `pull_macro.py`, `pull_news.py`,
  `pull_finnhub_news.py`. Run any of them from the repo root with the venv
  active; each prints a summary (rows inserted/failed) on completion.

## Setup

- Python 3.12, virtualenv at `venv/` (gitignored).
- Activate with `source venv/bin/activate`, install deps with `pip install -r requirements.txt`.
- `requirements.txt`: pandas, numpy, statsmodels (ARIMA), scikit-learn, requests,
  python-dotenv, finnhub-python, yfinance, lxml.
- `.env` (gitignored) holds `FRED_API_KEY`, `NEWS_API_KEY`, `FINNHUB_API_KEY` —
  loaded via `src/config.py`, never hardcode these.

## Commands

- `python scripts/init_db.py` — create `data/stockproject.db` from `src/schema.sql`.
- `python scripts/build_universe.py [--sample-size N] [--seed N] [--skip-refresh]` —
  refresh S&P 500 list from Wikipedia, sample the working universe.
- `python scripts/pull_prices.py [--period 10y]` — bulk OHLCV via yfinance.
- `python scripts/pull_macro.py [--years 10]` — FRED macro series, idempotent upsert.
- `python scripts/pull_news.py [--window-days N] [--batch 1|2|auto]` — NewsAPI headlines.
- `python scripts/pull_finnhub_news.py [--window-days N]` — Finnhub headlines, no batching needed.

## Rules (learned the hard way)

- **NewsAPI's free tier is 50 requests per 12 hours, not 100/day.** Pulling the
  full 50-ticker universe in one run leaves no headroom for reruns. `pull_news.py`
  splits into two fixed 25-ticker halves via `--batch {1,2,auto}`; `auto` picks a
  half by UTC hour so a plain twice-daily cron covers all 50/day. Finnhub's
  `/company-news` is free-tier at 60 req/min, so it pulls the full universe in
  one run with no batching.
- **Query NewsAPI on the quoted company name (`qInTitle`), not the bare ticker
  symbol** — ticker-symbol queries returned largely unrelated articles.
- **`news` is keyed `(provider, ticker, url)`**, not `(ticker, url)` — this lets
  NewsAPI and Finnhub sentiment be computed and compared independently as two
  distinct sources rather than merged into one. If you extend `news` again, keep
  `schema.sql` migrations non-destructive (`CREATE TABLE IF NOT EXISTS`); the
  provider-column change itself required a one-time manual drop/recreate of the
  dev table since old rows weren't migrated.

## MCP tooling

A `finnhub` HTTP MCP server (`https://mcp.finnhub.io/mcp`) is configured locally for this project, for pulling market data.

## Guidance for future Claude instances

- Do not assume language/framework choices beyond what's here without checking
  with the user — Phase 1 is data ingestion only; modeling approach for
  volatility/ARIMA/sentiment/mix-model is not yet built.
- Keep this file in sync with reality as each phase lands — update it in the
  same commit as the work, not after.
