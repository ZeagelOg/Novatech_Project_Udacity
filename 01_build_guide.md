# QuickSight build guide

Menu names may differ slightly from your version. Capture each screenshot named in **bold** as you go.
This model has not been run in QuickSight; its row counts and expected values were calculated in pandas.

## Part A: Data preparation

### A1. Import and type the three CSVs (SPICE)
Upload each CSV from `Structured Data/`. Capture each import showing **row and column counts**. Before adding columns, the files have:

| File | Rows | Columns | Screenshot |
|---|---|---|---|
| `novatech_crm_deals` | 499 | 20 | **`A1a_import_crm.png`** |
| `novatech_marketing_campaigns` | 2,240 | 20 | **`A1b_import_marketing.png`** |
| `novatech_support_tickets` | 3,000 | 20 | **`A1c_import_support.png`** |

If a screen does not show both counts, add a numbered shot of the one that does. Set these types:

| Dataset | Date | Integer | Decimal |
|---|---|---|---|
| CRM | `deal_created_date`, `deal_closed_date` (yyyy-MM-dd) | `employee_count`, `list_price` | `annual_revenue_usd`, `deal_value` |
| Marketing | `campaign_date` | `campaign_response`, `complaint_flag`, `web_visits_per_month`, `household_size`, `days_since_last_engagement`, `product_spend_tier1` to `3` | `campaign_spend`, `revenue_attributed`, `annual_income` |
| Support | `ticket_created_date` (yyyy-MM-dd), `ticket_resolved_date` (yyyy-MM-dd HH:mm:ss) | `users_affected`, `downtime_minutes`, `tickets_last_30_days`, `payment_impact`, `security_incident`, `data_loss` | `error_rate_pct` |

Add calculated columns:

```text
CRM        days_to_close    = dateDiff({deal_created_date}, {deal_closed_date}, 'DD')
CRM        won_flag         = ifelse({deal_stage} = 'Won', 1, 0)
Marketing  closed_won_flag  = ifelse({funnel_stage} = 'Closed Won', 1, 0)
Marketing  campaign_roi     = ({revenue_attributed} - {campaign_spend}) / {campaign_spend}
Support    resolution_hours = dateDiff({ticket_created_date}, {ticket_resolved_date}, 'MI') / 60
Support    negative_flag    = ifelse(coalesce({customer_sentiment}, '') = 'negative', 1, 0)
Support    rated_flag       = ifelse(coalesce({customer_sentiment}, '') = '', 0, 1)
```

**`A1_spice_datasets.png`**, **`A2_types_crm.png`**, **`A3_types_marketing.png`**, **`A4_types_support.png`**, **`A5_calculated_fields.png`** (use extra numbered shots if one screen is not enough).

### A2. Why not join the raw rows
Joining all three row-level tables on `account_id` produces **63,420 rows** from 499 deals. Do not do this.

### A3. Account dimension
From CRM: **Aggregate**, grouped by `account_id`, `company_name`, `industry`, `company_size_tier`. If a measure is required, count `opportunity_id`. Save as `account_dim`. **Expected: 85 rows.**

### A4. Enriched fact datasets
Left join each fact table to `account_dim` on `account_id`. Each lead or ticket matches at most one account. Keep the fact table's `account_id` and drop the duplicate.

| Saved dataset | Expected rows | Rows with no account match |
|---|---|---|
| `NovaTech Marketing` | 2,240 | 150 |
| `NovaTech Support` | 3,000 | 204 |

Use the prepared CRM dataset (499 rows) for sales.

### A5. NovaTech Unified Dataset (one row per account)
1. CRM: aggregate by `account_id`, `company_name`, `industry`, `company_size_tier`: count `opportunity_id` as `deals`; sum `won_flag` as `won_deals`; sum `deal_value` as `won_value`.
2. Marketing: aggregate by `account_id`: count `lead_id` as `leads`; sum `campaign_spend`, `revenue_attributed`.
3. Support: aggregate by `account_id`: count `ticket_id` as `tickets`; sum `negative_flag` as `negative_tickets`; sum `rated_flag` as `rated_tickets`.
4. Left join 1 to 2, then to 3, on `account_id`. **Expected: 85 rows, `won_value` totals 707,201.**
5. Add:

