# T1 Project Plan

## Project Goal

Develop a clear and reproducible research workflow for comparing three familiar asset classes represented by the illustrative ETFs `SPY` (US equities), `TLT` (long-term US Treasury bonds), and `GLD` (gold). The project starts with this written plan in T1, and later tutorials can reuse the same repository to design a bounded analysis task and organize a verifiable agent workflow.

## Available Data

`data/etf_snapshot.csv` contains one row per illustrative ETF with the columns documented in `data/data_dictionary.md`:

| Column | Meaning |
|---|---|
| `ticker` | Short identifier for the illustrative ETF |
| `asset_class` | Broad type of asset represented by the ETF |
| `expected_return_pct` | Illustrative annual return assumption (percent per year) |
| `volatility_pct` | Illustrative annual variability assumption (percent per year) |
| `max_drawdown_pct` | Illustrative largest peak-to-trough loss; negative values represent losses (percent) |
| `expense_ratio_pct` | Illustrative annual fund fee (percent per year) |

The three rows are: `SPY` (8.2% return, 18% volatility, -24% drawdown, 0.09% fee), `TLT` (4%, 14%, -18%, 0.15%), and `GLD` (5.5%, 15.5%, -16%, 0.4%). The dataset is synthetic teaching data: the values are illustrative teaching inputs, not live or historical market observations.

## Expected Final Deliverable

A concise, reproducible research workflow that compares the three asset classes using the fixed snapshot dataset. This includes this project plan, a bounded analysis task design, and a verifiable agent workflow with outputs (tables, charts, and conclusions) that can be checked against the source data. All analysis beyond this plan is planned work and has not been completed yet.

## Three Project Milestones

1. **Project setup (T1)** — Create this project plan, review the repository overview and data dictionary, and confirm the environment is ready. This milestone is currently in progress.
2. **Analysis design (planned)** — Define a bounded analysis task (e.g., comparing expected return, volatility, drawdown, and fees across the three ETFs) and organize a verifiable agent workflow.
3. **Final deliverable (planned)** — Run the analysis, produce the comparison outputs, and verify the results against the source data.

## One Data Limitation

All numeric values in the dataset are synthetic teaching assumptions. They are not current quotations, verified historical estimates, or forecasts, and the dataset omits correlations, taxes, transaction costs, liquidity, currency exposure, and investor-specific constraints. The data must not be used as investment advice or as the basis for a real investment decision.

## Next Action

Review this plan, then save it with Git as part of the T1 tutorial. JiuWenSwarm does not commit or push the change during T1; the student performs the version-control step. After the plan is saved, proceed to the next tutorial task that builds on this repository.
