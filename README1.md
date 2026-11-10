# Heikin-Ashi Triple Barrier Intelligent Forex Trading System

> A research-first Forex framework combining Heikin-Ashi reversal
> signals, volatility-adjusted Triple Barrier labeling, and
> machine-learning-based signal selection.

**Project status:** Research prototype; demo-only integration\
**Instruments:** EURUSD, GBPUSD, USDJPY\
**Primary timeframe:** Daily (D1)\
**Live trading:** Disabled

## Overview

This project studies how event-labeling methodology influences
machine-learning-based selection of technical trading signals in daily
Forex markets.

It combines: - Heikin-Ashi (HA) reversal signals as candidate events. -
ATR-based Triple Barrier labels for profit-taking, stop-loss, and
time-based outcomes. - Fixed-horizon and fixed-percentage labeling
baselines. - Random Forest and Logistic Regression models, compared with
a no-ML baseline. - Chronological train/validation/test evaluation with
event purging and embargo. - Backtesting, transaction-cost accounting,
and out-of-sample (OOS) reporting. - MetaTrader 5 (MT5) data/terminal
integration components and a separately implemented MQL5 baseline EA.

This is an empirical research system, **not a claim of a profitable
trading strategy**.

## Research question

**How does the choice of event-labeling methodology influence
machine-learning-based selection of technical trading signals in daily
Forex markets?**

## Research configuration

  Parameter                  Configuration
  -------------------------- ---------------------------------------
  Primary instrument         EURUSD
  Validation instruments     GBPUSD, USDJPY
  Timeframe                  Daily (D1)
  Signal                     Completed-candle Heikin-Ashi reversal
  Volatility measure         ATR(14)
  Profit-taking barrier      2 × ATR
  Stop-loss barrier          1 × ATR
  Vertical barrier           10 trading bars
  Entry assumption           Next bar's executable opening price
  Same-bar PT/SL ambiguity   Conservative stop-first

ATR is snapshotted at signal time and barriers are anchored to the
executable entry fill. Long exits are evaluated on Bid; short exits
account for Ask using spread. The vertical barrier exits at the
configured time limit. These are research assumptions, not trading
recommendations.

## Experiment suite

  -----------------------------------------------------------------------
  Experiment              Method                  Purpose
  ----------------------- ----------------------- -----------------------
  E0                      HA, no-ML baseline      Raw signal baseline

  E1                      Fixed-horizon labels +  Fixed-time outcome
                          ML                      definition

  E2                      Fixed-percentage        Percentage-based
                          barriers + ML           barriers

  E3                      ATR Triple Barrier + ML Volatility-adjusted
                                                  event outcomes

  E4                      ATR Triple Barrier +    Additional feature
                          expanded features       information

  E5                      Standard-candle         Compare signal source
                          reversal + ATR Triple   
                          Barrier + ML            
  -----------------------------------------------------------------------

## Features and models

Features include HA body/wicks/direction/streak, ATR and normalized ATR,
RSI, ADX, EMA50/EMA200 relationships, realized volatility, momentum,
signal direction, and market-regime descriptors.

Models: - **Random Forest:** primary nonlinear model. - **Logistic
Regression:** interpretable baseline. - **No-ML baseline:** raw signal
comparison.

Preprocessing must be fitted on training data only. Validation is used
for threshold selection; the test partition remains held out.

## Architecture

``` text
Historical Forex Data
        |
        v
Data Ingestion and Validation
        |
        v
Daily Candle / Feature Pipeline
        |
        v
Heikin-Ashi Candidate Signals
        |
        v
Event Labeling
  |        |          |
Fixed   Fixed %     ATR Triple
Horizon  Barriers    Barrier
  |        |          |
  +--------+----------+
           |
           v
Chronological Split + Purge / Embargo Audit
           |
           v
Model Training and Signal Selection
           |
           v
Backtest + Transaction-Cost Accounting
           |
           v
OOS Metrics, Reports, and Analysis

MT5 terminal <--> Python MT5 bridge
MQL5 EA      --> Independent baseline / Strategy Tester
```

### Main components

-   **Python research engine:** data processing, features, labels, model
    training, backtesting, experiment orchestration, and reports.
-   **MT5 Python bridge:** access to a running terminal for market data
    and demo-account context.
-   **MQL5 Expert Advisor:** separate HA/ATR baseline for MT5 Strategy
    Tester validation.
-   **SQLite:** experiment, signal, approval, and trade-event logging
    where enabled.
-   **Telegram:** optional notification/approval workflow; authorization
    must fail closed.
-   **CLI/configuration:** repeatable experiments and safety controls.

The Python engine is authoritative for ML research. The MQL5 EA is a
separate baseline; it does not imply that Python ML inference runs
inside MT5.

## Research-integrity controls

The implementation includes: - Future-feature leakage regression tests;
undefined feature values are preserved rather than backfilled from
future observations. - Explicit OOS evaluation states:
`evaluated_with_trades`, `evaluated_zero_trades`, `not_evaluated`,
`invalid`, and `legacy_unverified`. - Full-period metrics kept separate
as diagnostics; they are not substituted for missing OOS metrics. -
Chronological partitions, event-time validation, purging, embargo, and
split audit metadata. - Training-only preprocessing and validation-only
threshold selection. - Label/backtest convention alignment for entry
timing, barriers, spread treatment, and same-bar ambiguity.