```text
negative_pct = 100 * {negative_tickets} / {rated_tickets}
at_risk      = ifelse({won_value} >= 12716 AND {tickets} >= 37 AND {negative_pct} > 21.665, 'At risk', 'Not flagged')
```

The cut-offs are the top quarter for value and tickets, and the median negative share, fixed at their current values. Recalculate them if the data changes.

If your rubric requires one joined dataset with every row-level field, ask the reviewer how repeated rows should be handled.

**`A6_account_dim.png`**, **`A7_join_marketing.png`**, **`A8_join_support.png`**, **`A9_unified_join_diagram.png`**, **`A10_unified_preview_rowcount.png`**, **`A11_saved_datasets.png`** (must show the three source datasets and `NovaTech Unified Dataset` saved to SPICE)

## Part B: Dashboard

Create one analysis with all four datasets (three fact datasets plus unified).

Analysis calculated fields:

```text
Win Rate        = sum({won_flag}) / count({opportunity_id})
Avg Won Deal    = sum({deal_value}) / sum({won_flag})
Response Rate   = sum({campaign_response}) / count({lead_id})
Conversion Rate = sum({closed_won_flag}) / count({lead_id})
ROI             = (sum({revenue_attributed}) - sum({campaign_spend})) / sum({campaign_spend})
Negative Share  = sum({negative_flag}) / count({account_id})
```

Format ratios as percentages. Do **not** average the row-level `campaign_roi`. Use Count, not Count distinct, for deals and tickets: some IDs are reused across accounts.

### Sheet 1: Marketing Funnel (`NovaTech Marketing`)
| KPI | Expected |
|---|---|
| Total leads | 2,240 |
| Responses | 609 |
| Response rate | 27.2% |
| Campaign spend | $12.36M |
| Attributed revenue | $1.13M |
| ROI | -90.9% |

Visuals:
- **Funnel Stage by Channel**: stacked bar of lead count by `campaign_channel` and `funnel_stage`
- Funnel of current stage counts: Prospect 404, Lead 465, Qualified Lead 848, Opportunity 111, Closed Won 412 (set this order; label it "current stage")
- Conversion Rate by channel
- **Spend vs. Revenue** combo chart by `campaign_name`, using one shared axis if available
- ROI by campaign
- Response Rate by campaign

Filters: `campaign_name`, `campaign_channel`, `campaign_date`, `customer_segment`.

### Sheet 2: Sales Pipeline (CRM)
| KPI | Expected |
|---|---|
| Deals | 499 |
| Won value | $707,201 |
| Win rate | 63.1% |
| Average won deal | $2,245 |
| Average days to close | 66.8 |

Label the money "Won value": lost deals are 0 and there are no open deals.

Visuals: Won vs Lost counts; loss reasons (filter to Lost); **Revenue by Company Size** (stack by `product_category`); win rate by region, rep and product; average days to close by closed month; a table of average won deal by product, size and region. Add average `deal_value` by company size to a tooltip or table so the Q log has an all-deals comparison.

Filters: `sales_region`, `sales_manager`, `product_category`, `deal_created_date`, `deal_stage`.

### Sheet 3: Customer Health (`NovaTech Support` and unified)
| KPI | Expected |
|---|---|
| Tickets | 3,000 |
| Unresolved | 59 |
| Median resolution hours | about 51.9 |
| Negative share | 22.8% |
| At-risk accounts (unified) | 8 |

For Unresolved, count tickets with a visual filter for null `ticket_resolved_date`.

