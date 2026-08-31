# Corvia Forecasting Intelligence Engine

> **Company:** Corvia Technologies Ltd. — B2B SaaS, Compliance & Contract Intelligence, Dublin HQ  
> **Role simulated:** Revenue Operations Analyst  
> **Data:** Synthetic data generated to mirror a real Salesforce + HubSpot CRM environment  

---

## The Business Problem

Corvia's VP of Sales submits a quarterly revenue forecast to the board every quarter. In the last four quarters, that forecast has been wrong by an average of **27%** — missing by €1.1M in Q3 2023, beating by €800K in Q4 2023, missing by €900K in Q3 2024. No quarter has been explained. No quarter has been learned from.

The forecast is built the same way every time: the RevOps Manager exports open opportunities from Salesforce, asks each AE to classify their deals into "Commit," "Best Case," or "Pipeline," and totals the Commit column. The resulting number goes to the board. There is no model. There is no validation. There is no comparison between what reps say and what historical data would predict.

The consequences are not abstract:
- **€680,000** in Q3 2024 expenses were allocated based on a forecast that didn't materialise
- **3 headcount decisions** were made under a revenue assumption that turned out to be 26% optimistic
- The board has stopped trusting the number — and a board that doesn't trust revenue forecasts starts asking harder questions about the entire business

The root causes are three:

| Root cause | Symptom |
|-----------|---------|
| No stage-velocity model | Nobody knows which stage is the bottleneck or how long deals *should* take |
| No rep-vs-model comparison | Sandbagging and deal bloat are invisible; forecast inherits all rep bias |
| No at-risk early warning | Stalled deals are only visible when they slip out of a quarter, not 6 weeks before |

---

## The Solution

This project builds a **pipeline forecasting intelligence layer** that sits on top of Corvia's Salesforce data and produces:

1. **A model-generated (top-down) forecast** — derived from historical stage win rates and deal velocity, applied to current open pipeline. No rep input required. No sandbagging.

2. **A rep-submitted (bottom-up) forecast** — pulled directly from CRM commit/best-case classifications.

3. **Side-by-side comparison** — when the model and reps disagree by >15%, that's a signal requiring VP investigation, not a number to average and ignore.

4. **At-risk deal identification** — opportunities flagged when time-in-current-stage exceeds 1.5× the historical average for their segment and stage. These are the deals that slip quarters.

5. **AI-generated weekly forecast commentary** — a natural-language summary produced by Claude API: current pipeline coverage, velocity vs prior quarter, at-risk deal list with recommended actions.

6. **Quarterly forecast debrief document** — what the model predicted, what reps committed, what actually closed, and the structural reasons for the variance. This is the document a RevOps team should produce after every quarter and almost never does.

---

## Architecture

```
Salesforce (CRM)
      │
      ▼
Synthetic Data Layer        ← generate_data.py produces CSVs mirroring Salesforce exports
(fact_opportunities,
 fact_stage_history,
 dim_users, dim_accounts)
      │
      ▼
SQL Transformation Layer    ← staging → intermediate → mart models
(sql/staging, sql/marts)
      │
      ├──► Stage Velocity Analysis    ← avg days per stage, outlier detection
      ├──► Weighted Pipeline Model    ← ACV × historical win prob by stage
      ├──► Forecast Accuracy Model    ← rep-submitted vs model vs actuals (historical)
      └──► At-Risk Deal Flags         ← time-in-stage > 1.5× segment average
      │
      ▼
Python Analysis Layer       ← src/forecasting/
(forecast_model.py,
 stage_velocity.py,
 weighted_pipeline.py)
      │
      ├──► Power BI Dashboard         ← dashboard/
      └──► AI Commentary              ← src/ai_commentary/ (Claude API)
```

---

## Key Findings

*(Generated from 24 months of synthetic Corvia pipeline data — Jan 2023 to Dec 2024)*

| Finding | Detail |
|---------|--------|
| **Forecast variance** | Rep-submitted commit overstates actuals by 24–32% across all 8 quarters |
| **Primary sandbagging stage** | Stage 5 (Proposal): 41% of Stage 5 deals marked "Best Case" that historically convert at 58% |
| **Biggest velocity bottleneck** | Stage 4 (Demo/Evaluation): MM deals average 15 days but stalled deals average 34 days — 2.3× |
| **Slippage root cause** | 61% of slipped deals had time-in-stage > 1.5× average at Stage 4 or 5 — visible 3+ weeks before slip |
| **Highest win rate territory** | UK/Ireland: 31%. Lowest: DACH: 24% — despite DACH having 40% higher average ACV |
| **Coverage gap** | Pipeline coverage at 2.8× against a required 3.6× — a structural gap, not a pipeline quality issue |

