# Empirical Results: Out-of-Sample Performance and Model Evaluation

This document presents the out-of-sample performance metrics, comparative analysis, interest rate regime evaluations, and Monte Carlo simulation analysis for the calibrated interest rate models.

---

## 1. Out-of-Sample Performance Comparison

The models were evaluated on the held-out test dataset spanning **2024-04-01** to **2026-03-31** (488 trading days). For each trading day, all models were restricted to ingesting **only the 3-Month yield (ZC025YR)** as the proxy for the instantaneous short rate $r_t$. The remaining tenors (6-Month, 9-Month, 1-Year, and 2-Year) were reconstructed and compared against the empirical zero-coupon yields.

### 1.1 Summary Performance Metrics

| Model Configuration | Overall Out-of-Sample $R^2$ | Average MAE (bps) | Average RMSE (bps) | Feller Condition Satisfied? |
| :--- | :---: | :---: | :---: | :---: |
| **CIR Q-Calibrated (Direct)** | **0.8851** | **14.81** | **22.70** | Yes |
| **CIR++ (Shift-Extended)** | **0.8266** | **21.40** | **27.89** | Yes |
| **Two-Factor Kalman Filter** | **0.8054** | **20.10** | **29.54** | Yes (both factors) |
| **CIR MLE (P-Calibrated)** | **0.8093** | **19.27** | **29.25** | No (violated) |
| **CIR OLS Baseline (Naive)** | **-0.6117** | **64.28** | **85.03** | No (violated/negative $\kappa$) |

*Note: Basis points (bps) are defined as $1\text{ bp} = 0.0001$ ($0.01\%$).*

---

## 2. Key Quantitative Findings & Analysis

### 2.1 The Superiority of Direct $\mathbb{Q}$-Calibration
The direct cross-sectionally calibrated model under the risk-neutral measure $\mathbb{Q}$ achieves the highest performance, with an overall out-of-sample $R^2$ of **0.8851**. This outperforms both the physical measure calibration ($\mathbb{P}$-MLE adjusted by $\lambda$) and the model extensions.
* **Why physical calibration fails for pricing**: $\mathbb{P}$-measure calibrations describe the historical timeseries dynamics of the short rate. Over our training period (2016-2024), interest rates experienced structural shifts and strong trends, making parameters like speed of mean reversion ($\kappa$) unstable or negative under OLS, and forcing MLE to hit its boundary limits. 
* **Why $\mathbb{Q}$-calibration excels**: Direct calibration to the cross-section of the entire yield curve over the training set stabilizes the parameters ($\kappa_q = 0.153263$, $\theta_q = 0.026018$, $\sigma_q = 0.064228$) and satisfies the Feller condition ($2\kappa\theta = 0.007975 \ge \sigma^2 = 0.004125$). It directly optimizes what the pricing formula expects, ensuring high out-of-sample predictive accuracy.

### 2.2 Maturity-Wise Performance Decay (Single-Factor Limitation)
While the overall $R^2$ is high, the model's accuracy degrades significantly as maturity increases:
* **6-Month Tenor (`ZC050YR`)**: $R^2 = 0.9938$ (highly accurate)
* **1-Year Tenor (`ZC100YR`)**: $R^2 = 0.9030$ (strong fit)
* **2-Year Tenor (`ZC200YR`)**: $R^2 = 0.3524$ (poor fit)

This performance decay is a direct consequence of the **single-factor assumption**. A single-factor model assumes that the instantaneous short rate $r_t$ is the *only* source of uncertainty in the term structure. Consequently, yields for all maturities are perfectly correlated with $r_t$. In reality, longer-term yields are driven by independent macroeconomic factors (such as long-term inflation expectations and term premiums) that do not move in tandem with short-term rate fluctuations.

### 2.3 The Flexibility-Stability Tradeoff (Model Extensions)
A surprising quantitative result is that the more flexible models—**CIR++ (shift-extended)** and the **Two-Factor Kalman Filter**—underperformed the basic Q-calibrated model out-of-sample ($R^2$ of 0.8266 and 0.8054, respectively, compared to 0.8851).
1. **CIR++ Overfitting**: The CIR++ model adds deterministic shift factors $\Phi(\tau)$ estimated from the training set average residuals to improve the curve fit. However, because the macroeconomic regime shifted from a low-interest environment in the training set (2016-2024) to a high-interest environment in the test set (2024-2026), these static shifts introduced a persistent pricing bias out-of-sample.
2. **Two-Factor Kalman Filter Noise**: The two-factor model doubles the parameters and relies on a Kalman Filter to recursively track latent factors $(x_t, y_t)$ using only the 3-Month yield on the test set. The estimation of these latent states is highly sensitive to noise, causing estimation lag and filter noise, which degrades predictive performance compared to the direct single-factor mapping.

---

## 3. Interest Rate Regime Analysis

To test model robustness across different market environments, we segment the out-of-sample test set into two interest rate regimes based on the short-rate proxy (3M yield):
1. **Regime 1: Low Interest Rate / Decreasing Rate Regime** (short rate $r_t < 3.5\%$)
2. **Regime 2: High Interest Rate / Stable Rate Regime** (short rate $r_t \ge 3.5\%$)

### 3.1 Regime-wise Mean Absolute Error (MAE)
The direct Q-calibrated model shows varying pricing errors across the regimes:
* **Regime 1 (Low Rate)**: Reconstructed yields align closely with actual yields, with errors concentrated at the short end.
* **Regime 2 (High Rate)**: Errors increase, particularly for the 2-Year tenor, as the shape of the yield curve twists during monetary tightening cycles, which a single-factor model cannot capture.

---

## 4. Monte Carlo Simulations & Stress Testing

### 4.1 Physical Measure Simulations
Using parameters calibrated under the physical measure $\mathbb{P}$ via MLE, we run 1,000 Monte Carlo paths over a 1-year horizon (252 trading days) starting from the last observed test rate ($r_0$).
* **Mean Path**: Reverts back toward the long-term physical mean $\theta$, demonstrating the mean-reverting drift of the SDE.
* **Volatility Bands**: The 5th and 95th percentiles expand over time, demonstrating the range of potential rate paths and reflecting the square-root dependency of volatility on the interest rate level.

### 4.2 Stress Scenario Analysis
We simulate interest rate shock scenarios (e.g., sudden central bank rate hikes) to evaluate how the yield curve shape and pricing bounds evolve under stressed conditions, which is crucial for asset-liability management (ALM) and value-at-risk (VaR) calculations.
