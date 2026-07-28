# Target Corp Quarterly Revenue Regression

Google Colab notebook (`Quiz8.ipynb`) that models Target Corp (ticker `TGT`) quarterly revenue using a time trend plus dummy variables for holiday-quarter seasonality and a COVID-era level shift.

## What it does

1. Loads quarterly sales data for a panel of companies from `qSales_2024.csv` (columns include `gvkey`, `datadate`, `fyearq`, `fqtr`, `tic`, `conm`, `saleq`, etc. — looks like Compustat-style fundamentals data, based on the column names).
2. Filters the panel down to Target Corp (`tic == 'TGT'`) and parses `datadate` into a proper datetime.
3. Plots raw quarterly revenue (`saleq`) over time (2000–2024), which shows a clear upward trend with regular seasonal spikes.
4. Builds a `time` index (1, 2, 3, ...) as a linear trend variable.
5. Constructs two dummy variables:
   - `holiday_dv` = 1 when `fqtr == 4` (Target's Nov–Jan holiday selling season), else 0.
   - `covid_dv` = 1 for dates on/after 2020-04-30 (capturing a permanent step-up in Target's revenue base post-COVID), else 0.
6. Fits an OLS regression (`statsmodels`) of `saleq` on a constant, `time`, `holiday_dv`, and `covid_dv`.
7. Generates in-sample predictions and plots actual vs. predicted revenue.

## Model results (as shown in the notebook)

| Term | Coefficient |
|---|---|
| const | 9,447.95 |
| time | 130.01 |
| holiday_dv | 4,835.37 |
| covid_dv | 4,233.24 |

R-squared: 0.946 · Adjusted R-squared: 0.944 · N = 93 observations · All coefficients significant at p < 0.001.

I am not certain about the underlying data source's exact licensing/provider — the column naming (`gvkey`, `fyearq`, `indfmt`, `popsrc`, `datafmt`) strongly resembles S&P Compustat conventions, but the notebook itself doesn't state the source, so you may want to confirm that with whoever supplied `qSales_2024.csv`.

## Requirements

- Python 3
- `pandas`, `numpy`, `matplotlib`, `statsmodels`

```bash
pip install pandas numpy matplotlib statsmodels
```

## Usage

1. Place `qSales_2024.csv` in the same directory as the notebook (or update the `pd.read_csv(...)` path).
2. Open `Quiz8.ipynb` in Jupyter or Google Colab.
3. Run all cells top to bottom.

## Notes / known issues

- The notebook throws `SettingWithCopyWarning` when creating `time`, `holiday_dv`, and `covid_dv` on `target_sales` (a filtered slice of the full DataFrame). It still runs correctly, but using `.loc[row_indexer, col_indexer] = value` or `target_sales = target_sales.copy()` after filtering would silence the warning.
- Durbin-Watson statistic is 0.395, well below 2, indicating strong positive autocorrelation in the residuals — a heads up that standard errors from this OLS specification are likely understated. You may want to account for this (e.g., HAC/Newey-West standard errors) before treating the reported p-values as reliable.
