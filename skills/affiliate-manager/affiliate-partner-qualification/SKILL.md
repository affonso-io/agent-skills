---
name: affiliate-partner-qualification
description: "Assess pending affiliate applications for brand fit, audience relevance, program-policy fit, and fraud or compliance risk in Affonso. Use when the user asks to review, prioritize, approve, reject, or request more information from affiliate applicants. Return a reasoned recommendation and do not change applicant status without explicit approval."
---

# Affiliate Partner Qualification

Turn a pending applicant queue into consistent, explainable recommendations.

## Guardrails

- Verify access with `affonso whoami --json`; use `--json` on every command.
- Review only information relevant to commercial fit, policy compliance, and program safety. Do not use protected traits or make speculative personal judgments.
- Do not approve, reject, alter groups, or create coupons without explicit user approval for the target applicants.
- If an application lacks evidence, recommend `request information` rather than guessing. Treat a supplied website or social profile as evidence, not a guarantee of traffic quality.

## Qualification workflow

1. Read the program's positioning and allowed traffic before evaluating applicants:

   ```bash
   affonso program get --json
   affonso program restrictions get --json
   affonso program fraud-rules get --json
   affonso program groups list --json
   ```

2. Fetch all selected pending applicants. Page through the queue when needed.

   ```bash
   affonso affiliates list --json --status pending --limit 50 --page 1
   affonso affiliates get aff_example --json
   ```

3. Evaluate each applicant against four dimensions:

   | Dimension | Look for |
   | --- | --- |
   | Audience fit | Relevant audience, market, language, and product use case |
   | Channel fit | Proposed channels permitted by program restrictions |
   | Quality signals | Clear identity or business context, credible owned channels, realistic promotion plan |
   | Risk signals | Missing context, policy conflict, duplicated or inconsistent details, incentive-led traffic where prohibited |

4. Recommend one outcome: `approve`, `approve with conditions`, `request information`, or `reject`. Use `approve with conditions` only when the operational condition can be stated and enforced, such as a restricted traffic channel or a trial group.
5. For approved candidates, suggest the appropriate group, onboarding step, and one activation action. For rejected candidates, draft a brief, neutral reason that refers to policy rather than personal attributes.
6. Present recommendations first. After explicit approval, make only the confirmed updates, for example:

   ```bash
   affonso affiliates update aff_example --json --status approved --group-id grp_example
   ```

## Required output

Return a queue table containing affiliate ID, declared channel, fit summary, risks or missing information, recommendation, confidence, and next action. State the applicable traffic restrictions and distinguish application facts from assumptions. Include concise approval/rejection or information-request drafts when useful.

For CLI details, load `../../affonso-cli/SKILL.md` and its command reference when needed.