## Current verification status

The latest supplied remediation report records **105 Python tests
passing after P0-4**. P0-1 feature leakage, P0-2 OOS reporting, P0-3
split consistency, and P0-4 label/execution consistency were reported
implemented and tested.

Important qualifications: - The production experiment flow currently
uses a **single chronological train/validation/test split**. - A purged
walk-forward fold generator exists but is **not integrated** into the
experiment runner, ML pipeline, or CLI. Do not claim completed
walk-forward model evaluation. - Previously inspected OOS samples were
small and expectancy confidence intervals included zero; a robust
trading edge has not been established. - MQL5 formula-level parity has
been reviewed, but EA compilation and MT5 Strategy Tester validation are
not yet independently verified. - End-to-end MT5 Python bridge and
Telegram workflows still require environment-level verification.

Test counts describe the audited development state; rerun tests before
relying on results.

## Getting started

### Requirements

-   Python **3.11.x** (`>=3.11,<3.12`).
-   Windows recommended for MT5 terminal and MQL5 workflow.
-   MetaTrader 5 desktop terminal for MT5-specific steps.
-   Git.

### Clone

``` bash
git clone https://github.com/<your-username>/Heikin-Ashi-Triple-Barrier-Intelligent-Forex-Trading-System.git
cd Heikin-Ashi-Triple-Barrier-Intelligent-Forex-Trading-System
```

### Python environment (Windows PowerShell)

``` powershell
py -3.11 -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
```

Install dependencies using the repository's actual dependency manifest.
If `requirements.txt` is present:

``` powershell
pip install -r requirements.txt
```

If the repository uses `pyproject.toml` or another manager, follow that
manifest instead.

### Tests

If the repository uses pytest:

``` powershell
python -m pytest -q
```

### MT5 setup

1.  Install and open MetaTrader 5.
2.  Create a free demo account with a supported demo server.
3.  Open EURUSD, GBPUSD, and USDJPY charts and select D1.
4.  Increase chart/history limits and retrieve sufficient historical
    bars.
5.  Review EA inputs and compile the MQL5 EA in MetaEditor.
6.  Run it only in MT5 Strategy Tester after reviewing the test
    configuration.
7.  Connect the Python bridge only when the terminal is logged into the
    intended demo account.

A demo account is not required for every offline research component.
Historical depth and feed characteristics depend on the data provider.

## Safety defaults

-   `LIVE_TRADING=false`
-   `DEMO_EXECUTION=false` by default.
-   Manual confirmation is required for any optional demo order
    workflow.
-   Verify account type before any execution path.
-   Apply risk, spread, position-count, and daily-loss checks before any
    proposed order.
-   Telegram authorization must fail closed if the allowlist is absent
    or invalid.
-   Do not run Python and the MQL5 EA as competing execution engines.
-   Never use live funds to reproduce this academic research.

Never commit `.env` files, passwords, API keys, Telegram tokens, account
credentials, or other secrets.

## Reproducibility checklist

-   Record data source, retrieval date, symbol, timeframe, timezone, and
    bar count.
-   Check missing bars, duplicate timestamps, and candle boundaries.
-   Record the D1 construction convention, including New York-close
    handling where used.
-   Freeze configuration and random seeds.
-   Run the complete test suite.
-   Verify feature availability at signal time.
-   Audit chronological partitions, purged events, and embargo
    boundaries.
-   Confirm preprocessing is fitted on training data only.
-   Keep validation threshold selection separate from test evaluation.
-   Report OOS trade count, costs, uncertainty, and metrics.
-   Preserve zero-trade and not-evaluated states; never replace missing
    OOS results with full-period metrics.
-   Archive configuration, data version, logs, and reports.

## Metrics

**Classification:** precision, recall, F1, ROC-AUC / PR-AUC where
appropriate, calibration, confusion matrix, and selected-signal count.

**Trading simulation:** net return after modeled costs, Sharpe, Sortino,
maximum drawdown, profit factor, expectancy, trade count, holding
period, spread/commission/slippage assumptions, and uncertainty
intervals.

Only report metrics supported by the evaluation status and sample size.

## Known limitations

1.  Walk-forward evaluation is not integrated; the production flow uses
    one chronological split.
2.  No robust profitable edge has been established.
3.  MQL5 compilation and Strategy Tester validation remain outstanding.
4.  MT5 Python bridge and Telegram integration require end-to-end
    verification.
5.  Broker feeds, spreads, commissions, swaps, execution rules, and
    historical coverage vary.

## Responsible-use statement

Provided for academic research, engineering experimentation, and
demo-environment validation. This software is not investment advice, a
recommendation to buy or sell currencies, or a guarantee of future
performance. Forex and leveraged products involve substantial risk.
Users are responsible for complying with applicable laws, broker terms,
and platform rules.

## Academic use

When citing or describing this project, state the exact data source,
labeling rules, split methodology, execution assumptions, costs, and
limitations used. Do not present unverified integrations or uncompleted
walk-forward evaluation as completed work.
