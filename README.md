# KPI Anomaly Detection System (KPI Sentinel)

A production-style retail KPI monitoring system for anomaly detection and hierarchical root cause analysis, built on the **M5 Forecasting (Walmart daily sales)** dataset.

---

## Demo

**Dashboard screenshots** <img width="1838" height="811" alt="Dashboard - Overview" src="https://github.com/user-attachments/assets/fc98fd3b-3f46-474e-a459-7c8c8371b5b2" /> <img width="1919" height="890" alt="Dashboard - Root Cause Explorer" src="https://github.com/user-attachments/assets/937d21c5-eda1-4b22-bc19-415e21f5ac4d" />

**Short walkthrough video (click to play)** <a href="https://github.com/user-attachments/assets/39b9592b-3ac3-41f1-a793-6aace36f80fb"> <img src="https://github.com/user-attachments/assets/07bb545b-9d3a-4e5c-b64b-28725fc31485" width="900" alt="KPI Sentinel Demo Video" /> </a>

---

## What this project does

KPI Sentinel is an end-to-end decision-support analytics product that:

1. Computes retail KPIs (units, revenue, avg_price, price_index, zero_sales_rate, demand_volatility) at **daily** granularity
2. Detects anomalies using seasonality-aware baselines (**STL decomposition with weekly seasonality**)
3. Attributes root cause using **hierarchical contribution analysis** (global -> state -> store -> department)
4. Provides a Streamlit dashboard and auto-generated incident reports

This is not just a model. It is a complete monitoring + investigation workflow designed for retail operations and analytics teams.

---

## Dataset

This system uses the **M5 Forecasting – Accuracy** competition dataset.

