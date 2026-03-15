# India Youth Employment Intelligence Platform
### Built on Databricks Lakehouse | Delta Lake · MLflow · Structured Streaming

> A production-grade data platform that analyzes India's youth unemployment crisis —
> identifying which states need intervention most, which skills are missing,
> and streaming live job market signals in real time.

---

## 📌 The Problem

| Metric | Value |
|---|---|
| India youth unemployment rate | 45.4% (2022–23) |
| Graduates considered unemployable | 1 in 2 |
| Projected cost of skill gap by 2030 | ₹35 Lakh Crore |
| Youth share of total unemployed | 83% |

> *"India's economy grew 5–8% over three decades but job growth stayed at 1%.
> The country is experiencing jobless growth — and youth are paying the price."*

This is not just a portfolio project. This is a problem millions of Indian youth
face every day. I built this because I am one of these numbers.

---

## 🏗️ Architecture

```
Raw Data (CSV)
     │
     ▼
┌─────────────────────────────────────┐
│           BRONZE LAYER              │
│  unemployment_bronze (267 rows)     │
│  jobs_bronze (9,355 rows)           │
│  Delta format · OPTIMIZE applied    │
│  Time Travel enabled                │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│           SILVER LAYER              │
│  state_features_silver              │
│  skill_gap_silver                   │
│  category_demand_silver             │
│  top_roles_silver                   │
│  exp_demand_silver                  │
│  Feature engineering · Joins        │
└──────────────┬──────────────────────┘
               │
     ┌─────────┴──────────┐
     ▼                    ▼
┌─────────────┐    ┌──────────────────┐
│  ML MODEL   │    │ STREAMING LAYER  │
│  GBT        │    │ Structured       │
│  Classifier │    │ Streaming        │
│  AUC: 1.0   │    │ Live job demand  │
│  MLflow     │    │ Delta sink       │
│  tracked    │    │                  │
└──────┬──────┘    └────────┬─────────┘
       │                    │
       └─────────┬──────────┘
                 ▼
┌─────────────────────────────────────┐
│            GOLD LAYER               │
│  state_predictions_gold             │
│  live_demand_gold                   │
│  dashboard_state_risk               │
│  dashboard_region_summary           │
│  dashboard_exp_gap                  │
│  dashboard_top_roles                │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│      DATABRICKS SQL DASHBOARD       │
│  5 visualizations · 3 KPI counters  │
│  Live · Published · Shareable       │
└─────────────────────────────────────┘
```

---

## 📊 Datasets

| Dataset | Source | Rows | Description |
|---|---|---|---|
| Unemployment Rate India | Kaggle (CMIE) | 267 | State-wise monthly unemployment rate, labour participation, employed count across 27 Indian states (Jan–Nov 2020) |
| Jobs in Data Science | Kaggle | 9,355 | Global data science job postings with title, category, salary, experience level, work setting (2020–2023) |

---

## 🔧 Tech Stack

| Component | Technology |
|---|---|
| Data Platform | Databricks Community Edition |
| Storage Format | Delta Lake |
| Processing Engine | Apache Spark (PySpark) |
| ML Framework | Spark MLlib (GBTClassifier) |
| Experiment Tracking | MLflow |
| Streaming | Structured Streaming |
| Orchestration | Databricks Jobs (Scheduled) |
| Dashboard | Databricks SQL Dashboard |
| Language | Python 3.12 |

---

## 📁 Project Structure

```
india-youth-employment-intelligence-databricks/
│
├── notebooks/
│   ├── 00_Setup.ipynb              # Data verification
│   ├── 01_Bronze.ipynb             # CSV → Delta, OPTIMIZE, Time Travel
│   ├── 02_Silver.ipynb             # Feature engineering, skill gap
│   ├── 03_ML.ipynb                 # GBT model, MLflow tracking
│   ├── 04_Streaming.ipynb          # Live job demand streaming
│   └── 05_Dashboard.ipynb          # Gold dashboard tables
│
├── README.md
└── LICENSE
```

