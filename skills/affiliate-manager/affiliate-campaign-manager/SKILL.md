---
name: affiliate-campaign-manager
description: "Plan, prepare, operate, and evaluate affiliate marketing campaigns in Affonso. Use when the user wants to launch a seasonal promotion, product launch, coupon campaign, partner challenge, campaign briefing, partner selection, follow-up plan, or post-campaign review."
---

# Affiliate Campaign Manager

Manage a campaign from a measurable brief through partner execution and retrospective.

## Guardrails

- Verify access with `affonso whoami --json` and use `--json` for every CLI command.
- Confirm the objective, audience, dates, offer, eligible channels, budget or incentive cap, and success metric. Mark unknown items as assumptions for approval.
- Do not create coupons or creatives, change program terms, or make group assignments until the user explicitly approves the exact assets, eligibility, and commercial terms.
- Do not claim results caused by the campaign unless a comparison method and attribution data support it.
- Model contribution margin, refund exposure, and commission headroom with `$affiliate-profitability-manager` before proposing an incentive or scale decision.

## Campaign workflow

1. Inspect program constraints and existing assets:

   ```bash
   affonso program get --json
   affonso program payment-terms get --json
   affonso program restrictions get --json
   affonso program groups list --json
   affonso program creatives list --json --limit 100 --page 1
   affonso coupons list --json --limit 100 --page 1
   ```

2. Define a one-page brief: objective, customer proposition, partner proposition, audience, offer guardrails, launch/end times with timezone, tracking convention, owner, and success metric.
3. Select partners using relevant recent performance, audience/channel fit, current activity, and policy eligibility. Include a holdout or baseline when practical.
4. Prepare the execution kit: partner segments, personalized brief, approved messaging angles, links or UTM convention, assets, disclosure reminders, and follow-up schedule.
5. Present the campaign plan and required mutations. With explicit approval, create campaign assets, for example:

   ```bash
   affonso coupons create --json --affiliate-id aff_example --code LAUNCH20 --discount-type percentage --discount-value 20 --duration once
   affonso program creatives create --json --name "Launch banner" --type banner --file-url https://example.com/banner.png --width 728 --height 90
   ```

6. During and after the campaign, collect referrals and commissions for the campaign dates; compare results with the stated baseline and report outcomes, limitations, and next iteration.

## Required output

Return the brief, an eligible-partner shortlist with rationale, a timeline, launch checklist, partner message draft, measurement plan, and a list of changes awaiting approval. For a completed campaign, add results, comparison period, learnings, and next experiment. Include contribution profit only when validated financial data is available.

For CLI details, load `../../affonso-cli/SKILL.md` and its command reference when needed.
