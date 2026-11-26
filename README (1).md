# NovaTech Revenue Intelligence: Dashboard Audit and Corrected Analysis

An audit of an Amazon QuickSight revenue dashboard, followed by a reanalysis of its source data in Python and SQL.

**Main finding:** joining CRM deals to marketing leads and support tickets on `account_id` alone turned **499 deals into 63,420 rows**. Rebuilding that join reproduces all 13 dashboard KPIs tested. It inflates totals and changes which region, sales rep and campaign appear to lead.

## Credit and scope

- This repository contains the audit and reanalysis: code, SQL, tests, charts and reports.
- NovaTech Solutions is the company in the project's course scenario. The raw course files are not redistributed (see [`data/README.md`](data/README.md)).
- Analysis and documentation were prepared with help from Claude, an AI assistant.

## What the join changed

| Metric | Dashboard | Corrected (raw table) |
|---|---|---|
| Deal value (won deals only) | $100.9M | **$707,201** |
| Campaign spend | $353.51M | **$12.36M** |
| Attributed revenue | $25.05M | **$1.13M** |
| Campaign responses | 14.1K | **609** |
| Support tickets | 2.79K | **3,000** |
| Users affected | 13.74M | **758,557** |
| Downtime minutes | 1.24M | **47,255** |

| "Who leads?" | Dashboard | Corrected |
|---|---|---|
| Region by deal value | West | **Central** |
| Sales rep by deal value | Priya Nair | **Owen McCarthy** (Priya Nair is 11th) |
| Least-negative campaign ROI | Digital Retarget | **NovaPulse Launch** |
| Product area by downtime | Notifications | **Authentication** |

![Campaign ROI: joined vs raw](reports/figures/01_roi_by_campaign.png)

## Corrected findings

### Marketing
- Overall ROI on the attributed-revenue field is **-90.9%**. Every channel and campaign is negative.
- Direct Mail has the least-negative ROI (-70.7%) and the lowest spend per `Closed Won` lead ($9,881), on a small base of 149 leads. Organic Search is highest at $256,709.
- Response rate is **27.2%**. NovaEdge Awareness is far lowest at 6.2%.

![Spend per Closed Won lead](reports/figures/03_spend_per_closed_won.png)

### Sales
- **315 of 499 deals won (63.1%)**. Won value is $707,201 and the average won deal is $2,245.
- NovaPulse supplies **77.6%** of won value. Enterprise and Large accounts supply **83.6%**.
- Central wins 69.9% of deals, against 58.8% in West and 57.8% in East (overall test, p = 0.031). This is exploratory: several comparisons were run, and manager is nested within region.
- The largest loss reasons are No Decision Made and Poor Product Fit (23.4% of losses each).

![Win rate by region](reports/figures/04_win_rate_region.png)

### Customer health
- Median resolution time is **51.9 hours**. It is about the same for critical (50.5 h) and low-priority (52.9 h) tickets.
- Data Pipeline has the highest share of high or critical tickets (39.7%). Authentication has the most users affected and downtime minutes.
- Sentiment: 65.1% neutral, 22.8% negative, 10.1% positive, 2.0% blank.
- A screening rule flags **8 accounts** with $130,915 in won value (18.5%) for review. It is not a churn prediction.

![Resolution time by priority](reports/figures/05_resolution_by_priority.png)

## Read before using these numbers

- **Attribution does not reconcile with CRM.** Attributed revenue ($1.13M) exceeds all CRM won value ($707K), even after removing $79K from accounts not in CRM. The date windows and revenue definitions may differ, but the files do not establish how. Do not set budgets from ROI alone.
- **No key links a lead to a deal.** Channel-to-deal win rate cannot be measured from these files.
- **Funnel and response fields are linked.** All `Qualified Lead` rows are non-responders; all `Opportunity` and `Closed Won` rows are responders. `Closed Won` is a marketing field and is not matched to a CRM deal.

Full detail: [analysis report](reports/analysis_report.md) · [dashboard audit](reports/dashboard_audit.md)

## Method

1. Profile each table and run data-quality checks.
2. Compute each metric on its own source table.
3. For cross-source views, aggregate each table to one row per account, **then** join.
4. Rebuild the original join separately to reconcile the dashboard figures.
5. Test the account-level totals, and check that the SQL and pandas results match.

```text
CRM ---------> GROUP BY account_id --+
Marketing ---> GROUP BY account_id --+--> one row per CRM account (85)
Support -----> GROUP BY account_id --+
```

The account view keeps CRM accounts only. It excludes 150 marketing rows and 204 support rows with no CRM match; table-level metrics keep them.

## Run it

Requires Python 3.10 or later. First copy the three CSVs into `data/raw/`.

```bash
python -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install -r requirements.txt
python run_analysis.py             # writes reports/tables and reports/figures
python -m pytest -q
```

## Structure

```text
├── run_analysis.py          # entry point
├── src/
│   ├── data.py              # loading and quality checks
│   ├── metrics.py           # corrected metrics
│   ├── audit.py             # join reconstruction
│   └── charts.py            # figures
├── sql/account_summary.sql  # aggregate-then-join (SQLite)
├── tests/test_metrics.py
├── reports/                 # reports, figures, result tables
└── data/README.md           # how to get the raw files
```

**Tools:** Python (pandas, NumPy, SciPy, Matplotlib), SQL (SQLite), pytest.
