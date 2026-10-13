# Stop-loss backtest

If a stock falls 20% after you buy it, should you sell and move the money into SPY, or keep holding? This repository contains the notebook behind my video testing that question on about 1 million simulated purchases of U.S. stocks since 1993.

[![Do Stop-Losses Work? I Tested 1 Million Stock Trades](https://img.youtube.com/vi/8OoMhjhLIB8/hqdefault.jpg)](https://youtu.be/8OoMhjhLIB8)

**Video:** [Do Stop-Losses Work? I Tested 1 Million Stock Trades](https://youtu.be/8OoMhjhLIB8) · **Notebook:** [stop_loss_backtest.ipynb](stop_loss_backtest.ipynb)

## Key findings

The primary test uses a 20% stop and compares the two choices one year after entry. Positive numbers mean the stop did better than holding, in percentage points. Unless noted, results use the hard-QC sample, which drops 22 tickers with severe price-history discontinuities.

- **When a stop fires, selling usually wins.** Selling beat holding in 57% of the 488,249 triggered stops, with a median advantage of +4.3 points.
- **A few huge rebounds decide the average.** About 3% of stopped stocks more than doubled by the one-year mark. They pull the mean for triggered stops down to −2.9 points, and removing just the top 0.5% of rebounders flips it to +1.1.
- **The overall answer depends on how you average.** Across all entries, the event-weighted mean favors holding by 1.4 points, while weighting every stock equally favors the stop by 1.8 points (95% CI 0.8 to 2.6).
- **Much of holding's edge comes from sub-$5 stocks and the 2000s.** Restricting to entries priced at $5 or more brings the event-weighted mean to +0.1. By era, holding's advantage is concentrated in 2000–2009 (−6.0); entries since 2020 slightly favor the stop (+0.8).

In short, a 20% stop trades a frequent, modest gain for occasionally missing a large rebound. Whether it helps depends on how much weight those rare rebounds get.

## Primary result by weighting and QC sample

20% stop, 1-year horizon. Mean stop advantage in percentage points, with 95% cluster-bootstrap intervals in brackets where computed.

| Weighting | Raw | Hard QC | Broad QC |
|---|---:|---:|---:|
| Every entry equally | −3.4 | −1.4 | +0.4 |
| Every stock equally | +0.3 [−2.0, 2.0] | +1.8 [0.8, 2.6] | +4.8 [4.2, 5.3] |
| Every ticker-year equally | −3.2 | −1.2 | +0.6 |
| Every calendar year equally | −4.0 [−7.2, −1.6] | −2.1 [−3.8, −0.4] | −0.5 [−1.8, 0.8] |
| Entries / stocks | 1,045,080 / 5,437 | 1,039,693 / 5,415 | 902,847 / 4,805 |

Each weighting answers a slightly different question: the average purchase, the average stock, or the average year. Stock-level intervals resample tickers, and year-level intervals resample calendar years. Medians are close to zero because more than half of entries never hit the stop, so the two choices end up identical.

## What happens after the 20% stop fires

Hard-QC sample, grouped by the stock's buy-and-hold return one year after entry.

| Buy-and-hold return at 1 year | Share of triggered stops | Selling beat holding | Mean stop advantage |
|---|---:|---:|---:|
| Below 0% | 74.5% | 75.3% | +17.8 |
| 0% to +25% | 13.6% | 9.0% | −18.3 |
| +25% to +100% | 8.6% | 0.4% | −56.3 |
| Above +100% | 3.2% | 0.1% | −272.8 |
| **All triggered stops (488,249)** | **100%** | **57.4%** | **−2.9 (median +4.3)** |

The mean is sensitive to the largest rebounds. Removing the top 0.1% of rebounders moves it to −1.0, the top 0.5% to +1.1, the top 1% to +2.6, and the top 5% to +7.9.

## Robustness checks

- **Entry price.** With entries priced at $5 or more, the stop advantage is +0.1 (every entry), +8.0 (every stock), +1.1 (every ticker-year), and −0.3 (every year).
- **Era.** Event-weighted, the advantage is −1.4 before 2000, −6.0 in 2000–2009, −0.1 in 2010–2019, and +0.8 from 2020 on.
- **Distribution-robust diagnostics.** Across all entries, a 1% two-sided trimmed mean gives +1.3 points, and the mean log wealth ratio (log of stop-strategy wealth over buy-and-hold wealth) is +0.073. Both favor the stop, consistent with a few giant winners driving the arithmetic mean.
- **Ticker influence.** In the raw sample, two tickers account for about 31% of all negative contribution to the mean. Both fail the hard-QC screen.
- **QC screens.** Hard QC removes 22 tickers (0.4%) using explicit rules for extreme one-day jumps, split dislocations, discontinuities across data gaps, and implausible multi-year returns. Broad QC removes 632 (10.8%) and is an aggressive sensitivity bound, not proof that those histories are wrong.

The notebook reports every combination of 7 stop levels and 5 horizons; the tables above cover the primary test only.

## Method

- **Universe:** 5,887 current U.S. listings from the Nasdaq Trader symbol directories, after screening out ETFs and several non-common-share security types. Of these, 5,854 had usable daily histories from Yahoo Finance (via `yfinance`) that produced entries.
- **Entries:** first observed close each calendar month for each stock, from January 1993 through September 2026. Across all stops and horizons this produces 36.7 million rows, stored as Parquet and queried with DuckDB.
- **Stop levels:** fixed losses of 5%, 10%, 15%, 20%, 30%, 40%, and 50% from entry (not trailing stops). A close below the threshold triggers a sale at the next available opening price.
- **After selling:** buy SPY at the corresponding open and hold it through the original endpoint.
- **Horizons:** 21, 63, 126, 252, and 756 SPY trading sessions. The main comparison uses a 20% stop and a 252-session horizon.
- **Returns:** adjusted prices account approximately for splits and distributions. Triggers use the split-adjusted, non-dividend-adjusted close. The $1/$5 historical price screens use a separately reconstructed as-traded quote.
- **Inference:** entries from the same stock or the same year are correlated, so confidence intervals come from cluster bootstraps that resample tickers or calendar years.

The notebook also includes a common-cohort check across horizons, rolling-start and annual results, tail summaries, and a survivorship tipping-point calculation.

## Limits

This is a **current-listed** stock universe, not a historical database containing delisted companies. It therefore has survivorship bias, which can favor holding because delisted failures are missing. The omitted-failure calculations in the notebook are sensitivity scenarios, not estimates of the true delisting effect.

The entry windows overlap. The backtest assumes execution at the next reported open and does not include transaction costs, taxes, or slippage. The QC screens are heuristics and may exclude genuine extreme moves. Results can change if the listing universe or historical prices are updated; the numbers above come from a run with prices through October 6, 2026.

## Running it

Python 3.13 was used for the original run (with `yfinance` 1.7.0 and `pandas` 3.0.2). Install the dependencies and open the notebook:

```bash
python -m pip install -r requirements.txt
jupyter lab
```

Run the cells in order. By default `MAX_TICKERS = None`, which attempts the full current-listed universe. For a smaller test, set `MAX_TICKERS = 100` in the configuration cell. Fetching the full history can take a while and may hit provider rate limits. Cached histories are reused on subsequent runs.

## Data and outputs

The notebook downloads data when run. Generated prices, ticker-level QC files, and event-level observations are stored under `sell_timing_work/`. Cross-stock summaries are written to `sell_timing_aggregates/`. Both folders are ignored by Git, and no market data or saved notebook outputs are included here.

The data retrieval uses `yfinance`; anyone running or reusing the analysis is responsible for following the applicable data-provider terms. The code is for research and education, not investment advice.
