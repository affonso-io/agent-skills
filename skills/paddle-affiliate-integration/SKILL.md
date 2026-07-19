---
name: paddle-affiliate-integration
description: Set up Affonso affiliate program tracking with Paddle integration. Use this skill when the user wants to implement or configure an affiliate program using Affonso with Paddle for payment processing. Handles tracking script installation, Google Tag Manager integration, GDPR/cookie consent compliance, and integration with Paddle Billing Checkout via Paddle.js customData. Triggers on requests like "set up affiliate program with Paddle", "integrate Affonso tracking with Paddle", "add affiliate tracking to Paddle checkout", or similar affiliate program implementation tasks.
---

# Paddle Affiliate Integration

This skill guides you through setting up Affonso affiliate tracking with Paddle payment integration.

## Overview

Affonso is an affiliate program platform that tracks referrals and commissions. This skill helps implement:

1. **Tracking script installation** - Monitor affiliate traffic and conversions
2. **Paddle integration** - Pass referral data to Paddle for commission tracking
3. **Optional features** - Signup tracking, GTM integration, GDPR compliance

## Prerequisites & Information Gathering

Before beginning implementation, gather the following information by asking the user:

### Required Information

1. **Program ID**: Ask "What is your Affonso program ID? You can find it at https://affonso.io/app/affiliate-program"

2. **Cookie Duration**: Ask "How many days should the affiliate tracking cookie persist? (Common values: 30, 60, 90 days)"

3. **Google Tag Manager**: Ask "Do you use Google Tag Manager to manage scripts on your website?"

4. **Cookie Consent Banner**: Ask "Do you use a cookie consent banner or need GDPR compliance?" If yes, follow up with "Which cookie consent platform do you use?" (e.g., Cookiebot, OneTrust, CookieYes, custom solution)

### Optional Features

5. **Signup Tracking**: Ask "Do you want to track user signups in addition to purchases? (Recommended - provides better insights on affiliate performance)"

### Connect Paddle

If the user's Paddle account is not already connected in Affonso, do this before implementing the code integration:

1. Go to https://affonso.io/app/affiliate-program/connect/paddle
2. Enter their Paddle API key
3. After connecting, they will receive a webhook URL
4. Add this webhook URL in Paddle's webhook settings

If Paddle is already connected, continue with the code changes and keep the webhook configuration active.

## Workflow Decision Tree

```
1. Connect Paddle to Affonso if it is not connected yet

2. Install Tracking Script
   ├─ Uses GTM? → See GTM Integration Guide
   ├─ Has Cookie Consent? → See GDPR Consent Guide
   └─ Neither → Direct script installation

3. Track Signups (Optional)
   └─ Add signup tracking code

4. Pass Data to Paddle
   └─ Paddle Billing Checkout → See Paddle Billing Checkout Guide

5. Test Integration
   └─ Verify tracking works end-to-end
```

## Step 1: Install Tracking Script

The tracking script monitors affiliate traffic and sets `window.affonso_referral` which gets passed to Paddle.

### Decision: GTM vs Direct Installation

**If user uses Google Tag Manager:**
- Read and follow the [GTM Integration Guide](references/gtm-integration.md)
- This includes instructions for creating the GTM tag, setting triggers, and publishing

**If user does NOT use GTM:**
- Continue with direct script installation below

### Direct Script Installation

#### Basic Installation (No Cookie Consent)

Add this script to the website's `<head>` tag:

```html
<!-- Place in <head> tag -->
<script
  async
  defer
  src="https://cdn.affonso.io/js/pixel.min.js"
  data-affonso="YOUR_PUBLIC_PROGRAM_ID"
  data-cookie_duration="YOUR_COOKIE_DURATION"
></script>
```

- Replace `YOUR_PUBLIC_PROGRAM_ID` with the user's program ID
- Replace `YOUR_COOKIE_DURATION` with the cookie duration (e.g., `30` for 30 days)

#### GDPR/Cookie Consent Installation

