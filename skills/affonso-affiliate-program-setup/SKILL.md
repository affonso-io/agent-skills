---
name: affonso-setup
description: "Automatically detect the user's payment provider and integration method, then set up Affonso affiliate tracking end-to-end. Use this skill when the user wants to add affiliate tracking, set up Affonso, integrate a referral program, or says 'set up affiliate program' without specifying a provider. This is the main entry point — it detects everything and handles the full setup."
---

# Affonso Setup

This skill automatically detects which payment provider and integration method the user's project uses, then performs the full Affonso affiliate tracking setup.

## Step 1: Detect Payment Provider

Scan the project to identify which payment provider is in use. Search for these indicators:

### Package Detection
Search `package.json`, `requirements.txt`, `Gemfile`, `composer.json`, or `go.mod` for:

| Package | Provider |
|---------|----------|
| `stripe` / `@stripe/stripe-js` / `stripe-node` | Stripe |
| `@polar-sh/sdk` / `polar` | Polar |
| `dodopayments` | Dodo Payments |
| `@creem/sdk` / `creem` | Creem |
| `@paddle/paddle-js` / `@paddle/paddle-node-sdk` | Paddle |

### Code Detection
If no package match, search source files for:

| Pattern | Provider |
|---------|----------|
| `stripe.checkout.sessions.create` / `Stripe(` / `stripe.com` | Stripe |
| `polar.checkouts.create` / `buy.polar.sh` | Polar |
| `client.payments.create` + `dodopayments` / `checkout.dodopayments.com` | Dodo Payments |
| `api.creem.io` / `creem.io/payment` | Creem |
| `Paddle.Checkout.open` / `paddle.com` / `Paddle.Initialize` | Paddle |

### If No Provider Detected
Ask the user: "I couldn't detect your payment provider. Which one do you use?"
- Stripe
- Polar
- Paddle
- Creem
- Dodo Payments
- Other (will use the custom integration via API)

## Step 2: Detect Integration Method

Based on the detected provider, determine the integration method:

### Stripe
Search for:
- `stripe.checkout.sessions.create` → **Checkout API**
- `buy.stripe.com` in HTML/JSX → **Payment Links**
- `<stripe-buy-button` → **Buy Button**
- `<stripe-pricing-table` → **Pricing Tables**

### Polar
Search for:
- `polar.checkouts.create` → **Checkout API**
- `buy.polar.sh` in HTML/JSX → **Payment Links**

### Dodo Payments
Search for:
- `client.payments.create` / `client.subscriptions.create` with `dodopayments` → **Checkout API**
- `checkout.dodopayments.com` in HTML/JSX → **Payment Links**

### Creem
Search for:
- `api.creem.io/v1/checkouts` → **Checkout API**
- `creem.io/payment` in HTML/JSX → **Payment Links**

### Paddle
- Always **Billing Checkout** (only method available)

### If Multiple Methods Detected
List what was found and ask the user which one they want to integrate with.

### If No Method Detected
Ask the user how they handle payments (Checkout API, Payment Links, etc.)

## Step 3: Check Existing Setup

Before making changes, check if Affonso is already partially set up:

1. **Tracking script**: Search HTML/layouts for `cdn.affonso.io/js/pixel.min.js` or `data-affonso`
2. **Cookie reading**: Search for `affonso_referral` in server-side code
3. **Enhancement scripts**: Search for `enhanceStripeLinks`, `enhancePolarLinks`, etc.

Report what's already in place and what still needs to be done.

## Step 4: Connect Payment Provider

If not already connected, tell the user:

> Before I can set up the integration, you need to connect your payment provider in the Affonso dashboard:
>
> 1. Go to https://affonso.io/app/affiliate-program/connect
> 2. Select **{detected provider}**
> 3. Enter your **{provider} API key**
> 4. Copy the **webhook URL** that Affonso gives you
> 5. Add the webhook URL in your **{provider} dashboard**
>
> Let me know once that's done and I'll continue with the code integration.

## Step 5: Gather Required Information

Ask the user for:

1. **Program ID**: "What is your Affonso program ID? Find it at https://affonso.io/app/affiliate-program"
2. **Cookie Duration**: "How many days should the affiliate tracking cookie persist? (Common: 30, 60, 90 days)"
3. **Google Tag Manager**: "Do you use Google Tag Manager?"
4. **Cookie Consent**: "Do you use a cookie consent banner? If yes, which one?"
5. **Signup Tracking**: "Do you want to track user signups too? (Recommended)"

## Step 6: Implement Integration

Based on the detected provider and method, follow the corresponding skill's instructions:

| Provider | Skill to Follow |
|----------|----------------|
| Stripe | [stripe-affiliate-integration](../stripe-affiliate-integration/SKILL.md) |
| Polar | [polar-affiliate-integration](../polar-affiliate-integration/SKILL.md) |
| Dodo Payments | [dodo-affiliate-integration](../dodo-affiliate-integration/SKILL.md) |
| Creem | [creem-affiliate-integration](../creem-affiliate-integration/SKILL.md) |
| Paddle | [paddle-affiliate-integration](../paddle-affiliate-integration/SKILL.md) |
| Other | [custom-affiliate-integration](../custom-affiliate-integration/SKILL.md) |

Read the appropriate skill and its reference files, then implement all steps:

1. Install the Affonso tracking script
2. Add signup tracking (if requested)
3. Pass referral data to the payment provider
4. Set up GTM (if applicable)
5. Set up GDPR consent (if applicable)

## Step 7: Verify Setup

After implementation, guide the user through testing:

1. "Visit your site with `?atp=test` in the URL"
2. "Open DevTools → Application → Cookies → check for `affonso_referral`"
3. Verify the provider-specific integration (referral data passed correctly)
4. "Check your Affonso dashboard for the test entry"

## Summary Output

After completing the setup, provide a summary:

```
Affonso Affiliate Tracking Setup Complete

Provider: {provider}
Method: {method}
Tracking Script: Installed
Signup Tracking: Yes/No
GTM Integration: Yes/No/N/A
GDPR Consent: Yes/No/N/A
Referral Passing: Configured

Next steps:
1. Test with ?atp=test parameter
2. Set up affiliate recruitment in Affonso dashboard
3. Consider embedding the affiliate dashboard: /embedded-affiliate-dashboard
```
