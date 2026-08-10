---
name: affiliate-fraud-review
description: "Review affiliate conversions, commissions, and pending payouts for fraud risk in an Affonso program. Use when the user asks to screen open payouts, investigate suspicious referrals, assess self-referrals, conversion spikes, duplicate IP or VPN signals, coupon abuse, or decide which cases need manual investigation. Produce evidence-based recommendations; never change payout or commission status without explicit user approval."
---

# Affiliate Fraud Review

Review a defined period or payout batch and return an auditable triage, not an automatic fraud verdict.

## Guardrails

- Use `affonso ... --json` for every CLI call. Verify access with `affonso whoami --json` before inspecting program data.
- Start read-only. Do not approve, cancel, process, or complete a payout; do not change commission status or fraud rules unless the user explicitly approves the exact change.
- Treat a signal as evidence to investigate, not proof of fraud. Avoid conclusions based on protected characteristics, location alone, or a single weak signal.
- Keep customer and affiliate data in the report to the minimum needed. Prefer IDs and aggregated evidence over personal data.
- State data gaps plainly. The CLI may not expose all click, device, payment, refund, or KYC evidence needed for a final decision.

## Review workflow

1. Confirm scope: payout IDs, affiliate IDs, or a date range; use the current open payout cycle when no scope is supplied.
2. Read the policy context:

   ```bash
   affonso program payment-terms get --json
   affonso program fraud-rules get --json
   affonso program restrictions get --json
   ```

3. Fetch the batch and investigate each material or flagged case. Use pagination until the selected scope is complete.

   ```bash
   affonso payouts list --json --status pending --limit 50 --page 1
   affonso payouts get pay_example --json
   affonso affiliates get aff_example --json
   affonso commissions list --json --affiliate-id aff_example --limit 100 --page 1
   affonso referrals list --json --affiliate-id aff_example --limit 100 --order desc
   ```

4. Compare each case with its own historical baseline and the program cohort. Check for a sudden concentration of sales, abnormal conversion timing, repeated customer or attribution identifiers where available, unusual coupon usage, recently created partners with large balances, and conflicts with the configured fraud rules.
5. Separate observed facts, reasonable inferences, and missing evidence. Do not infer fraud from high performance alone.
6. Assign one recommendation: `clear`, `pay with monitoring`, `hold for investigation`, or `escalate`. Explain the minimum next evidence required for every hold or escalation.
7. Ask for confirmation before performing any status change. If approved, state the target IDs and intended new status immediately before executing it.

## Signal interpretation

Use combinations of signals and record the comparison window. Useful signals include:

| Signal | Check | Typical follow-up |
| --- | --- | --- |
| Self-referral | Shared customer, affiliate, payment, or device evidence when available | Request identity/payment review; apply configured policy |
| Spike | Change in sales, conversion rate, or payout relative to baseline | Inspect source, campaign, and individual referrals |
| Low-quality conversions | Rapid cancellations, reversals, refunds, or short retention | Wait for the hold period or request billing evidence |
| Attribution anomaly | Repeated identifiers, unexpected click-to-sale timing, missing attribution fields | Review click and server-side tracking logs |
| Coupon misuse | Coupon appears on unapproved sources or is redeemed outside campaign intent | Check terms, source evidence, and coupon distribution |

Prioritize cases where payout value, signal strength, and potential irreversibility are all high. Do not add unsupported thresholds; use program policy or clearly label a proposed threshold.

## Required report

Return a compact table with payout/affiliate ID, amount, observed signals, counter-evidence, confidence (`low`, `medium`, `high`), recommendation, and next step. Follow it with:

- batch totals by recommendation;
- the exact policy rules consulted;
- evidence unavailable from Affonso that would change the decision; and
- an explicit list of proposed mutations, if any, awaiting approval.

For CLI details, load `../../affonso-cli/SKILL.md` and its command reference when needed.
