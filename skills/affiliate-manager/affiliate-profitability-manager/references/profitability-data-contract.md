# Profitability Data Contract

Use this reference when requesting or mapping financial data. Use the smallest grain that supports the decision: one row per referred customer is best; affiliate-by-cohort is acceptable for portfolio decisions; affiliate-by-period is insufficient for new-customer CAC or LTV analysis.

## Required context

Provide the reporting timezone, currency, date range, revenue-recognition basis (cash collected or recognized revenue), customer key, affiliate key, and data sources. Do not mix currencies or revenue-recognition bases without an explicit conversion and reconciliation.

## Minimum viable inputs

| Field | Meaning | Source |
| --- | --- | --- |
| `affiliate_id` | Affonso affiliate identifier | Affonso |
| `customer_id` or cohort key | Privacy-safe stable join key | Billing/warehouse |
| `period_start`, `period_end` | Reported period or cohort window | All sources |
| `gross_revenue` | Revenue before discounts, tax, refunds, and chargebacks | Billing |
| `discount_amount`, `tax_amount` | Reductions excluded from net collected revenue | Billing |
| `refund_amount`, `chargeback_amount` | Reversals recognized in the selected period | Billing |
| `cogs`, `payment_processing_fees`, `other_variable_cost` | Incremental cost to serve | Finance/warehouse |
| `affiliate_commission_accrued` | Attributed commission, with status | Affonso/billing |

Compute `net_collected_revenue = gross_revenue - discount_amount - tax_amount - refund_amount - chargeback_amount`. Do not subtract a cost twice if the accounting source already supplies net revenue or net margin.

## Inputs for stronger decisions

| Decision | Additional inputs |
| --- | --- |
| Partner contribution | `affiliate_bonus`, placement/creative costs, affiliate-platform allocation, incremental support cost |
| New-customer acquisition cost | New-customer flag, first paid date, direct partner costs, allocation rule for shared program costs |
| LTV/CAC and payback | Monthly cohort revenue, retention/churn, future COGS and service costs, LTV horizon, realized vs forecast marker |
| Commission headroom | Target contribution margin or target contribution per customer, existing affiliate cost, expected future contribution |
| Incrementality | Holdout, lift test, reliable channel-exposure/control data, or an explicitly approved causal model |

## Definitions and formulas

Use these consistently and label the horizon and scope.

| Metric | Formula |
| --- | --- |
| Affiliate cost | Commission + bonus + direct partner costs + allocated affiliate-platform cost |
| Contribution before affiliate cost | Net collected revenue − COGS − payment fees − other variable cost |
| Contribution after affiliate cost | Contribution before affiliate cost − affiliate cost |
| Contribution margin | Contribution after affiliate cost ÷ net collected revenue |
| Attributed acquisition cost | Affiliate cost for verified new customers ÷ number of verified new customers |
| Contribution LTV | Cumulative or forecast contribution before affiliate cost over the declared horizon |
| LTV:CAC | Contribution LTV ÷ attributed acquisition cost; call this LTV:CAC only if the acquisition cost has a justified allocation |
| Payback | Time until cumulative contribution after affiliate cost is non-negative; do not calculate from a single period without a cohort curve |
| Maximum incremental commission | Contribution before affiliate cost − target contribution after affiliate cost − existing affiliate cost |

Do not call attributed customers incremental. Attribution answers who received credit; incrementality requires a credible counterfactual.

## Validation checklist

- Reconcile partner-level totals to billing/finance totals for the same period.
- Use the same refund/chargeback recognition rule in every comparison.
- Keep accrued, approved, and paid commission statuses separate.
- Deduplicate customer, subscription, invoice, and commission records before aggregation.
- Convert currencies at a declared rate and date; do not average percentages across currencies or cohorts without weighting.
- Flag immature cohorts and forecasts. Show a sensitivity range for assumptions that materially affect the decision.
- Minimize customer data; use hashed or internal IDs and do not include unnecessary personal information in reports.
