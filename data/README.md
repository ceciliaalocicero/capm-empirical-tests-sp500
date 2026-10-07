# Data

No data files are committed to this repository. The notebook is saved **with its outputs**, so every table and chart can be read on GitHub without running it. To re-run the analysis, obtain the two datasets below and place them in `data/raw/` (this folder is excluded from Git by `.gitignore`).

```
data/
├── README.md
└── raw/                              # create locally, never commit
    ├── Data_Stock.csv                # eight-stock price file (Sections 2–5)
    └── crsp_sp500_2007_2023.csv.gz   # CRSP daily panel (Section 6)
```

## 1. Eight-stock price file (Sections 2–5)

**Original source:** course-provided file; not redistributable here.

**Expected format:** CSV, one row per trading day, columns:

| Column | Description |
|--------|-------------|
| `Date` | Trading date (YYYY-MM-DD) |
| `AAPL`, `BA`, `T`, `MGM`, `AMZN`, `IBM`, `TSLA`, `GOOG` | Daily closing price, split-adjusted, **not** dividend-adjusted |
| `sp500` | S&P 500 price index level |

**Period:** 12 January 2012 to 11 August 2020 (2,159 trading days).

**Rebuilding an equivalent file from public data (approximate).** Closing prices that are not dividend-adjusted can be downloaded with the unofficial `yfinance` package (`pip install yfinance`). Price *levels* will differ because Yahoo applies all later stock splits, but daily *returns* should be very close. Expect small numerical differences from the committed results.

```python
import yfinance as yf

tickers = ["AAPL", "BA", "T", "MGM", "AMZN", "IBM", "TSLA", "GOOG", "^GSPC"]
px = yf.download(tickers, start="2012-01-12", end="2020-08-12",
                 auto_adjust=False)["Close"]
px = px.rename(columns={"^GSPC": "sp500"}).reset_index()
px.to_csv("data/raw/Data_Stock.csv", index=False)
```

## 2. CRSP daily S&P 500 panel (Section 6)

**Source:** CRSP US Stock Database via Wharton Research Data Services (WRDS). Requires an institutional WRDS subscription. **CRSP data may not be redistributed**, including on GitHub.

**Extract used:** daily records for all stocks that were S&P 500 constituents on each date, 3 January 2007 to 29 December 2023 (~2.15 million stock-days, 873 PERMNOs).

**Required columns (CRSP CIZ / v2 format):**

| CRSP field | Renamed to | Description |
|------------|-----------|-------------|
| `DlyCalDt` | `date` | Calendar date |
| `PERMNO` | `permno` | Permanent security identifier |
| `Ticker` | `ticker` | Ticker on that date |
| `DlyPrc` | `price` | Daily price |
| `DlyRet` | `ret` | Daily total return (incl. dividends) |
| `DlyCumFacPr` | `adj_factor` | Cumulative price adjustment factor |
| `DlyCap` | `mcap` | Market capitalization (USD thousands) |
| `DlyPrcFlg` | `price_flag` | Price flag (TR = trade, BA = bid/ask average, DA/DP = delisting) |

**How to obtain it:** in WRDS, use the CRSP daily stock file together with the S&P 500 constituents list, restrict each stock to the days on which it was an index member, select the fields above for 2007–2023, and export as a gzip-compressed CSV named `crsp_sp500_2007_2023.csv.gz`. The exact WRDS query or web-query settings used for the original extract are not available; the description above (fields, universe and period) is sufficient to rebuild an equivalent file.

**Cleaning applied in the notebook:** zero prices on delisting days set to missing; rows with no return dropped (77 rows); delisting-day returns kept to avoid survivorship bias.
