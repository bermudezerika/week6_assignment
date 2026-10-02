# Plan: SF Rent Growth vs. the Real 10-Year Interest Rate

**Question:** Does San Francisco rent growth rise when the real (inflation-adjusted) 10-year interest rate is higher?

## 1. Expectation
**Positive relationship, with a lag of 6–18 months.** When real rates rise, mortgages get more expensive and buying a home costs more. More households keep renting, so rental demand and rent growth increase.
- **Competing view (negative):** high real rates tighten financial conditions, which slows tech hiring and lowers SF rental demand.

## 2. Data: FRED API (2 series)
| Role | Series ID | Description | Frequency / range |
|---|---|---|---|
| X | `FII10` | 10-Year TIPS (real) yield, % | Monthly, Jan 2003 to present |
| Y | `CUURA422SEHA` | CPI-U Rent of Primary Residence, SF-Oakland-Hayward, index | Monthly since 2002, NSA |

- **Endpoint:** `https://fred.stlouisfed.org/graph/fredgraph.csv?id=<SERIES>`, read with `pandas.read_csv` (no key needed).
- **Optional:** the `fredapi` package with a `FRED_API_KEY` env var.
- **Packages:** `pandas`, `numpy`, `matplotlib`, `statsmodels`, `scipy`.

## 3. Cleaning
1. **Parse and validate:** parse `observation_date` as the date index and apply `pd.to_numeric(errors="coerce")`.
2. **Missing values:** drop blank cells with no interpolation. Since 2002 the only blank rent month is **Oct 2025** (likely the federal shutdown), which removes the Oct 2025 and Oct 2026 YoY values. The notebook prints any missing months.
3. **Date range:** keep rent from Jan 2002. `FII10` starts in Jan 2003, so the 12-month change and lag first exist in **Jan 2004**. The final sample runs from 2004-01 to the latest month both series share.
4. **Transform:**
   - Rent YoY % = value / same month one year earlier − 1.
   - 12-month change in the real rate (pp).
   - Real rate lagged **12 months**, fixed in advance (the middle of the expected 6–18 month window) and not chosen from the data.
5. **Merge:** inner-join on month, build all derived columns in this step, and drop NaNs. Add a COVID dummy for Mar 2020 to Dec 2021. **All tests use this one common sample.**

## 4. Charts
- Two stacked line panels (real rate; rent YoY) sharing a time axis, with COVID shaded. No dual axis.
- A scatter plot of rent YoY vs. the real rate, with an OLS fit line.
- A bar chart of the correlation between rent YoY and the real rate lagged 0–24 months, with the 12-month lag highlighted. This chart is **descriptive only** and isn't used to pick the lag.

## 5. Tests
| Test | Purpose |
|---|---|
| OLS: rent YoY ~ real rate (+ COVID), Newey-West SEs (12 lags) | Level relationship |
| OLS: rent YoY ~ 12-month Δ real rate (+ COVID), Newey-West SEs | Do *rising* rates go with faster rent growth? |
| OLS: rent YoY ~ real rate lagged 12 months (+ COVID), Newey-West SEs | Delayed effect (pre-specified lag) |
| Welch t-test: rent growth when the real rate is above vs. below its median | Compare high- vs. low-rate periods |

Every test reports its N. The conclusion text is generated from the computed results, so it stays correct when the data updates.

## 6. Interpreting results
- **Supports the expectation:** positive coefficients with HAC p < 0.05 (including the 12-month lagged model), and faster mean rent growth in high-rate months. These should still hold with the COVID dummy included.
- **Contradicts the expectation:** a significantly negative coefficient, which would favor the tighter-financial-conditions view, as would slower rent growth in high-rate months.
- **No result:** coefficients near zero and insignificant. Caveats to note: about 22 years of data, few independent cycles, correlation rather than causation, and rent CPI lagging market rents.
