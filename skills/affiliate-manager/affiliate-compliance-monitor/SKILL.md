---
name: affiliate-compliance-monitor
description: "Monitor and investigate affiliate compliance with program traffic restrictions, brand rules, disclosure requirements, coupon policies, and promotion terms in Affonso. Use when the user asks to review partner sites or evidence, audit a campaign, investigate trademark bidding, coupon leakage, prohibited traffic, misleading claims, or decide whether a partner needs remediation."
---

# Affiliate Compliance Monitor

Turn policy and observed evidence into a fair, documented remediation recommendation.

## Guardrails

- Verify access with `affonso whoami --json`; use `--json` for every CLI command.
- Treat program restrictions and supplied terms as the source of truth. Request the applicable policy when it is absent; do not invent legal, platform, or brand requirements.
- Capture the exact evidence, timestamp, URL or source, and policy clause. Clearly separate a violation, a risk, and an unverified allegation.
- Do not suspend or reject a partner, cancel payouts, change program restrictions, or contact a partner without explicit approval. Escalate legal questions to the user or counsel rather than providing legal conclusions.

## Compliance workflow

1. Read the current rules and commercial terms:

   ```bash
   affonso program get --json
   affonso program restrictions get --json
   affonso program fraud-rules get --json
   affonso program payment-terms get --json
   ```

2. Confirm the audit scope: partner IDs, campaign, channels, period, policy version, and evidence source. Request screenshots, URLs, ad-library exports, coupon listings, or tracking data when needed.
3. Retrieve the relevant partner and transactional context:

   ```bash
   affonso affiliates get aff_example --json
   affonso coupons list --json --affiliate-id aff_example --limit 100 --page 1
   affonso referrals list --json --affiliate-id aff_example --created-gte YYYY-MM-DD --limit 100 --order desc
   ```

4. Map evidence to one or more categories: unauthorized channel, trademark bidding, missing disclosure, misleading or prohibited claim, coupon leakage/misuse, incentivized traffic, or tracking manipulation. Preserve ambiguity when evidence is inconclusive.
5. Assign a proportionate recommendation: `no issue`, `monitor`, `request correction`, `pause campaign participation`, or `escalate`. State remediation terms, an evidence deadline, and the owner.
6. Draft a factual partner notice. Ask for approval before any external contact or account change. Re-check the same evidence after remediation before closing the case.

## Required output

Return an evidence table with partner ID, policy rule, source/timestamp, finding, confidence, recommendation, remediation deadline, and follow-up owner. Include a neutral partner-notice draft and a separate list of actions that require explicit approval.

Route suspicious conversions or payout concerns to `$affiliate-fraud-review`. For CLI details, load `../../affonso-cli/SKILL.md` and its command reference when needed.