---

## Repository Structure

```
corvia-forecasting-engine/
├── README.md                          ← You are here
├── requirements.txt
│
├── data/
│   ├── synthetic_data_generator/
│   │   └── generate_data.py           ← Generates all synthetic CRM data
│   └── sample/                        ← Generated output files (gitignored in prod)
│       ├── dim_users.csv
│       ├── dim_accounts.csv
│       ├── fact_opportunities.csv
│       ├── fact_stage_history.csv
│       └── fact_forecast_submissions.csv
│
├── sql/
│   ├── staging/                       ← Light cleaning of raw source data
│   ├── marts/                         ← Business-ready analytical tables
│   └── analysis/                      ← Ad-hoc analysis queries
│
├── src/
│   ├── forecasting/
│   │   ├── weighted_pipeline.py       ← Top-down model-generated forecast
│   │   ├── stage_velocity.py          ← Stage duration analysis + at-risk flags
│   │   └── forecast_model.py          ← Main forecasting engine (combines both)
│   └── ai_commentary/
│       └── generate_commentary.py     ← Weekly AI narrative via Claude API
│
├── notebooks/
│   └── 01_eda_pipeline_analysis.ipynb ← Exploratory analysis with visualisations
│
├── dashboard/
│   └── screenshots/                   ← Power BI dashboard exports
│
└── docs/
    ├── problem-statement.md
    ├── data-dictionary.md
    └── ai-governance.md
```

---

## How to Run

```bash
# 1. Clone and install dependencies
git clone https://github.com/[your-username]/corvia-forecasting-engine
cd corvia-forecasting-engine
pip install -r requirements.txt

# 2. Generate synthetic data
python data/synthetic_data_generator/generate_data.py

# 3. Run forecasting analysis
python src/forecasting/forecast_model.py

# 4. Generate AI commentary (requires Anthropic API key)
export ANTHROPIC_API_KEY=your_key_here
python src/ai_commentary/generate_commentary.py
```

---

## Data & Assumptions

All data in this project is **synthetically generated** to mirror real Salesforce CRM output for a fictional B2B SaaS company (Corvia Technologies Ltd.). No real customer or commercial data is used.

Key generation parameters:
- 24 months of pipeline history: January 2023 – December 2024
- ~95 new opportunities created per month (±12 seasonal variation)
- Win rates: SMB 35%, Mid-Market 30%, Enterprise 18%
- ACV ranges: SMB €6k–€18k, MM €20k–€75k, Enterprise €80k–€300k
- Sales cycles: SMB 42 days, MM 78 days, Enterprise 154 days

In a real deployment, `fact_opportunities` and `fact_stage_history` would be pulled directly from Salesforce using the Bulk API or a reverse ETL tool (e.g., Airbyte → Snowflake).

See `docs/data-dictionary.md` for full field definitions.

---

## What This Demonstrates

| Stakeholder | Evidence in this project |
|------------|--------------------------|
| **RevOps Manager** | Understanding of forecast methodology, pipeline hygiene, stage velocity, and how to build a validation layer on top of rep-submitted data |
| **VP of Sales** | At-risk deal identification before slippage; model vs rep forecast comparison for focused pipeline review conversations |
| **CFO / Finance** | Forecast accuracy improvement from ~72% to a modeled 84%+; quantified impact of coverage gap |
| **Analytics Manager** | Clean data model, tested SQL marts, assumption documentation, model validation approach |

---

## Honest Limitations

- Data is synthetic. Stage durations, win rates, and ACV distributions are calibrated to realistic SaaS benchmarks but do not represent any real company's performance.
- The AI commentary uses a simplified prompt architecture. In production, this would incorporate live CRM data, rep-level context, and a retrieval layer over historical commentary.
- The Power BI dashboard uses imported CSV data. In production, this would connect to a Snowflake or BigQuery data warehouse via DirectQuery.
- This project does not include a Salesforce → warehouse data pipeline (ETL/ELT). That would be a separate infrastructure project.
