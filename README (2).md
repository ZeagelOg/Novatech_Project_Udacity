# NovaTech Revenue Intelligence: Amazon QuickSight Project

**Author:** <your name>
**Date:** <date>
**Project:** Udacity, Data Preparation, Dashboard Design and AI-Powered Analysis with Amazon QuickSight

## About

NovaTech's VP of Revenue, Sarah Chen, wants one view of marketing, sales and customer support. This repository holds my project work: data preparation and a three-sheet dashboard in Amazon QuickSight, a test of QuickSight Q (natural-language querying), a written report for Sarah Chen, and a supporting Python and SQL analysis that checks the numbers.

**Tools:** Amazon QuickSight (SPICE, Q Topics, Quick Chat), Python (pandas, NumPy, SciPy, Matplotlib), SQL (SQLite), pytest

## Repository contents

| Path | What it is |
|---|---|
| `report/report_to_sarah_chen.md` | Report for Sarah Chen: findings, design rationale, Topic effect, AI vs dashboard |
| `logs/verification_log.md` | Quick Chat data verification log (7 questions across CRM, marketing, support and reference documents) |
| `logs/q_exploration_log.md` | Q exploration log (5 questions, before and after Topic) |
| `logs/answer_key.md` | Values calculated from the raw CSVs, used to check Q's answers |
| `docs/01_build_guide.md` | Step-by-step build plan for the datasets, dashboard and Topic |
| `docs/screenshot_checklist.md` | List of screenshots captured and their file names |
| `supporting_analysis/` | Python and SQL reanalysis of the source data, with charts, result tables and tests |
| `screenshots/` | QuickSight screenshots from my own session (data prep, dashboard, Q and Topic) |
| `export/NovaTech_Dashboard.pdf` | Published dashboard exported as PDF (all three sheets) |

## Data model

- **Source files:** CRM deals (499 rows), marketing leads (2,240) and support tickets (3,000).
- **Problem:** joining all three on `account_id` directly turns 499 deals into 63,420 rows, because each table repeats account IDs.
- **Solution:** aggregate each source to one row per account, then join. The anchor table is the CRM account aggregate, joined with left joins on `account_id`, so every CRM account stays in the view even with no leads or tickets.
- **Result:** an 85-row unified dataset. Won value should check to $707,201.
- **Segment:** CRM company size tier is the shared segment, because marketing segment varies within accounts.

## Dashboard

Three sheets answer three questions:

1. **Marketing Funnel:** which channels and campaigns convert, and what do they cost?
2. **Sales Pipeline:** how are deals won and lost, by region, product and company size?
3. **Customer Health:** how quickly are tickets resolved, and which accounts need attention?

Interactivity: filter controls, a filter action on Sales Pipeline, and a navigation action from Sales Pipeline to Customer Health.

## Key findings

> Confirm each number against the published QuickSight dashboard before finalizing.

- Sales won 315 of 499 deals (63.1%), worth $707,201.
- Every marketing campaign shows attributed revenue below spend (overall ROI -90.9%), and that revenue does not reconcile with CRM. Budget decisions should wait until the definitions are confirmed.
- Critical tickets take about as long to resolve as low-priority tickets (median 50.5 h vs 52.9 h).
- Eight accounts, holding 18.5% of won value, combine high value, high ticket volume and above-median negative sentiment, and merit an account review.

## Supporting analysis

`supporting_analysis/` reanalyses the raw files in Python and SQL. It checks my QuickSight figures and shows how a direct three-way join distorts totals and rankings. It also audits a published QuickSight dashboard from a different project: [Chiragchhabra1127/revenue-intelligence-dashboard](https://github.com/Chiragchhabra1127/revenue-intelligence-dashboard) (commit `c891ad4`). That dashboard is the other author's work and is credited here, not claimed as mine. The raw course data is not redistributed. See `supporting_analysis/README.md` to run it.

## Limitations

- The three sources cover different date ranges.
- No key links a marketing lead to a CRM deal, so channel-level deal win rate is not available.
- Marketing `Closed Won` is not matched to CRM wins.
- Some deal and ticket IDs are reused across accounts; each row is treated as a separate record.
- 150 marketing rows and 204 support rows have no CRM account.
- Results are descriptive, with no forecasts or causal claims.

## Process and AI assistance

- QuickSight screenshots, Quick Chat answers and Topic settings come from my own QuickSight session.
- Analysis, documentation and drafts were prepared with help from Claude, an AI assistant. I reviewed the results and, where I made changes, checked them against my own dashboard and the raw data.
