# Crypto Arbitrage Strategy: BTC-EUR/USD Spread

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1fWBl0cES-AXnWlrExQsmB4R__Zqb9zci?usp=sharing)

This repository contains a Python-based analysis and simulation of a statistical arbitrage strategy. The strategy aims to exploit temporary pricing inefficiencies between Bitcoin priced in US Dollars (`BTC-USD`) and Bitcoin priced in Euros (`BTC-EUR`).

The core of the strategy involves calculating an "implied" `EUR/USD` exchange rate from the two Bitcoin pairs and trading the spread against the actual `EUR/USD` spot rate. The project includes historical data backtesting, rigorous statistical validation of the strategy's core assumptions, and a Monte Carlo simulation to assess its forward-looking robustness.

## Strategy Overview

Statistical arbitrage strategies rely on the principle of **mean reversion**—the idea that a price or value will tend to return to its historical average over time.

In this project, we create a synthetic instrument, the "Spread," defined as:
$$\text{Spread} = \left( \frac{\text{BTC-USD Price}}{\text{BTC-EUR Price}} \right) - \text{Actual EURUSD Rate}$$

Theoretically, the implied rate (`BTC-USD / BTC-EUR`) should be very close to the actual `EURUSD=X` rate. When they diverge, the strategy assumes this gap is temporary and will close. By taking positions when the spread widens and closing them when it reverts to its mean, we can potentially generate profit.

## Statistical Validation: The Regime Change

A mean-reversion strategy is only viable if the underlying signal (the "Spread") is **stationary**. A stationary time series is one whose statistical properties (like mean and variance) are constant over time.

We use two formal statistical tests to validate this assumption:

1.  **Augmented Dickey-Fuller (ADF) Test**: Tests for non-stationarity.
2.  **Kwiatkowski-Phillips-Schmidt-Shin (KPSS) Test**: Tests for stationarity.

A key finding of this analysis was that over the full 5-year historical period, the tests gave conflicting results. However, upon splitting the data at a suspected **regime change date (`2022-11-01`)**, both the ADF and KPSS tests provided strong, concurring evidence that the Spread **is stationary** in the more recent market environment. This provides a solid statistical foundation for the strategy's viability going forward.

## How It Works

The entire process is orchestrated within the single Google Colab notebook linked above.

### 1\. Data Fetching

Historical daily closing prices for `BTC-USD`, `BTC-EUR`, `EURUSD=X`, and the DXY (`DX-Y.NYB`) are downloaded from Yahoo Finance using the `yfinance` library.

### 2\. Spread Calculation

The core "Spread" series is calculated daily as described above.

### 3\. Signal Generation

Trading signals are generated using a method similar to Bollinger Bands:

  - A **rolling mean** and **rolling standard deviation** of the Spread are calculated over a defined lookback window.
  - When the Spread moves above the `Upper Band` (mean + N \* std\_dev), a **Short signal** (-1) is generated.
  - When the Spread falls below the `Lower Band` (mean - N \* std\_dev), a **Long signal** (1) is generated.
  - The position is held until a new signal is generated.

### 4\. Backtesting

The strategy is simulated over the historical data. The backtest calculates daily profit and loss, accounts for estimated transaction costs, and generates a final equity curve to visualize overall performance.

### 5\. Monte Carlo Simulation

To stress-test the strategy and understand the range of potential future outcomes, a Monte Carlo simulation is performed. It runs thousands of randomized trading scenarios based on the strategy's historical daily returns. This analysis produces:

  - A distribution of potential final returns.
  - The probability of incurring a loss over a given period.
  - "Best-case" (95th percentile) and "worst-case" (5th percentile) performance paths.

## How to Use

### Prerequisites

The notebook is self-contained and runs on Google Colab, which has all necessary libraries pre-installed. If you wish to run it locally, you will need Python 3 and the following libraries. You can install them using pip:

```bash
pip install yfinance pandas numpy matplotlib seaborn statsmodels
```

### Configuration

All key parameters can be easily modified in the second cell of the notebook:

  - `TICKERS`: The list of financial assets to analyze.
  - `PERIODE_DATA`: The duration of historical data to download (e.g., `'5y'`).
  - `ROLLING_WINDOW`: The lookback period for the moving average.
  - `ENTRY_THRESHOLD`: The number of standard deviations for trade entry.
  - `INITIAL_CAPITAL`: The starting capital for the backtest.
  - `TRANSACTION_COST`: The estimated commission per trade.

### Execution

1.  Click the "Open in Colab" badge at the top of this document to open the notebook.
2.  Set your desired parameters in the configuration cell.
3.  Run the cells sequentially from top to bottom (`Runtime` \> `Run all`). The notebook will automatically perform all calculations and display the results.

## Results & Key Findings

  - **Backtest Performance**: The historical backtest demonstrates the strategy's performance based on past data, including an equity curve and key metrics like the Sharpe Ratio, Max Drawdown, and Win Rate.
  - **Statistical Significance**: The strategy is underpinned by strong statistical evidence of mean-reversion (stationarity) in the current market regime (post-Nov 2022), as confirmed by both ADF and KPSS tests.
  - **Future Viability**: The Monte Carlo simulation provides a probabilistic forecast of future performance, showing a median expected return and the range of likely outcomes, which helps in assessing risk.

## Disclaimer

This project is for educational and research purposes only. The information and code provided here are not financial advice. Trading financial markets involves significant risk, and past performance is not indicative of future results. Always conduct your own research and consult with a qualified financial advisor before making any investment decisions.