---

## 🤖 ML Model

| Property | Value |
|---|---|
| Algorithm | Gradient Boosted Trees (GBT) Classifier |
| Label | `needs_intervention` (binary: 0/1) |
| Features | avg_unemployment_rate, peak_unemployment_rate, unemployment_volatility, avg_labour_participation, skill_gap_score, months_tracked |
| Train / Test split | 80% / 20% |
| AUC Score | 1.0 |
| Tracking | MLflow experiment logging |

**Note on AUC = 1.0:** The dataset has 27 states with clean, well-separated
unemployment patterns. Perfect separation is expected and valid at this scale.
A production version would use district-level data (700+ districts) for more
nuanced predictions.

---

## 📈 Key Findings

- **Highest unemployment states:** Tripura, Jharkhand, Bihar, Haryana, Himachal Pradesh
- **Most critical region:** Northeast and North India
- **Most in-demand job category:** Data Science and Research (3,014 openings)
- **Biggest experience gap:** 6,709 Senior roles vs only 496 Entry-level roles
- **Top paying role:** Data Science Tech Lead (~$200K+ avg)
- **States flagged for intervention:** Multiple High-risk states identified by ML model

---

## ⚡ Streaming Pipeline

Live job market demand is ingested via **Structured Streaming**:

```
jobs_bronze (Delta source)
      │
      ▼ readStream
Live aggregation by job_category + experience_level + work_setting
      │
      ▼ writeStream (complete mode)
ecommerce.gold.live_demand_gold (Delta sink)
```

Trigger: `availableNow=True` — processes all available data immediately,
simulating a real-time job posting feed updating the Gold layer.

---

## 🚀 How to Run

### Prerequisites
- Databricks account (Community Edition or above)
- Both datasets uploaded to your catalog

### Step 1 — Upload datasets
Upload both CSVs to your Databricks catalog:
- `Unemployment_Rate_upto_11_2020.csv`
- `jobs_in_data.csv`

### Step 2 — Run notebooks in order
```
00_Setup → 01_Bronze → 02_Silver → 03_ML → 04_Streaming → 05_Dashboard
```

### Step 3 — View dashboard
Open **Dashboards** in Databricks sidebar →
`India Youth Employment Intelligence`

---

## 🗄️ Delta Lake Features Used

| Feature | Where used |
|---|---|
| `OPTIMIZE` | Bronze layer — compacts small files |
| Time Travel | Bronze layer — version history verified |
| `saveAsTable` | All layers — managed Delta tables |
| Streaming sink | Gold layer — live demand table |
| Schema enforcement | All reads and writes |

---

## 📉 Dashboard

The published Databricks SQL Dashboard includes:

- **3 KPI Counters** — Total states, critical states, India avg unemployment
- **Unemployment Risk by State** — Color coded by risk level (High/Medium/Low)
- **Unemployment Rate by State** — Raw rate comparison across all 27 states
- **Jobs by Experience Level** — Entry vs Mid vs Senior demand gap
- **Live Job Demand by Category** — Pie chart from streaming Gold table
- **Top Paying Data Roles** — Salary benchmarks for career guidance

---

## 🎯 Real World Impact

This platform directly serves:

| Stakeholder | How they use it |
|---|---|
| **NSDC / Skill India** | Identify which states need skilling programs most urgently |
| **State Governments** | Target employment exchange resources to high-risk districts |
| **EdTech platforms** | Design courses aligned with actual job market demand |
| **Job seekers** | Understand which skills and roles offer best opportunities |
| **Policy makers** | Data-driven evidence for budget allocation decisions |

---

## 🏆 Built For

**Databricks 14-Day AI Challenge — Build With Databricks 2**
Organized by [Codebasics](https://codebasics.io) ×
[Indian Data Club](https://www.indiandataclub.com) ×
[Databricks](https://www.databricks.com)

---

## 👤 Author

**Sima Mishra**
- LinkedIn: [https://www.linkedin.com/in/sima-analyst/]

---