If the user has a cookie consent banner, read and follow the [GDPR Consent Guide](references/gdpr-consent.md) which includes:
- How to enable consent mode with `data-requires-consent="true"`
- Integration code for popular consent platforms (Cookiebot, OneTrust, CookieYes, etc.)
- URL parameter propagation requirements
- Testing consent-based tracking

**Consent-enabled script:**
```html
<script
  async
  defer
  src="https://cdn.affonso.io/js/pixel.min.js"
  data-affonso="YOUR_PUBLIC_PROGRAM_ID"
  data-cookie_duration="YOUR_COOKIE_DURATION"
  data-requires-consent="true"
></script>
```

Then add consent callback integration based on their consent platform (see GDPR guide).

## Step 2: Track User Signups (Optional)

If the user wants to track signups, add this code after their user registration logic:

```javascript
// After successful user registration
window.Affonso.signup(userEmail);
```

**Benefits of signup tracking:**
- See which affiliates drive the most registrations
- Calculate conversion rates from clicks to signups
- Optimize funnel based on affiliate performance

## Step 3: Pass Referral Data to Paddle

The tracking script sets `window.affonso_referral` which must be passed to Paddle through the checkout's `customData` parameter.

### Paddle Billing Checkout

Read [Paddle Billing Checkout Guide](references/paddle-billing-checkout.md) which includes:
- How to pass `affonso_referral` via `customData`
- How the webhook flow works
- Testing procedures

**Quick implementation:**
```javascript
Paddle.Checkout.open({
  product: 12345,
  email: "customer@example.com",
  customData: {
    affonso_referral: window.affonso_referral
  },
});
```

This works entirely client-side — no need to read cookies on the server.

## Step 4: Testing & Verification

### Test Tracking Script

1. Visit the website with test parameter: `yoursite.com?atp=test`
2. Open browser DevTools → Console
3. Type `window.affonso_referral` and verify it returns the test referral data
4. Check Affonso dashboard for test entry

### Test Paddle Integration

1. Visit the website with `?atp=test` parameter
2. Open browser DevTools → Console
3. Verify `window.affonso_referral` is set
4. Trigger a checkout and verify `customData` includes the referral ID in your server logs

### End-to-End Test

1. Visit site with `?atp=test` parameter
2. Complete a purchase using a discount code (for live testing without full payment)
3. Check Affonso dashboard to verify the conversion was tracked
4. Verify the test affiliate received credit for the sale

**Note:** Paddle sandbox mode transactions won't appear in Affonso - use discount codes on live mode or check server logs for verification.

### GDPR Consent Testing

If using consent mode:
1. Visit site with `?atp=test` - verify no cookie is set
2. Accept cookies through consent banner - verify cookie is now set
3. Navigate between pages - verify `affonso_id` parameter persists
4. Complete purchase - verify tracking works

## Troubleshooting

### Cookie Not Set
- Check that script is in `<head>` tag
- Verify program ID is correct
- If using consent mode, ensure `window.affonsoConsentGranted()` is called after user accepts

### Paddle Not Receiving Data
- Verify `window.affonso_referral` is set before opening checkout
- Check that `customData` is included in the `Paddle.Checkout.open()` call
- Inspect Network tab for script loading errors

### Tracking Not Appearing in Dashboard
- Ensure Paddle is connected in Affonso dashboard. If not, connect it first and verify the webhook URL in Paddle.
- For testing, use live mode with discount codes (sandbox mode doesn't sync)
- Check that the purchase completed successfully in Paddle
- Verify the webhook URL is correctly configured in Paddle

## Additional Resources

For framework-specific implementation guidance, the user can check:
- https://affonso.io/help/installation-guides (Next.js, React, Vue, WordPress, etc.)

## Next Steps

After successful integration:
1. Set up affiliate recruitment and program rules in Affonso dashboard
2. Create affiliate resources (landing pages, promotional materials)
3. Monitor performance and optimize commission structure