* Dataset link: [https://www.kaggle.com/competitions/m5-forecasting-accuracy/data](https://www.kaggle.com/competitions/m5-forecasting-accuracy/data)
* Files required (place in `data/raw/`):

  * `sales_train_evaluation.csv`
  * `calendar.csv`
  * `sell_prices.csv`

| File                         | Description                                                                |
| ---------------------------- | -------------------------------------------------------------------------- |
| `sales_train_evaluation.csv` | Daily unit sales for ~30k product series across 10 stores over ~1,900 days |
| `calendar.csv`               | Date mapping + events (holidays, sports, etc.) + SNAP benefit days         |
| `sell_prices.csv`            | Weekly selling prices by store and item                                    |

### Data hierarchy (used for RCA)

```
Global
  └─ State (CA, TX, WI)
      └─ Store (CA_1..CA_4, TX_1..TX_3, WI_1..WI_3)
          └─ Department (FOODS_1..3, HOBBIES_1..2, HOUSEHOLD_1..2)
              └─ Item (~30,000)
```

---

## KPIs computed

All KPIs are computed **daily** at multiple hierarchy levels (global, state, store, category, department).

| KPI                 | Definition                                 | Business meaning                 |
| ------------------- | ------------------------------------------ | -------------------------------- |
| `units`             | sum(daily units sold)                      | Sales volume / demand            |
| `revenue`           | sum(units * sell_price)                    | Sales value                      |
| `avg_price`         | revenue / units                            | Revenue-weighted average price   |
| `price_index`       | avg_price / rolling_median(avg_price, 28d) | Price relative to recent history |
| `zero_sales_rate`   | count(series with 0 sales) / total series  | Stockout / demand collapse proxy |
| `demand_volatility` | rolling MAD of units (28d)                 | Demand stability / risk          |

---

## Methodology

### Baseline (seasonality-aware)

Uses **STL (Seasonal-Trend decomposition using Loess)**:

* Period = 7 (weekly seasonality)
* Robust fitting to reduce outlier impact
  Baseline = Trend + Seasonal

### Anomaly scoring (robust)

Residual-based scoring using **Median Absolute Deviation (MAD)**:

```
sigma(t) = rolling_MAD(residual, 28 days)
score(t) = residual(t) / (sigma(t) + eps)
anomaly_flag = |score| >= 3.5
```

### Severity metrics

* `z_severity = min(100, 20 * |score|)`
* `pct_change = (y - baseline) / (|baseline| + eps)`
* `pct_severity = min(100, 100 * |pct_change|)`

### False-positive controls

1. **Minimum history**: no scoring until enough history is available
2. **Cooldown**: after a flagged anomaly, suppress the next 2 days unless severity increases > 25%
3. **Event grouping**: consecutive anomaly days are grouped into a single incident event (start/end/peak)

### Root cause analysis (hierarchical RCA)

For each incident at the **peak date**:

1. Identify child segments one level down the hierarchy
2. Compare a **28-day “before” window** vs the **peak day**
3. Compute contribution changes and value deltas
4. Rank drivers by:

```
driver_score = 0.7 * |delta_value| + 0.3 * |delta_share| * |total_after|
```

Calendar context (events/SNAP/weekday) is attached to the incident for better interpretability.

---

## Evaluation

M5 has no anomaly labels, so evaluation uses **synthetic anomaly injection**:

1. Select representative series across metrics and hierarchy levels
2. Inject known anomalies (e.g., 30% revenue drop, 20% price spike) and record ground truth
3. Run detection and compute:

   * event-level Precision / Recall / F1
   * Mean Time to Detect (MTTD)
   * False positives per 30 days

See `notebooks/02_detection_eval.ipynb` for details.

---

## Repository structure

```
kpi-anomaly-detection-system/
  README.md
  requirements.txt
  .gitignore

  data/
    raw/                     # M5 CSV files (NOT committed)
    processed/               # Generated artifacts (NOT committed)

  src/
    config.py
    utils.py
    m5_ingest.py
    kpi_build.py
    baseline.py
    detect.py
    root_cause.py
    report.py

  app/
    streamlit_app.py

  notebooks/
    01_data_prep_eda.ipynb
    02_detection_eval.ipynb
    03_rca_examples.ipynb

  outputs/
    reports/                 # Generated incident reports (NOT committed)
    figures/                 # Generated plots (NOT committed)

  tests/
    test_detect.py
    test_root_cause.py
```

---

## Quickstart

### Prerequisites

* Python 3.11+
* Kaggle M5 CSVs downloaded into `data/raw/`

### Install

```bash
cd kpi-anomaly-detection-system
pip install -r requirements.txt
```

### Run the pipeline

```bash
python -m src.m5_ingest
python -m src.kpi_build
python -m src.detect
python -m src.root_cause
```

### Launch the dashboard

```bash
streamlit run app/streamlit_app.py
```

Dashboard pages:

1. Overview (KPI trends + anomalies)
2. Root Cause Explorer (hierarchical drilldown)
3. Incident Report Generator (Markdown reports)

### Run tests

```bash
pytest -v
```

---

## Configuration

Key parameters in `src/config.py`:

| Parameter            | Default | Description                        |
| -------------------- | ------- | ---------------------------------- |
| `STL_PERIOD`         | 7       | Weekly seasonality period          |
| `MAD_WINDOW`         | 28      | Rolling window for MAD calculation |
| `ANOMALY_Z`          | 3.5     | Threshold for anomaly flagging     |
| `COOLDOWN_DAYS`      | 2       | Days to suppress after anomaly     |
| `BEFORE_WINDOW_DAYS` | 28      | RCA comparison window              |
| `MIN_HISTORY_DAYS`   | 35      | Minimum history before scoring     |

---

## Incident reports

Generated reports (Markdown) include:

1. Summary (metric/segment/direction/magnitude)
2. KPI trend (observed vs expected)
3. Severity assessment (z + pct severity)
4. Top drivers (hierarchical RCA)
5. Calendar context (events/SNAP/weekday)
6. Recommended checks (rule-based)

Example checks:

* Revenue drop + stable avg_price -> investigate demand/supply disruption
* avg_price spike -> inspect sell_price changes by store/item
* zero_sales_rate spike -> check for stockouts in top affected segments

---


## License

MIT License

---

## Acknowledgments

* M5 Forecasting Competition organizers and Walmart for the dataset
* statsmodels for STL decomposition
* Streamlit for the dashboard framework
