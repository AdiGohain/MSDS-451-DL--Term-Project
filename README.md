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

## Aligning Data
Because financial assets had different trading calendars like holidays, futures roll dates, missing history, etc..
I incorporated this function to synchronize the commodity futures, equity time series, and my benchmark ETFs.
This made sure that all datasets shared a common calendar using index intersection. Forward filling helped in handling short gaps like holidays or missing futures data. Rows with all missing data are being dropped to preserve numerical integrity.
This was an important process to add because without it, the vectorized operations (weights × returns) were getting misaligned and producing incorrect results.

## Signal Generation

In my model, I use momentum signals from commodities.

A commodity is “bullish” when its price closes above its moving average. I am combining multiple commodities to create a broad commodity risk-on signal. I am doing so by incorporating two functions here:

1.  **compute_ma_signal()**
   * This one computes a binary indicator: 1 if price is above its moving average 0 is price is below This represents a short term commodity momentum.

2.  **aggregate_signal()**
   * This one combines all commodities by summing signals: If at least N commodities are bullish, the model enters equities. By doing so I am reducing noise and avoiding overreacting to single commodity volatility.
   * Because commodities are often leading indicators for industrial/equipment sector performance. I chose to use this rule to captures macroeconomic demand conditions without relying on equity prices directly.

3. **apply_persistence_and_hold()**
   * I added this function to implement realistic trading constraints to my raw signals: min_consec_days requires consecutive bullish days before entering a position. This is done in order to filters out whipsaw signals. min_hold_days enforces a mandatory holding period once a position is entered therefore preventing overtrading and reducing transaction costs. This mimics institutional trading mandates where churn is expensive. Internally, a finite-state loop steps day-by-day to extend a buy signal across required holding windows. This produces a more stable and realistic signal series for backtesting.


## Backtesting

The backtesting process is my core module where I am translates my trading signals into daily portfolio returns.

* I designed it to performs the following steps:
   * First computing the daily equity returns using percent change.
   * Then Lagging the signal by a day for preventing lookahead bias.
   * When signal equals 1, the algorithm invest equally across all equities
   * When signal = 0, the algorithm holds 0% exposure.

* For estimating transaction costs realistically, I am calculating turnover by measuring how much the portfolio changes day to day.
* Transaction cost model being applied daily to produce net returns.
   * Costs = turnover × (commission + slippage)

Gross and net cumulative equity curves are being calculated via cumulative products.
The output df then includes: gross_ret net_ret turnover pos (exposure) gross_eq net_eq
I am doing this primarily to prepares my results for easy plotting, analyzing, and exporting.



## Evaluation Metrics

For this part, I am summarized the strategy performance into key evaluation metrics:
- **Annualized return:** compounded daily net returns.
- **Annualized volatility:** daily standard deviation × √252.
- **Sharpe ratios:** risk-adjusted performance indicator.
- **Average turnover:** measure of trading intensity.
- **Total trades:** days when turnover > 0.
- **Days in market:** proxy for exposure and signal frequency.
- **Final net equity:** ending value of $1 invested.

Using these metrics, I am comparing strategy variants, MA windows, or parameter choices.


## Monte Carlo Simulation
I optimized the Programming_Assignment_02 using chat GPT to bring in the Monte Carlo simulation as a module in this project. I am running 5000 random portfolio simulations using annualized mean returns and annualized covariance matrix

Similar to the assignment, I am evaluating 2 cases:
1. Long-only portfolios
2. Long + short allowed (market-neutral) portfolios

For each simulated weight vector, I am computing and storing the expected annual return, expected volatility, and Sharpe ratios
I am also adding a scatter plot to visualize the comparison between long-only vs long–short allowed sets.

