---
name: affiliate-partner-activation
description: "Identify approved affiliate partners who have not launched or have become inactive, prioritize them by likely value, and prepare personalized activation plans and outreach. Use when the user asks to activate affiliates, re-engage dormant partners, improve time to first referral, or create partner follow-up sequences."
---

# Affiliate Partner Activation

Find the smallest set of actions most likely to turn eligible partners into productive advocates.

## Guardrails

- Verify access with `affonso whoami --json`; use `--json` on all CLI commands.
- Confirm the inactivity window and channel before drafting outreach. If none is given, use 30 days after approval for launch activation and 90 days without referral activity for reactivation, and label these defaults.
- Draft messages; do not send emails, create coupons, alter groups, or change terms without explicit approval.
- Do not promise commission rates, discounts, approval, or product capabilities that are not documented in the program configuration.

## Activation workflow

1. Read program context, available creative assets, and approved-partner list:

   ```bash
   affonso program get --json
   affonso program payment-terms get --json
   affonso program creatives list --json --limit 100 --page 1
   affonso affiliates list --json --status approved --limit 100 --page 1
   ```

2. For each candidate partner, collect referral and commission history over the relevant window:

   ```bash
   affonso referrals list --json --affiliate-id aff_example --created-gte YYYY-MM-DD --limit 100 --order desc
   affonso commissions list --json --affiliate-id aff_example --date-from YYYY-MM-DD --limit 100 --page 1
   ```

3. Segment candidates into `new and not launched`, `previously active and dormant`, `seasonal`, or `tracking/onboarding blocked`. Prefer evidence over a blanket message.
4. Prioritize by audience fit, historical output, recency, available campaign relevance, and ease of removing the stated blocker. Do not infer audience size or intent from missing data.
5. Build one action per partner: a specific resource, product angle, campaign, tracking check, or short call request. Draft a concise message with a clear call to action and one follow-up date.
6. Present the plan for approval. After approval, create only the confirmed operational items, such as a coupon or group assignment, and never send outreach through an external system unless separately authorized.

## Required output

Return a ranked table with partner ID/name, segment, evidence, activation hypothesis, recommended action, confidence, and follow-up date. Include reusable but personalized outreach drafts, each with subject line, message, and CTA. List any missing asset, offer, or tracking dependency blocking activation.

For CLI details, load `../../affonso-cli/SKILL.md` and its command reference when needed.
