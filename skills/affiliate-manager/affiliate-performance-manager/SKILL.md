---
name: affiliate-performance-manager
description: "Analyze affiliate program and partner performance in Affonso, identify trends, risks, and growth opportunities, and prioritize concrete next actions. Use for weekly or monthly affiliate reporting, partner portfolio reviews, conversion or commission analysis, unexplained changes, and account-health management."
---

# Affiliate Performance Manager

Convert program data into a short, action-oriented operating review.

## Working rules

- Verify access with `affonso whoami --json`; use `--json` for every CLI request and page through the entire requested scope.
- Confirm the reporting period, currency handling, and success metric. If absent, declare a sensible window, such as the last 30 days compared with the preceding 30 days.
- Distinguish observed metrics from causes. Label causal explanations as hypotheses until supported by campaign, tracking, or partner evidence.
- Do not alter commissions, groups, payment terms, or partner status during analysis without explicit approval.

## Analysis workflow

1. Establish context and pull the program configuration:

   ```bash
   affonso program get --json
   affonso program payment-terms get --json
   ```

2. Collect scoped program and partner data. Start broad, then drill into material changes:

   ```bash
   affonso affiliates list --json --status approved --limit 100 --page 1
   affonso commissions list --json --date-from YYYY-MM-DD --date-to YYYY-MM-DD --limit 100 --page 1
   affonso referrals list --json --created-gte YYYY-MM-DD --created-lte YYYY-MM-DD --limit 100 --order desc
   affonso payouts list --json --date-from YYYY-MM-DD --date-to YYYY-MM-DD --limit 100 --page 1
   ```

3. Measure only metrics supported by the returned data. Typical views are approved partners, active partners, referred customers, commission value, average commission per referral, pending versus approved commission mix, and payout exposure. Calculate conversion rate or EPC only when both valid numerator and denominator data are available.
4. Compare current results with the prior equivalent period and with each partner's own baseline. Identify the few drivers that explain most movement: emerging partners, declining partners, concentration risk, pending-value backlog, and suspicious changes requiring the fraud review skill.
5. Turn each finding into one owner, action, expected effect, and check-in date. Prioritize by expected value, confidence, and effort.

When a recommendation concerns contribution margin, refunds, LTV, CAC, payback, or commission rate, route the decision to `$affiliate-profitability-manager` instead of estimating economics from Affonso data alone.

## Required output

Return:

1. An executive summary with three to five material changes.
2. A partner table: partner ID/name, current-period output, trend, opportunity or risk, and next action.
3. A prioritized action list split into `grow`, `retain`, `fix`, and `investigate`.
4. Data limitations and any metric definitions used.

Route suspicious payouts or conversion patterns to `$affiliate-fraud-review`. Route inactive but promising partners to `$affiliate-partner-activation`.

For CLI details, load `../../affonso-cli/SKILL.md` and its command reference when needed.
