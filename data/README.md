# Empirical Zero-Coupon Yield Curve Datasets

This directory contains the historical zero-coupon yield curve datasets used for calibrating, pricing, and testing the Cox-Ingersoll-Ross (CIR) interest rate model.

## Dataset Files

1. **`train_data.csv`**: Contains daily zero-coupon yield curve data from **2016-01-04** to **2024-03-28** (approx. 2,028 observations) used for model calibration under the physical ($\mathbb{P}$) and risk-neutral ($\mathbb{Q}$) measures.
2. **`test_data.csv`**: Contains daily zero-coupon yield curve data from **2024-04-01** to **2026-03-31** (approx. 488 observations) used for out-of-sample prediction and model comparison.
3. **`test_data_3M.csv`**: Contains only the 3-Month yield tenor for the testing period. In out-of-sample yield curve reconstruction, this file serves as the *only* input representing the short rate $r_t$.

## Data Schema & Metadata

All files contain the following structure:
- **`Date`**: Date of the trading day in `YYYY-MM-DD` format.
- **Tenor Columns**: Zero-coupon yields corresponding to different maturities (represented as decimals, e.g., `0.05` represents a 5% yield):
  - **`ZC025YR`**: 3-Month (0.25 Years) - serves as the short-rate proxy $r_t$
  - **`ZC050YR`**: 6-Month (0.5 Years)
  - **`ZC075YR`**: 9-Month (0.75 Years)
  - **`ZC100YR`**: 1-Year (1.0 Years)
  - **`ZC200YR`**: 2-Year (2.0 Years)
  - **`ZC500YR`**: 5-Year (5.0 Years)
  - **`ZC1000YR`**: 10-Year (10.0 Years)
  - **`ZC2000YR`**: 20-Year (20.0 Years)
  - **`ZC3000YR`**: 30-Year (30.0 Years)

## Data Acquisition and Preprocessing

The raw yield curve data is obtained from central bank zero-coupon bond databases (e.g., Federal Reserve Board, ECB, or Reserve Bank of India zero-coupon yield curves).

### Preprocessing Checklist
If you are updating or replacing the data files, ensure:
1. **Clean Column Formatting**: Strip leading and trailing spaces from column names (e.g., `" Date "` should be `"Date"`).
2. **Chronological Sorting**: Sort observations chronologically by the `Date` column.
3. **Yield Format**: Verify that interest rates are expressed as decimals (not percentages, e.g., use `0.045` for `4.5%`).
4. **Outlier Detection**: Ensure there are no negative values or extreme data entry anomalies (spikes/dropouts), which can cause MLE optimization to fail or violate the Feller condition.
