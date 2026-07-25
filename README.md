 # Onboarding Funnel & Churn Analysis

![Python](https://img.shields.io/badge/Python-3.10+-blue?style=flat&logo=python)
![SQL](https://img.shields.io/badge/SQL-SQLite-003B57?style=flat&logo=sqlite)
![Tableau](https://img.shields.io/badge/Tableau-Public-E97627?style=flat&logo=tableau)
![Plotly](https://img.shields.io/badge/Plotly-Data%20Viz-3F4F75?style=flat&logo=plotly)

A product analytics case study analyzing user drop-off across a 4-stage onboarding funnel. Synthesized 2500 user registration logs in Python, analyzed conversion bottlenecks using SQL window functions (`LAG`, `FIRST_VALUE`), and visualized user flows in Plotly and Tableau Public with product retention strategies.


# Executive Summary

**Primary Bottleneck:** Identified a major conversion drop at **Step 3 (ID Verification)**, where user progression dropped by **~52.9%** compared to the previous step (overall conversion fell from 85% to ~40%).
**Root Cause:** Analysis of explicit failure logs vs. drop-offs reveals that **abandonment during ID upload** (friction/latency) accounts for the vast majority of lost users, rather than technical document rejection.
**Product Recommendation:** Proposed a **"Preview Mode"** during ID processing to keep users engaged with core value props (FX rate tracking, vault previews) while identity checks complete asynchronously.

**[Click Here to View the Interactive Tableau Dashboard](https://public.tableau.com/views/YOUR_DASHBOARD_NAME/Dashboard1https://public.tableau.com/views/Revolut-Onboarding-Funnel-Analysis_/Dashboard1?:language=en-US&:sid=&:redirect=auth&:display_count=n&:origin=viz_share_link)**

# Project Architecture & Data Pipeline
```
┌─────────────────────────┐     ┌─────────────────────────┐     ┌─────────────────────────┐
│     Synthetic Data      │ ──> │      SQLite Engine      │ ──> │  Visualization Layer    │
│  (Pandas, Faker, NumPy) │     │ (CTEs & Window Functions│     │ (Plotly & Tableau)      │
└─────────────────────────┘     └─────────────────────────┘     └─────────────────────────┘
```

The data model for real-world fintech user activation across 2 tables:
1. **`users`**: Contains `user_id`, `signup_timestamp`, `country` (UK, PL, PT, UAE, ES), and `acquisition_channel` (Paid Ad, Organic, Referral, Influencer).
2. **`funnel_events`**: Sequential timestamped logs tracking:
   * `1_app_download` (100% baseline)
   * `2_phone_verified` (~85% retention)
   * `3_id_uploaded` (~45% attempt rate; explicit success/failure tagging)
   * `4_account_activated` (~90% pass rate for successful IDs)

# Onboarding Funnel Performance

| Funnel Step | Users Started | Users Completed | Explicit Failures | Total Drop-off | Overall Conv. (%) | Step Retention (%) |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: |
| **1. App Download** | 2,500 | 2,500 | 0 | 0 | **100.0%** | Baseline |
| **2. Phone Verified** | 2,500 | 2,125 | 0 | 375 | **85.0%** | **85.0%** |
| **3. ID Uploaded** | 2,125 | 1,000 | 125 | 1,125 | **40.0%** | **47.1%** *(Major Bottleneck)* |
| **4. Account Activated**| 1,000 | 900 | 0 | 100 | **36.0%** | **90.0%** |

# SQL Pipeline (`SQLite`)

The following SQL query processes raw event logs using window functions to dynamically compute step-over-step retention and baseline conversion without hardcoding thresholds:

```sql
WITH funnel_counts AS (
    SELECT 
        event_name,
        COUNT(DISTINCT user_id) AS user_count,
        CASE event_name
            WHEN '1_app_download' THEN 1
            WHEN '2_phone_verified' THEN 2
            WHEN '3_id_uploaded' THEN 3
            WHEN '4_account_activated' THEN 4
        END AS step_order
    FROM funnel_events
    WHERE status = 'success'
    GROUP BY event_name
)
SELECT 
    event_name,
    user_count,
    ROUND(
        (user_count * 100.0) / FIRST_VALUE(user_count) OVER (ORDER BY step_order), 
        2
    ) AS overall_conversion_pct,
    ROUND(
        (user_count * 100.0) / LAG(user_count, 1) OVER (ORDER BY step_order), 
        2
    ) AS step_retention_pct
FROM funnel_counts
ORDER BY step_order;
```

# Key Product Insights & Recommendations

1. **Reduce ID Upload Friction:**
   * Integrate real-time document quality checks (blur/lighting detection) prior to submission to reduce explicit rejection rates.
2. **Implement Asynchronous "Preview Mode":**
   * Allow users to explore read-only app features (e.g., live exchange rate alerts, savings vaults) while waiting for document verification rather than displaying a static loading screen.
3. **Targeted Re-engagement Campaigns:**
   * Trigger push notifications/SMS reminders 2 hours post-phone verification for users who stall before uploading ID documents.

# Repository Structure

```
.
├── notebooks/
│   └── revolut_funnel_analysis.ipynb   # Complete Python pipeline & SQL execution    
├── assets/
│   └── funnel_chart.png                # Plotly/Tableau dashboard screenshots & Processed CSV output for Tableau Public
│   └── tableu.png
│   └── tableau_funnel_data.csv        
├── README.md                           # Project documentation
└── LICENSE
```

# How to Run

1. Clone the repository:
   ```bash
   git clone https://github.com/your-username/revolut-funnel-analysis.git
   ```
2. Open `notebooks/revolut_funnel_analysis.ipynb` in Google Colab or Jupyter Notebook.
3. Run all cells to generate synthetic logs, execute SQL analysis, and display Plotly funnel charts.