Visuals:
- **Resolution Time by Priority**: average and median `resolution_hours`
- Tickets and resolution time by product area
- Sentiment distribution
- Tickets against won value by account (scatter, unified)
- **At-Risk Accounts** table (unified): all accounts, sorted by `tickets`, showing company, won value, tickets, negative % and `at_risk`; highlight flagged rows. Keeping all rows lets you check the top 10 by tickets.

Filters: `account_id`, `priority`, `product_area`, `customer_tier`, `region`.

### Cross-sheet filters and actions
- Create `company_size_tier` and `account_id` controls and apply them across datasets and sheets, where your version allows. Test each visual. Rows without a CRM match drop out when a size filter is selected.
- `customer_segment` exists only in marketing and varies within accounts. The brief's example of a segment filter updating every view cannot use it reliably. Use CRM company size as the shared segment and say so in the report. A sales-region filter also does not automatically apply to the other datasets.
- Put filter controls on all three sheets (the reviewer requires at least two).
- **One-click filter actions on at least two visuals:** select the visual, open **Actions**, add a **Filter action**, set activation to **On select**, and target a detail table on the same sheet. Do this on one visual each in Marketing and Sales. Capture the setup and the result.
- **Cross-sheet navigation action:** on a Sales visual, add an action of type **Navigation**, with Customer Health as the target sheet. Test it and capture the setup, the source sheet and the destination. Navigation alone does not necessarily carry the filter across.
- Add a drill-down hierarchy on one chart per sheet, such as channel to campaign, region to rep, and product area to priority.

### Annotations (required: 3 to 5)
Add each as a text box beside its visual. **Check every number against your dashboard with all filters cleared** before adding it. If yours differs, find out why before using the text. Label annotations "All data", because text does not change with filters.

1. **Marketing:** Finding: attributed revenue is $1.13M against $12.36M spend (ROI, return on investment, -90.9%). Implication: payback is not shown, and this revenue does not reconcile with CRM. Action: confirm attribution definitions before changing budgets.
2. **Sales:** Finding: No Decision Made and Poor Product Fit are 86 of 184 losses (46.7%). Implication: two reasons account for nearly half of losses. Action: review those deals and qualification criteria.
3. **Sales:** Finding: NovaPulse is 77.6% of won value. Implication: revenue depends heavily on one product line. Action: include this concentration in account planning.
4. **Customer Health:** Finding: median resolution is 50.5 h for critical and 52.9 h for low priority. Implication: urgent tickets are resolved only slightly sooner. Action: check triage rules against agreed service time limits.
5. **Customer Health (unified):** Finding: 8 flagged accounts hold $130,915, or 18.5% of won value. Implication: these valuable accounts have high support load and above-median negative sentiment; this does not predict churn. Action: assign account owners to review them.

Publish only after the annotations are in place. Export to `export/NovaTech_Dashboard.pdf`, open it and check that **all three sheets and every annotation** are readable. If QuickSight exports one sheet at a time, combine the three exports in order.

## Part C: Quick Chat and Topic

Order matters, because the report must describe the Topic's effect.

1. Run the verification log questions in Quick Chat against each knowledge base. Capture each answer with its question.
2. Do the cross-check in a QuickSight dataset preview and capture it.
3. **Before creating the Topic**, ask all five Q log questions and capture them (the reviewer asks for at least 2 to 3 before-and-after pairs). Record the exact answers as the baseline. If your environment does not allow a no-Topic baseline, record that instead.
4. Create a Topic on the prepared datasets. Capture its settings.
   - Friendly names and synonyms: `deal_value` = won value, revenue; `company_size_tier` = company size; `campaign_channel` = channel, campaign source; `resolution_hours` = resolution time; `tickets` = ticket volume
   - Describe `deal_value` as zero for lost deals, and `closed_won_flag` as marketing-stage conversion that is not matched to CRM deals
   - Set defaults: sum for spend and revenue; average for `resolution_hours`; do not sum IDs
   - Exclude row-level `campaign_roi` from Topic answers, or describe the weighted ROI formula
5. Ask the same five questions again. Record every answer, including wrong ones.
