# README — Nifty50 Closing Prices Analysis

**Project:** Experiments on Nifty50 closing prices — EDA, mutual-fund-style portfolio construction, and expected-return simulation.

**Source file:** `experiments.ipynb` (analysis performed on `nifty50_closing_prices.csv`).

---

## 1. Problem statement

Analyze historical closing prices for Nifty50 constituents to:

* Identify high-growth, low-risk companies.
* Construct a simple mutual-fund-style portfolio that balances return and volatility.
* Estimate expected future value of a regular monthly investment into that portfolio.

All insights and steps below are taken from and reproduce the analysis in `experiments.ipynb`.

---

## 2. Data

* **File:** `nifty50_closing_prices.csv` (daily closing prices indexed by `Date`).
* **Time span:** The notebook uses the full date range present in the CSV (converted to `datetime` and set as index).
* **Cleaning:** forward-fill (`ffill`) is used to handle missing values.

---

## 3. Preprocessing

* Convert `Date` column to `datetime` and set as index.
* Forward-fill missing values: `df.fillna(method='ffill', inplace=True)`.
* Compute daily percent changes to obtain growth rates: `df.pct_change()`.
* Compute per-symbol volatility as the standard deviation of price time series: `df.std()`.
* Compute ROI per symbol as `(final_price - initial_price)/initial_price * 100`.

---

## 4. Exploratory Data Analysis (key findings)

### Top indicators (examples from the notebook):

* **Most volatile tickers (by price std dev)** — top 5:

| Rank | Ticker     | Volatility (std dev) |
| ---- | ---------- | -------------------: |
| 1    | BAJAJ-AUTO |               659.81 |
| 2    | SHREECEM   |               429.92 |
| 3    | BAJFINANCE |               306.66 |
| 4    | DIVISLAB   |               247.67 |
| 5    | HEROMOTOCO |               247.09 |

* **Highest average daily growth rate (mean % change) — top 5:**

| Rank | Ticker     | Avg daily growth (%) |
| ---- | ---------- | -------------------: |
| 1    | BAJAJ-AUTO |               0.8834 |
| 2    | BAJAJFINSV |               0.7917 |
| 3    | BHARTIARTL |               0.7352 |
| 4    | DIVISLAB   |               0.6349 |
| 5    | HEROMOTOCO |               0.6022 |

* **Top ROI (price appreciation from first to last day) — top 5:**

| Rank | Ticker     | ROI (%) |
| ---- | ---------- | ------: |
| 1    | BAJAJ-AUTO |   22.11 |
| 2    | BAJAJFINSV |   19.64 |
| 3    | BHARTIARTL |   18.12 |
| 4    | DIVISLAB   |   15.40 |
| 5    | HEROMOTOCO |   14.66 |

> *Note:* numbers above are computed from the provided CSV and match the notebook's computations.

---

## 5. Mutual-fund-style portfolio construction (methodology)

To build a small portfolio that aims for **higher-than-median ROI** while keeping **below-median volatility**, the notebook applies the following selection rule:

* Compute the median ROI across all tickers: `roi_threshold = roi.median()`.
* Compute the median volatility: `volatility_threshold = volatility['Volatility'].median()`.
* **Select tickers where `roi > roi_threshold` AND `volatility < volatility_threshold`.**

**Selected tickers (examples from this dataset):**

* HDFCBANK
* ICICIBANK
* KOTAKBANK
* AXISBANK
* SUNPHARMA
* JSWSTEEL
* NTPC
* INDUSINDBK
* CIPLA

(These tickers appear in the notebook as `HDFCBANK.NS`, `ICICIBANK.NS`, etc.; the `.NS` suffix denotes the exchange symbol in the CSV.)

### Allocation rule

To balance exposure by risk, weights are assigned using **inverse-volatility weighting**:

1. Compute `Inverse Volatility = 1 / Volatility` for each selected ticker.
2. Normalize to sum to 1: `weight_i = inverse_vol_i / sum(inverse_vol_j)`.

This results in larger allocations to lower-volatility tickers and smaller allocations to higher-volatility ones.

**Example weights (rounded %):**

| Ticker     | Weight (%) |
| ---------- | ---------: |
| HDFCBANK   |      8.93% |
| ICICIBANK  |      6.93% |
| KOTAKBANK  |      7.66% |
| AXISBANK   |      9.22% |
| SUNPHARMA  |      7.26% |
| JSWSTEEL   |     16.00% |
| NTPC       |     28.08% |
| INDUSINDBK |      7.44% |
| CIPLA      |      8.48% |

*(Weights computed as inverse-volatility normalized; numbers are taken from the notebook computations.)*

---

## 6. Expected returns (simulation)

The notebook simulates the future value of a **monthly SIP (systematic investment)** into the constructed portfolio using the portfolio's average ROI.

* **Monthly contribution:** ₹5,000
* **Compounding frequency:** monthly (n = 12)
* **Average portfolio ROI (annual, decimal):** ≈ 0.06765 (≈ 6.76% per annum)
* **Years simulated:** 1, 3, 5, 10

Future value formula used (standard future value of series with compounding):

```text
FV = P * [ ((1 + r/n)^(n*t) - 1) / (r/n) ] * (1 + r/n)
```

**Results (approximate future values for ₹5,000 monthly):**

| Years | Future value (₹) |
| ----: | ---------------: |
|     1 |        62,244.67 |
|     3 |       200,068.59 |
|     5 |       357,800.09 |
|    10 |       859,131.70 |

**Interpretation:** At ~6.76% annual average return, a modest monthly SIP of ₹5,000 grows significantly over multi-year horizons; the compounding effect becomes visible by 5–10 years.

---

## 7. Visualizations (in the notebook)

The notebook includes (or recreates) the following plots:

* Time series of closing prices for all tickers.
* Bar plots comparing volatility and ROI across chosen groups (top growth tickers vs. selected portfolio tickers).
* Line plot showing future value of monthly investment across the different time horizons.

Refer to the plotted figures in `experiments.ipynb` for visual detail.


---

## 8. Conclusions & recommendations

* A simple **ROI > median** and **volatility < median** selection rule finds companies that historically delivered above-average returns while being less volatile.
* **Inverse-volatility weighting** helps allocate more capital to stable names and reduces concentration risk driven by noisy high-volatility tickers.
* Based on historic ROI for the selected basket, a disciplined monthly SIP can grow meaningfully over 5–10 years.

**Caveats:** past price performance is not a guarantee of future returns. The analysis uses historical price series (not fundamentals), ignores transaction costs, corporate actions, dividends, and portfolio rebalancing frequency.

---

