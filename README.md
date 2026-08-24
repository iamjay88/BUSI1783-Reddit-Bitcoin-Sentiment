# Reddit Sentiment and Bitcoin Price Volatility Before and After the US Spot Bitcoin ETF Approval

**BUSI 1783 — Business Analytics Project**
MSc Business Analytics, University of Greenwich

- **Student:** Jay Panchal (001495232)
- **Supervisor:** Dr. Martina Testori
- **Submission:** 25 August 2026

---

## Research question

Did the predictive relationship between Reddit sentiment and Bitcoin price volatility change
significantly following the approval of US spot Bitcoin ETFs in January 2024?

A comparative analysis of the pre-ETF (2022–2023) and post-ETF (2024–2025) periods, with the
regime cut-off set at **10 January 2024** — the first day of spot ETF trading.

## Headline finding

Lagged Reddit sentiment is a small but statistically significant negative predictor of next-day
Bitcoin volatility across the full period (β = −0.0095, p = 0.034). The sentiment × post-ETF
interaction term — the direct test of the research question — is **not significant**
(β = 0.0011, p = 0.856), indicating no detectable difference in the relationship between the two
periods once volatility clustering and posting volume are controlled.

The result is sensitive to the volatility window and is reported as weak and
specification-dependent rather than as a clean null. See Chapter 4 of the report.

---

## Data sources

| Series | Source | Notes |
|---|---|---|
| Bitcoin daily close (BTC-USD) | Yahoo Finance via the `yfinance` library | 1,460 daily observations, 1 Jan 2022 – 30 Dec 2025. Used instead of CoinGecko, whose free tier serves only the most recent 365 days. |
| r/Bitcoin historical posts | [Arctic Shift](https://arctic-shift.photon-reddit.com/) archive (compressed JSON-lines) | 295,336 posts with scoreable text. Used instead of PRAW, which cannot retrieve historical posts at scale; Pushshift has been restricted since 2023. |

**The raw Reddit archive is not included in this repository.** It is excluded for size reasons and
to avoid redistributing individual post content. The derived, fully aggregated day-level datasets
are included and are sufficient to reproduce every result in the report.

No individual Reddit user is identified. No usernames are retained in any dataset in this
repository. Only day-level aggregates enter the models.

---

## Repository contents

```
├── notebooks/
│   ├── 01_data_collection.ipynb      # yfinance price download; Arctic Shift archive load
│   ├── 02_sentiment_analysis.ipynb   # VADER scoring of 295,336 posts; daily aggregation
│   ├── 03_merge_and_features.ipynb   # returns, rolling volatility, merge, lags, regime label
│   └── 04_analysis_and_findings.ipynb# Models 1 & 2, diagnostics, robustness, Chow tests
├── data/
│   ├── bitcoin_prices_clean.csv      # 1,453 rows: date, price, return, volatility, regime
│   ├── daily_sentiment.csv           # 1,460 rows: date, sentiment_mean, post_volume, sentiment_std
│   └── analysis_dataset.csv          # 1,452 rows: merged, lagged — the modelling dataset
├── outputs/                          # figures used in the report
└── README.md
```

Run the notebooks in numerical order. Notebooks 03 and 04 read the CSVs in `data/`, so
**notebook 04 can be run on its own** to reproduce every statistic in the report without
re-downloading anything.

## Final analysis dataset

1,452 daily observations, 9 January 2022 – 30 December 2025: **731 pre-ETF days, 721 post-ETF days**.
All predictors are lagged by one day so that each day's volatility is explained only by information
available the previous day.

| Column | Description |
|---|---|
| `date` | Calendar date (UTC) |
| `price` | BTC-USD daily closing price |
| `return` | Daily percentage change in closing price |
| `volatility` | 7-day rolling standard deviation of daily returns (not annualised) |
| `regime` | `pre_etf` / `post_etf`, split at 10 Jan 2024 |
| `sentiment_mean` | Mean VADER compound score across that day's posts |
| `post_volume` | Number of r/Bitcoin posts that day |
| `sentiment_std` | Standard deviation of that day's compound scores (opinion dispersion) |
| `sentiment_lag1`, `volatility_lag1`, `volume_lag1`, `sentiment_std_lag1` | One-day lags of the above |

## Method

Primary specification — a single pooled OLS model estimated over all 1,452 observations, containing
a post-ETF dummy and a sentiment × dummy interaction:

```
volatility_t = β0 + β1·sentiment_{t-1} + β2·volatility_{t-1} + β3·dummy
             + β4·(sentiment_{t-1} × dummy) + β5·volume_{t-1} + β6·dispersion_{t-1} + ε_t
```

β4 is the direct test of the research question: it *is* the difference between the pre- and post-ETF
sentiment slopes, and carries its own standard error. Estimated with Newey–West (HAC) standard
errors at 7 lags, justified by a Breusch–Godfrey test (LM = 262.44, p < 0.001).

The 7-day volatility window was **pre-specified** before any model was estimated and is reported as
primary regardless of outcome. Five-, 14- and 30-day windows are reported as sensitivity analyses.

## Environment

Python 3.11 with:

```
pandas
numpy
yfinance
vaderSentiment
statsmodels
matplotlib
```

```bash
pip install pandas numpy yfinance vaderSentiment statsmodels matplotlib
```

## Academic integrity

This repository contains work submitted for assessment on BUSI 1783 at the University of Greenwich.
A Declaration of AI Use form accompanies the submission in line with the University's
[AI guidance for students](https://www.gre.ac.uk/aiguidance/students).
