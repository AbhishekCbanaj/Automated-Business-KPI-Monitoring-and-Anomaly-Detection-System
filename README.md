# Automated Business KPI Monitoring and Anomaly Detection System

A production-style business KPI monitoring system for anomaly detection and hierarchical root cause analysis, built on the M5 Forecasting (Walmart daily sales) dataset.

---

## Overview

This system simulates how real businesses monitor operational performance and detect issues proactively.

It provides:

1. Automated KPI computation across multiple business dimensions  
2. Statistical anomaly detection with seasonality awareness  
3. Hierarchical root cause analysis for issue diagnosis  
4. Interactive dashboard and incident reporting for decision-making  

This is not just a model. It is a complete monitoring and decision-support system for business analytics workflows.

---

## Demo

Dashboard screenshots:  
<img width="1838" height="811" src="https://github.com/user-attachments/assets/fc98fd3b-3f46-474e-a459-7c8c8371b5b2" />  
<img width="1919" height="890" src="https://github.com/user-attachments/assets/937d21c5-eda1-4b22-bc19-415e21f5ac4d" />

Walkthrough video:  
https://github.com/user-attachments/assets/39b9592b-3ac3-41f1-a793-6aace36f80fb

---

## What This System Does

The system performs an end-to-end monitoring workflow:

1. Computes business KPIs such as revenue, demand, pricing, and volatility at daily granularity  
2. Detects anomalies using seasonality-aware baselines based on STL decomposition  
3. Identifies root causes using hierarchical contribution analysis across business segments  
4. Generates automated incident reports and enables interactive investigation through dashboards  

---

## Dataset

M5 Forecasting – Accuracy dataset:  
https://www.kaggle.com/competitions/m5-forecasting-accuracy/data

Required files in data/raw/:

1. sales_train_evaluation.csv  
2. calendar.csv  
3. sell_prices.csv  

---

## Data Hierarchy

Global  
State (CA, TX, WI)  
Store  
Department  
Item  

This enables multi-level monitoring and root cause analysis.

---

## Key KPIs

1. units: total units sold  
2. revenue: units multiplied by price  
3. avg_price: revenue divided by units  
4. price_index: price relative to historical median  
5. zero_sales_rate: proportion of zero-sale items  
6. demand_volatility: rolling variability of demand  

---

## Methodology

### Baseline Modeling
Uses STL decomposition with weekly seasonality.

Baseline equals trend plus seasonal components.

---

### Anomaly Detection
Residual-based scoring using Median Absolute Deviation.

Anomalies are flagged when deviation exceeds threshold.

---

### Severity Measurement

1. Statistical severity based on deviation  
2. Percentage deviation from baseline  

---

### False Positive Control

1. Minimum history requirement  
2. Cooldown after anomaly detection  
3. Grouping of consecutive anomaly days  

---

### Root Cause Analysis

1. Compare pre-event and event data  
2. Compute contribution changes  
3. Rank drivers based on impact  

---

## Evaluation

Synthetic anomaly injection is used:

1. Inject controlled anomalies  
2. Measure precision, recall, and F1 score  
3. Track mean time to detect  

---

## System Architecture

Data Ingestion → KPI Computation → Baseline Modeling → Anomaly Detection → Root Cause Analysis → Reporting → Dashboard

---

## Repository Structure

kpi-monitoring-system/

README.md  
requirements.txt  

data/  
raw/  
processed/  

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
data_prep.ipynb  
detection_eval.ipynb  
rca_analysis.ipynb  

outputs/  
reports/  
figures/  

tests/  
test_detect.py  
test_root_cause.py  

---

## Quickstart

Install dependencies:

pip install -r requirements.txt

Run pipeline:

python -m src.m5_ingest  
python -m src.kpi_build  
python -m src.detect  
python -m src.root_cause  

Launch dashboard:

streamlit run app/streamlit_app.py  

---

## Key Features

1. Automated KPI monitoring across business dimensions  
2. Seasonality-aware anomaly detection  
3. Root cause identification  
4. Interactive dashboard  
5. Incident reporting  

---

## Business Use Cases

1. Revenue monitoring  
2. Supply chain issue detection  
3. Pricing anomaly tracking  
4. Demand volatility analysis  
5. Operational performance monitoring  

---

## Business Impact

1. Enables early detection of performance issues  
2. Reduces manual monitoring effort  
3. Improves decision-making through root cause insights  
4. Scales monitoring across business segments  
5. Converts raw data into actionable intelligence  

---

## License

MIT License  

---

## Acknowledgments

1. M5 Forecasting Competition and Walmart  
2. statsmodels for STL decomposition  
3. Streamlit for dashboard framework  
