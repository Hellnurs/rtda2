# AITU Smart Classroom Energy Stream

**Assignment 2 — Real-Time Data Processing with Pandas and NumPy**
Astana IT University · Real-time Data Analysis · 2026-2027

| | |
| --- | --- |
| **Author** | Nurkeldi Samuratov, Mendulda Alisher, Zharkyn Mukhambetiyar (group BDA-2404) |
| **Version** | `commit-v1.2-completed-assignment2` |
| **Submitted** | September 21, 2026 |

## Overview

The project processes one-minute streaming telemetry from classroom C1.3.101 (12 timestamps, 10:00–10:11) with Pandas and NumPy. It cleans missing sensor records, computes 3-minute rolling statistics, engineers three domain features, measures the relationship between occupancy and power draw, and produces two figures with facility-management recommendations.

Input columns: `Temp_C`, `CO2_ppm`, `Occupancy`, `Power_kW`.

## Repository contents

| File | Purpose |
| --- | --- |
| `RTDA_2assignmentipynb.ipynb` | Main notebook: cleaning, rolling statistics, features, statistics, plots |
| `Assignment_2_Tests.py` | Tests and experiment checks |
| `README.md` | This file |

## Requirements

- Python
- Pandas 
- NumPy 
- Matplotlib 
- Seaborn 

## How to run

### Google Colab

1. Upload `RTDA_2assignmentipynb.ipynb` (and `Assignment_2_Tests.py`) to Colab.
2. Select **Runtime → Run all**.

### Local Jupyter

```bash
git clone <repository-url>
cd <repository-folder>
python -m venv .venv
source .venv/bin/activate
pip install "pandas>=2.0" "numpy>=1.26" "matplotlib==3.10.0" "seaborn>=0.13" jupyter
jupyter notebook RTDA_2assignmentipynb.ipynb
```

On Windows, activate the environment with `.venv\Scripts\activate`.

Run all cells from top to bottom; the notebook is designed to execute in order, and later cells depend on the cleaned DataFrame from cells 2–5.

### Tests

```bash
python Assignment_2_Tests.py
```

## Pipeline

| Step | What it does | Method |
| --- | --- | --- |
| 1. Cleaning | Fills `Temp_C` (10:02), `CO2_ppm` (10:04), `Occupancy` (10:07) | Linear interpolation for the two continuous variables, forward fill for the integer headcount |
| 2. Rolling statistics | 3-minute trailing mean of temperature, CO2 and power | `rolling(window=3).mean()`; first two rows are `NaN` |
| 3. Feature engineering | `occupancy_density`, `energy_per_student`, `comfort_flag` | Occupancy / 40; Power_kW / Occupancy; CO2 ≤ 1000 ppm and 20 ≤ Temp ≤ 24 °C |
| 4. Statistics | Descriptive statistics and Pearson correlation | Occupancy vs. power |
| 5. Visualization | Dual-axis rolling CO2/temperature plot; occupancy vs. power scatter with OLS trendline | Matplotlib, Seaborn |

## Key results

| Item | Result |
| --- | --- |
| Imputed values | 21.9 °C, 750.0 ppm, 29.0 students; 0 nulls remain |
| Temperature (mean / median / min / max) | 22.48 / 22.55 / 21.50 / 23.30 °C |
| CO2 (mean / max) | 789.17 / 950.00 ppm |
| Peak power | 5.7 kW at 10:11 |
| Pearson r (occupancy vs. power) | 0.9946 |
| Energy per student | Falls from 0.178 kW at 10:00 to a plateau of ≈ 0.154 kW |

Operational recommendations for AITU facility management:

- Raise fresh-air intake when CO2 crosses 850 ppm or occupancy exceeds 25–28 students, before the 1000 ppm comfort ceiling is reached.
- Use occupancy sensors integrated with the Building Management System to cut AV and non-essential lighting load when `occupancy_density` < 0.20; the extrapolated non-occupant baseload is roughly 1.5–1.8 kW.

## Design decisions

- **Interpolation vs. mean/median imputation:** temperature and CO2 change continuously, so linear interpolation preserves the signal; global mean or median would distort the rolling windows.
- **Forward fill for occupancy:** headcount is a discrete integer, so carrying forward the last verified value avoids fractional occupants.
- **Testing:** the `comfort_flag` expression was boundary-tested with CO2 = 1001 ppm and Temp = 24.1 °C.
- **Plotting bug:** the leading `NaN` rows from `rolling(window=3)` are removed with `.dropna()` on the plotting subset only, so the main index stays intact.

## Limitations and future work

- Rolling windows use a fixed row count, which assumes uniform 1-minute intervals. A time-aware window (`.rolling('3min')` on a `DatetimeIndex`) would remove that assumption.
- Two-sided linear interpolation needs the next observation, so it is not causal. A production stream would need a small buffer or a one-sided estimator such as an adaptive Kalman filter.
- The dataset is a 12-minute sample from a single room, so the regression baseload and thresholds are indicative rather than validated.

