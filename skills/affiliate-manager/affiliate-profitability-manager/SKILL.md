---
name: affiliate-profitability-manager
description: "Evaluate affiliate, campaign, and commission decisions against contribution margin, refunds, acquisition cost, LTV, and payback. Use when the user wants to assess affiliate profitability, set or negotiate commission rates, compare partner economics, model incentives, calculate margin headroom, or decide whether a campaign can scale. Require supplied financial data and label projections, assumptions, and data gaps; never invent costs or change commercial terms without explicit approval."
---

# Affiliate Profitability Manager

Recommend affiliate actions that grow contribution profit, not merely attributed revenue.

## Guardrails

- Read [the profitability data contract](references/profitability-data-contract.md) before requesting, mapping, or calculating financial data.
- Use source-of-truth finance, billing, and Affonso data. Keep the source, currency, period, revenue-recognition basis, and calculation grain visible in every result.
- Never assume LTV, margin, refund rate, payment costs, discounts, or the incrementality of attributed customers. Mark unavailable values as unknown and return the decision that can be supported without them.
- Separate realized results from forecasts. Use the same cohort, customer definition, currency, and time horizon when comparing partners or periods.
- Do not create or change commission terms, bonuses, coupons, groups, or payouts without explicit approval of the exact proposal.

## Workflow

1. Confirm the decision: evaluate existing partners, set a new commission, approve a bonus, select a campaign, or scale/stop a channel. Confirm the target contribution margin, payback period, and LTV horizon; if absent, request them rather than choosing business thresholds.
2. Gather Affonso attribution and commission data for the exact period and partner scope:

   ```bash
   affonso program payment-terms get --json
   affonso affiliates list --json --status approved --limit 100 --page 1
   affonso referrals list --json --created-gte YYYY-MM-DD --created-lte YYYY-MM-DD --limit 100 --order desc
   affonso commissions list --json --date-from YYYY-MM-DD --date-to YYYY-MM-DD --limit 100 --page 1
   ```

3. Request or ingest the matching financial dataset. Start with the minimum viable inputs in the data contract; add cohort revenue, retention, and direct-partner costs for LTV/CAC or commission-headroom decisions.
4. Reconcile data before calculating: join key, date timezone, currency conversion, refunded/chargeback treatment, commission status, customer eligibility, and possible duplicate rows. Stop and explain the discrepancy if totals cannot be reconciled.
5. Calculate only metrics supported by the inputs. At a minimum, show net revenue after refunds, variable cost, affiliate cost, and contribution after affiliate cost. Use `attributed acquisition cost`, not CAC, unless the customer is verified as new and the required cost allocation is available.
6. Compare actual result with the stated target and a relevant partner/cohort baseline. For projected LTV, state the model, horizon, cohort maturity, and sensitivity to refund/retention assumptions.
7. Recommend `scale`, `maintain`, `test with cap`, `renegotiate`, `pause expansion`, or `insufficient data`, with the expected financial effect and the next validation point. Present any commercial change as a proposal awaiting approval.

## Decision outputs

Return:

1. A decision summary containing scope, recommendation, confidence, and material caveats.
2. A partner or campaign table with net revenue, refunds/chargebacks, variable cost, affiliate cost, contribution after affiliate cost, contribution margin, and the target comparison.
3. LTV, acquisition-cost, payback, or commission-headroom analysis only when the data contract supports it; otherwise list the missing inputs.
4. A sensitivity table for material assumptions such as refund rate, retention, or commission rate.
5. Exact proposed terms, affected partners, expected cost, stop condition, and approval required for every suggested change.

Use `$affiliate-performance-manager` for broader operational prioritization and `$affiliate-campaign-manager` for campaign planning. For CLI details, load `../../affonso-cli/SKILL.md` and its command reference when needed.
