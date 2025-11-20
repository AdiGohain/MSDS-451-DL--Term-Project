# MSDS 451-DL Term-Project

## Libraries Used:
- **NumPy:** vectorized numerical operations, especially for portfolio simulations.
- **Pandas:** time series handling, financial data structures, percent-change returns, rolling windows, etc.
- **Matplotlib & Seaborn:** visualization of strategy performance, risk metrics, and Monte Carlo results.
- **yfinance:** real-time historical price downloads for stocks, ETFs, and futures.
- **os & datetime:** directory management and date formatting for saved outputs.

## Data Ingestion Processs:
I defined the global tickers for my commodities, equities, and benchmarks. I did this in order to make make the system modular — changing tickers requires editing only a single block. I also decided to create directories OUTPUT_DIR and CURVES_DIR to create a clean file structure for project output, figures, and logs. This project can be run easily from any root folder and the code will automatically save project files by creating the **Output** folders.

I am also including the following ETF's in my mix to add diversification. Adding these uncorrelated ETF's with the four industry assets allows me to improve portfolio performance.

Here is the reasoning behind picking these ETF's: (These ETF's were suggested by Chat GPT)

* Asset Classes With Low or Negative Correlation to Industrials Long-Duration U.S. Treasuries (defensive macro hedge)
    * 20+ Year U.S. Treasuries **TLT**
    * 7–10 Year Treasuries **IEF**

* Commodities and Real Assets (Energy & Metals/ Non Agricultural)
    * Gold **GLD**
    * Broad Commodities (energy-weighted) **DBC**

* International Diversification / Developed Markets (non-U.S.)
    * Developed Markets ex-US **EFA**
    * Vanguard Developed ex-US **VEA**

* Market-Neutral / Low-Beta / Risk-Managed Strategies
    * Anti-Beta (long low beta, short high beta) **BTAL**
    * Multi-strategy, alt-risk premia **LALT**


I am incorporating a download_prices function that downloads historical price data from Yahoo Finance for the group of tickers assigned above. I was running into various issues where tickers had varying timelines as well as dataframes had format errors and therefore to mitigate this, I added logic to handle multi index futures data from Yahoo (ex. "Adj Close" nested under the ticker symbol).

I used Chat GPT to develop a logic to resolve:
- Skipping unavailable tickers instead of crashing the pipeline.
- Using adjusted close prices, which reflect corporate actions (splits, dividends).
- Ensuring all time indices are converted to DatetimeIndex, avoiding alignment bugs later.
- Collecting all valid series into a dictionary and concatenating them horizontally, to create a clean, consistent price matrix where each column corresponds to a single asset.
- Raising an error to prevent any failures if no data is successfully retrieved.


