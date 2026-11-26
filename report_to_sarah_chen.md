**To:** Sarah Chen, VP of Revenue
**From:** Lisha Bhowmik, BI Analyst
**Subject:** Revenue Intelligence Dashboard: findings, design and AI check

## Summary
Sarah, every marketing campaign shows attributed revenue below spend, but that revenue figure does not reconcile with CRM, so I would not move budget on it yet. Sales won 315 of 499 deals ($707,201). Support takes about as long to resolve critical tickets as low-priority ones. Eight high-value, high-ticket accounts merit an account review.

## Findings and actions

| Finding | Why it matters | Recommended action |
|---|---|---|
| Campaign spend $12.36M; attributed revenue $1.13M; ROI (return on investment: revenue minus spend, divided by spend) -90.9%. All six campaigns are negative. | Payback is not shown on current fields. Attributed revenue also exceeds total CRM won value ($707K). | Confirm how spend and attributed revenue are defined, and add a lead-to-deal key. |
| Direct Mail has the highest marketing-stage conversion (49.0%, 149 leads); Organic Search the lowest (0.9%, 671 leads). | Channels differ widely, but this does not show what changing budget would cause. | Check stage definitions, then run a controlled budget test. |
| Win rate 63.1%. Central 69.9%, West 58.8%, East 57.8%. | The regional gap may reflect deal mix or manager differences. This is exploratory. | Compare deal mix and practices before setting targets. |
| NovaPulse provides 77.6% of won value; Enterprise and Large accounts 83.6%. | Revenue is concentrated. | Include this concentration in account planning. |
| Largest loss reasons: No Decision Made and Poor Product Fit (43 each of 184). | Together they are 46.7% of losses. | Review these opportunities and qualification criteria. |
| Median resolution: critical 50.5 h, low 52.9 h. | Priority does not appear to shorten resolution much. SLA (service-level agreement: the agreed time limit for service) targets were not provided. | Check triage rules against agreed SLAs. |
| Data Pipeline has the highest high/critical share (39.7%); Authentication has the most users affected and downtime. | These areas are candidates for reliability work. Users may be counted on more than one ticket. | Review root causes with the product owners. |
| 8 accounts, holding $130,915 (18.5%) of won value, meet the review rule. | They combine high value, high volume and above-median negative sentiment. This is not a churn prediction. | Assign account owners, starting with Falcon Software (41.7% negative). |

## Design rationale
- **Three sheets** follow your three questions: marketing, sales and customer health. KPI cards come first, with breakdowns below.
- **Data model:** a row-level join (matching every detail record with every matching record in the other files) turns 499 deals into 63,420 rows. I keep each source at its own grain (what one row represents: one deal, lead or ticket), add account details through a one-record-per-account lookup, and aggregate before building the unified account view.
- **Chart choices:** bars compare categories, trends show change over time, and the account table supports follow-up.
- **Filters and actions:** [FILL: which controls work across sheets, which do not, and which actions and drill-downs you built]
- **Segment:** CRM company size is the shared segment. Marketing segment varies within accounts, so it filters marketing only.
- **What changed while building, and why:** [FILL]

## Effect of the Topic (QuickSight settings that tell Q what data fields mean)
- **Settings I configured:** [FILL]
- **Answers before the Topic:** [FILL]
- **Answers after the Topic:** [FILL]
- **What improved, stayed wrong or got worse:** [FILL]

## AI answers compared with the dashboard

| Question | Q after Topic | Dashboard | Match? | Reason for any difference |
|---|---|---|---|---|
| Highest-converting channel | [FILL] | [FILL] | [FILL] | [FILL] |
| Deal size by company size | [FILL] | [FILL] | [FILL] | [FILL] |
| Resolution: critical vs low | [FILL] | [FILL] | [FILL] | [FILL] |
| Top 10 accounts by tickets and their revenue | [FILL] | [FILL] | [FILL] | [FILL] |
| Campaigns spending more than earned | [FILL] | [FILL] | [FILL] | [FILL] |

**My recommendation on when to use Q or the dashboard:** [FILL]

## Limits
- The sources cover different date ranges.
- No key links a marketing lead to a CRM deal, so channel-level deal win rate is not available.
- Marketing `Closed Won` is not matched to CRM wins.
- Some deal and ticket IDs are reused across accounts; each row is treated as a separate record.
- 150 marketing rows and 204 support rows have no CRM account.
- Results are descriptive, with no forecasts or causal claims.
