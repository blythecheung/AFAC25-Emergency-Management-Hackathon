# AFAC25-Emergency-Management-Hackathon

## Problem Statement

Using DFES overtime, employee and termination data from 2022–2025, this project predicts monthly staffing-driven overtime hours at the station level, and flags stations at highest risk of an overtime spike. Success is measured against a naive baseline (last month's actual value).

This addresses a challenge described directly in the DFES Workforce Services brief: 
the reactive nature of rostering significantly increases overtime costs, and there is a clear need for predictive analytics to better anticipate staffing gaps.

## Data

- **Overtime Data** - individual overtime shift records, 2022–2025, including employee ids, overtime hours worked (units), classification description, fire station names (cost centre), and reason for overtime.
- **Current CFRS Employee List** — active employee roster with station, region, work pattern
- **Terminations** — historical departures, used to explain employees present in overtime history but absent from the current employee list

Raw files are not included in this repository due to data governance restrictions on DFES workforce data. See `.gitignore`. The head of each of dataset can be seen in the Jupyter notebook.

## Approach

- **Grain:** station-month (employee-day was considered, but not feasible without 
  roster/establishment data, which wasn't available in this dataset)
- **Target:** staffing-driven overtime hours — overtime specifically coded as 
  "Cover Insufficient Staff" or "Cover Sick Leave," isolated from incident/training overtime
- **Features:** 1-month lag, 3-month rolling average, month-of-year, station identity 
  (one-hot encoded)
- **Model:** Linear Regression, evaluated against a naive persistence baseline 
  (predict = last month's value)
- **Risk flag:** a station-month is flagged if its predicted value exceeds that 
  station's own historical 80th percentile — thresholds are station-specific since 
  overtime volume varies enormously by station size

## Key Findings

- **Model MAE: 101.18 vs. naive baseline MAE: 162.36** — a ~38% reduction in error
- Data quality: 1,890 overtime rows (73 distinct employees) had no match in the current employee list; 90% of those employees were confirmed as departed via the Terminations table
- Perth Fire Station has the highest raw overtime of any station, but its station-level model coefficient is near zero — its overtime is well explained by its own recent trend, not an unexplained "Perth effect"
- Lag and rolling-average features are correlated (r≈0.91); their individual coefficients should not be read as independent effects, though overall model accuracy is unaffected

## Dashboard

(https://public.tableau.com/views/OvertimeRiskForecast/StationOvertimeRiskForecast?:language=en-US&publish=yes&:sid=&:redirect=auth&:display_count=n&:origin=viz_share_link)

Interactive dashboard showing actual vs. predicted vs. baseline overtime trends by station, and a station-level risk summary sorted by predicted overtime.

## Limitations & Future Work

- Scoped to station-month grain due to no available roster/establishment table; employee-day prediction would better match DFES's stated operational framing but requires shift roster data not provided in this dataset
- Month-of-year is encoded linearly (1–12), which doesn't capture cyclical seasonality (December/January adjacency); cyclical encoding (sine/cosine) is a natural next iteration
- Risk flagging uses a fixed historical percentile threshold rather than a probabilistic/confidence-interval approach
- Incident Mobilisation and Health & Safety data were not incorporated due to unresolved employee-ID linkage (masked IDs in one table, no employee ID at all in the other) — a potential future data source once a crosswalk is available
- Leave Booking data was not incorporated but is a natural next addition, since it would allow predicting sick-leave-driven overtime ahead of time rather than inferring it from overtime records after the fact

## Tools

Python (pandas, scikit-learn, matplotlib), Tableau Public

## Author

Blythe Cheung — BSc Data Science & Finance, University of Western Australia (2025)