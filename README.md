# Corvia Forecasting Engine

## Problem Statement

**What's broken**

Corvia's VP of Sales submits a quarterly forecast to the board. Last quarter: €4.2M forecast, €3.1M actual - a 26% miss. The quarter before: €3.8M forecast, €4.9M actual - a 29% positive variance that nobody predicted either. Neither miss was investigated or explained.

The VP has no model, no stage-velocity analysis, and no comparison of rep-submitted commit vs. what the data would predict. The forecast is a bottom-up survey of rep opinions, submitted in a Salesforce field, exported to Excel, manually colour-coded by Siobhán, and presented to a board that's losing confidence.

**Who owns the pain**

- VP of Sales - embarrassed every quarter in front of the board.
- CFO - cannot accurately plan headcount or expenses against revenue she doesn't trust.
- Siobhán (RevOps Manager) - rebuilds the forecast in Excel every Monday, every week, manually.

**Financial impact**

A 26% miss on a €4.5M quarterly target equals a €1.17M unforecasted revenue gap. Finance allocated €680,000 in Q3 expenses (two new hires, a conference sponsorship, an office expansion) based on a forecast that didn't materialise. The company is now managing a cash shortfall.

---

## Repository Structure

```
corvia-forecasting-engine/
|-- README.md
|-- data/
|   `-- synthetic_data_generator/   (empty - synthetic data generation for forecasting scenarios)
|-- src/                            (core application and modeling code)
|-- notebooks/                      (exploratory analysis and prototyping)
|-- docs/                           (additional documentation)
`-- tests/                          (test suite)
```
