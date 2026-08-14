# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project status

No application code yet — the repo currently has the Python environment and tooling scaffolded (venv, requirements, .env), but no source modules. The user will be feeding additional context and requirements incrementally as the project develops, rather than up front.

## Project intent

Predict stock prices by combining multiple signals/approaches into one system:
- Volatility
- News sentiment
- ARIMA
- A mix model (combining the above)

## Setup

- Python 3.12, virtualenv at `venv/` (gitignored).
- Activate with `source venv/bin/activate`, install deps with `pip install -r requirements.txt`.
- `requirements.txt`: pandas, numpy, statsmodels (ARIMA), scikit-learn, requests, python-dotenv, finnhub-python.
- `.env` (gitignored) holds `FINNHUB_API_KEY` — load it with python-dotenv rather than hardcoding.

## MCP tooling

A `finnhub` HTTP MCP server (`https://mcp.finnhub.io/mcp`) is configured locally for this project, for pulling market data.

## Guidance for future Claude instances

- Do not assume any particular language, framework, or architecture until the user specifies one.
- As real structure (build commands, test commands, module layout) is established, update this file to reflect it — replace this placeholder content rather than appending to it indefinitely.
