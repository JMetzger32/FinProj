# FinProj

Predicting stock price movement by combining multiple independent signals into
one system: volatility, news sentiment, ARIMA, and a mix model blending them.
See `Technical Specs.pdf` for the full design.

## Status

**Phase 1 (data foundation) is done.** A working 50-ticker universe sampled
from the S&P 500, with 10 years of daily prices, macro indicators, and
two independent news sources feeding into a local SQLite database. No
modeling code yet — that's next.

## Setup

```bash
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

Create a `.env` file (gitignored) with:

```
FRED_API_KEY=...
NEWS_API_KEY=...
FINNHUB_API_KEY=...
```

## Usage

Run from the repo root with the venv active, in order:

```bash
python scripts/init_db.py            # create data/stockproject.db
python scripts/build_universe.py     # sample a 50-ticker working universe
python scripts/pull_prices.py        # 10y daily OHLCV via yfinance
python scripts/pull_macro.py         # fed funds rate, 10y yield, VIX via FRED
python scripts/pull_news.py          # recent headlines via NewsAPI
python scripts/pull_finnhub_news.py  # recent headlines via Finnhub
```

Each script is idempotent and safe to rerun. See `CLAUDE.md` for schema
details, per-source rate-limit notes, and other gotchas.
