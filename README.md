# Upside-vs.-Realized-Returns
Time-series study of Apple (AAPL) testing whether analyst target-price implied upside predicts 12-month returns (2005-2024). Predictive regression with controls and Newey-West HAC finds headline upside is weak/noisy; firm size is the most robust predictor.


Here’s a deeper README you can paste into `README.md` and then tweak the file names/paths once you upload your `.do` file and dataset.

## Overview

Investors often see headlines like “analysts expect X% upside” based on consensus target prices. This project tests whether that **target-price–implied upside** actually predicts **future realized returns** for Apple (AAPL), or whether it mainly reflects optimism that gets corrected over time. 

Using a monthly AAPL time series (231 observations, **2005m1–2024m3**), I estimate a predictive regression where the outcome is Apple’s **12-month forward return** and the key regressor is the **consensus implied upside** from analyst target prices. 

---

## Research Question

**Does higher analyst “upside” (consensus target price relative to current price) predict higher realized 12-month returns for AAPL?** 

A motivating possibility (supported in prior literature) is that target prices can be informative, but also biased/optimistic—meaning the implied-upside slope does not have to be positive and could even be negative in some settings. 

---

## Data

### Sample

* **Monthly data:** 2005m1–2024m3
* **N = 231 observations** 

During sample construction, monthly dates were generated in Excel using an iterative approach that created a small number of duplicate dates; **4 duplicate records were removed** to preserve one observation per month. 

### Sources

* **Bloomberg (Excel API)** fields:

  * `BEST_ANALYST_RECS_BULK` (consensus target-price input)
  * `CUR_MKT_CAP` (market cap)
  * `PE_Ratio` (P/E)
  * `RK004` (underlying volatility series used for VIX proxy) 
* **Excel `STOCKHISTORY()`** for AAPL historical prices (used to construct realized returns). 

Consensus target price is constructed monthly in Excel as an **AVERAGE() across available analyst entries** from the Bloomberg field. 

---

## Variable Construction

### Dependent variable: realized forward return

* `ret_act_t`: **12-month forward price return**, computed monthly as the percent change from month *t* to month *t+12*. 

### Main regressor: analyst implied upside

* `ret_est_t`: **target-price–implied return (headline upside)**
  [
  ret_est_t = \frac{\text{Avg Target Price}_t - \text{Current Price}_t}{\text{Current Price}_t}
  ]


### Controls

* `ret_past_t`: AAPL’s **12-month lagged return** 
* `log_mktcap_t`: ln(Market Cap) from Bloomberg `CUR_MKT_CAP` 
* `vix_t`: monthly volatility series derived from Bloomberg `RK004` 
* `log_pe_t`: ln(P/E) from Bloomberg `PE_Ratio` 

Notes:

* No winsorization/trimming is applied; logs are used for market cap and P/E to reduce skewness. 
* Return variables are treated as stationary based on ADF tests; `log_mktcap` is trending/borderline non-stationary and is kept as a slowly trending control. 

---

## Methodology

### Predictive regression

I estimate the monthly time-series model:

[
ret_act_t = \beta_0 + \beta_1 ret_est_t + \beta_2 ret_past_t + \beta_3 log_mktcap_t + \beta_4 vix_t + \beta_5 log_pe_t + u_t
]


**Timing:** all regressors are measured using information available at time *t*, and the dependent variable is a forward return over *t → t+12*. 

### Why Newey–West (HAC) standard errors

Because the dependent variable is a **12-month forward return measured monthly**, observations overlap across adjacent months, which induces serial correlation in errors. Inference is therefore based on **Newey–West HAC standard errors with a 12-month lag window (NW(12))** as the preferred specification. 

---

## Results Summary

### Key finding: “headline upside” is weak/noisy for AAPL

Under the preferred NW(12) inference:

* `ret_est` is **negative** in point estimates (**β̂ ≈ −0.501**), suggesting higher stated upside is followed (if anything) by slightly lower future returns, but it is **not statistically strong** (**p ≈ 0.145**). 
* `log_mktcap` is the **only clearly robust predictor**: coefficient **β̂ ≈ −0.148**, statistically strong (**p ≈ 0.001**). 
* Other controls (past return, VIX, valuation) do not deliver stable predictive power after HAC adjustment. 

Model fit (baseline OLS): **R² ≈ 0.2565** (typical for single-firm return prediction). 

---

## Diagnostics (Why HAC matters here)

Standard OLS assumptions are violated in this setting (as expected in overlapping-horizon predictive regressions):

* Serial correlation and heteroskedasticity motivate HAC inference. 
* Functional-form concerns show up via RESET; multicollinearity appears mild (mean VIF ≈ 1.88). 

---

## How to Run

1. Install Stata (version used in your environment).
2. Place the raw in data in "Data" folder into a .dta environment.
3. Run the do-file:

   * from Stata GUI: open `code/analysis.do` and execute
   * or from command line (example):

     * `do code/analysis.do`

Your do-file should:

* import the processed dataset
* declare the time series (monthly)
* construct forward return and control variables if not pre-built
* run OLS and Newey–West (NW(12)) regressions
* export tables/figures into `output/`

---

## Limitations / Interpretation

This is a **single-firm time-series** design, so:

* external validity is limited (results may differ across firms/sectors/subperiods)
* causal interpretation is not appropriate (analyst targets respond to information that may also affect expected returns)
* the consensus implied-upside measure may contain measurement error (stale targets, coverage changes, averaging). 

Practical implication: reported “X% upside” for AAPL should be treated as a **noisy sentiment/valuation indicator**, not a mechanical forecast of next year’s return. 

If you paste your `.do` file (or even just the top half with your imports + variable creation), I’ll tighten this README to match your **exact** filenames, commands, and outputs so it’s perfectly reproducible.
