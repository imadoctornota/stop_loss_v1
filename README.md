# Stop-loss backtest

This is the notebook behind my stop-loss analysis. It compares holding a stock through a fixed horizon with selling after a specified loss and putting the proceeds into SPY.

[Open the notebook](stop_loss_backtest.ipynb)

## Running it

Python 3.13 was used for the original run (with `yfinance` 1.7.0 and `pandas` 3.0.2). Install the dependencies and open the notebook:

```bash
python -m pip install -r requirements.txt
jupyter lab
```

Run the cells in order. By default `MAX_TICKERS = None`, which attempts the full current-listed universe. For a smaller test, set `MAX_TICKERS = 100` in the configuration cell. Fetching the full history can take a while and may hit provider rate limits. Cached histories are reused on subsequent runs.

## Method

- **Universe:** current U.S. equity listings from the Nasdaq Trader symbol directories, with ETFs and several non-common-share security types screened out. Available daily histories come from Yahoo Finance via `yfinance`.
- **Entries:** first observed close each calendar month for each stock.
- **Stop levels:** fixed losses of 5%, 10%, 15%, 20%, 30%, 40%, and 50% from entry (not trailing stops). A close below the threshold triggers a sale at the next available opening price.
- **After selling:** buy SPY at the corresponding open and hold it through the original endpoint.
- **Horizons:** 21, 63, 126, 252, and 756 SPY trading sessions. The main comparison uses a 20% stop and a 252-session horizon.
- **Returns:** adjusted prices account approximately for splits and distributions. Triggers use the split-adjusted, non-dividend-adjusted close. The $1/$5 historical price screens use a separately reconstructed as-traded quote.

The notebook compares raw observations with two price-history QC filters. It also reports several weighting approaches, clustered bootstrap intervals, time-period and entry-price sensitivity, extreme-return influence, and stock rebounds after stops execute.

## Limits

This is a **current-listed** stock universe, not a historical database containing delisted companies. It therefore has survivorship bias. The omitted-failure calculations in the notebook are sensitivity scenarios, not estimates of the true delisting effect.

The entry windows overlap. The backtest assumes execution at the next reported open and does not include transaction costs, taxes, or slippage. The QC screens are heuristics and may exclude genuine extreme moves. Results can change if the listing universe or historical prices are updated.

## Data and outputs

The notebook downloads data when run. Generated prices, ticker-level QC files, and event-level observations are stored under `sell_timing_work/`. Cross-stock summaries are written to `sell_timing_aggregates/`. Both folders are ignored by Git, and no market data or saved notebook outputs are included here.

The data retrieval uses `yfinance`; anyone running or reusing the analysis is responsible for following the applicable data-provider terms. The code is for research and education, not investment advice.
