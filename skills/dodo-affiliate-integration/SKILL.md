---
name: dodo-affiliate-integration
description: Set up Affonso affiliate program tracking with Dodo Payments integration. Use this skill when the user wants to implement or configure an affiliate program using Affonso with Dodo Payments for payment processing. Handles tracking script installation, Google Tag Manager integration, GDPR/cookie consent compliance, and integration with Dodo Checkout API or Payment Links. Triggers on requests like "set up affiliate program with Dodo Payments", "integrate Affonso tracking with Dodo", "add affiliate tracking to Dodo checkout", or similar affiliate program implementation tasks.
---

# Dodo Payments Affiliate Integration

This skill guides you through setting up Affonso affiliate tracking with Dodo Payments integration.

## Overview

Affonso is an affiliate program platform that tracks referrals and commissions. This skill helps implement:

1. **Tracking script installation** - Monitor affiliate traffic and conversions
2. **Dodo Payments integration** - Pass referral data to Dodo for commission tracking
3. **Optional features** - Signup tracking, GTM integration, GDPR compliance

## Prerequisites & Information Gathering

Before beginning implementation, gather the following information by asking the user:

### Required Information

1. **Program ID**: Ask "What is your Affonso program ID? You can find it at https://affonso.io/app/affiliate-program"

2. **Cookie Duration**: Ask "How many days should the affiliate tracking cookie persist? (Common values: 30, 60, 90 days)"

3. **Dodo Integration Method**: Ask "How do you communicate with Dodo Payments?" Provide options:
   - Dodo Checkout API
   - Dodo Payment Links

4. **Google Tag Manager**: Ask "Do you use Google Tag Manager to manage scripts on your website?"

5. **Cookie Consent Banner**: Ask "Do you use a cookie consent banner or need GDPR compliance?" If yes, follow up with "Which cookie consent platform do you use?" (e.g., Cookiebot, OneTrust, CookieYes, custom solution)

### Optional Features

6. **Signup Tracking**: Ask "Do you want to track user signups in addition to purchases? (Recommended - provides better insights on affiliate performance)"

### Connect Payment Provider

Before implementing the integration, the user must connect Dodo Payments to Affonso:

1. Go to https://affonso.io/app/affiliate-program/connect
2. Enter their Dodo Payments API key
3. After connecting, they will receive a webhook URL
4. Add the webhook URL in their Dodo Payments dashboard

## Workflow Decision Tree

```
1. Install Tracking Script
   ├─ Uses GTM? → See GTM Integration Guide
   ├─ Has Cookie Consent? → See GDPR Consent Guide
   └─ Neither → Direct script installation

2. Track Signups (Optional)
   └─ Add signup tracking code

3. Pass Data to Dodo Payments
   ├─ Checkout API → See Dodo Checkout API Guide
   ├─ Payment Links → See Dodo Payment Links Guide
   └─ Don't Know → Help identify integration method

4. Test Integration
   └─ Verify tracking works end-to-end
```

## Step 1: Install Tracking Script

The tracking script monitors affiliate traffic and creates the `affonso_referral` cookie that gets passed to Dodo Payments.

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

## Step 3: Pass Referral Data to Dodo Payments

The tracking script creates an `affonso_referral` cookie that must be passed to Dodo Payments. Implementation varies by Dodo integration method.

### Identify Dodo Integration Method

If the user is unsure, help them identify their integration method:

- **Checkout API**: They create Dodo payment or subscription sessions programmatically in backend code (Node.js, etc.)
- **Payment Links**: They use links containing `checkout.dodopayments.com` that redirect to Dodo's hosted checkout

### Integration Guides

Based on the identified method, read and implement the appropriate guide:

#### Dodo Checkout API
**When to use**: User creates payments or subscriptions programmatically in backend code

Read [Dodo Checkout API Guide](references/dodo-checkout-api.md) which includes:
- How to pass `affonso_referral` cookie as metadata
- Examples for payments and subscriptions
- Testing with server logs

**Quick implementation** (Next.js example):
```javascript
import { cookies } from 'next/headers';
import DodoPayments from 'dodopayments';

const cookieStore = await cookies();
const affonsoReferral = cookieStore.get('affonso_referral')?.value || '';

const client = new DodoPayments({
  bearerToken: process.env.DODO_PAYMENTS_API_KEY,
});

// For payments
const payment = await client.payments.create({
  metadata: { affonso_referral: affonsoReferral },
});

// For subscriptions
const subscription = await client.subscriptions.create({
  metadata: { affonso_referral: affonsoReferral },
});
```

#### Dodo Payment Links
**When to use**: User has links containing `checkout.dodopayments.com`

Read [Payment Links Guide](references/dodo-payment-links.md) which includes:
- Enhancement script that automatically appends `metadata_affonso_referral` to links
- How the script works and when it runs
- Testing procedures

## Step 4: Testing & Verification

### Test Tracking Script

1. Visit the website with test parameter: `yoursite.com?atp=test`
2. Open browser DevTools → Application → Cookies
3. Verify `affonso_referral` cookie is set (unless consent mode is enabled and consent not given)
4. Check Affonso dashboard for test entry

### Test Dodo Integration

**For Checkout API:**
- Trigger checkout flow and check server logs for `metadata: { affonso_referral: "..." }`

**For Payment Links:**
- Inspect the enhanced URLs to verify `metadata_affonso_referral` is added
- DevTools → Elements tab → Find the Dodo link and check the URL

### End-to-End Test

1. Visit site with `?atp=test` parameter
2. Complete a purchase using a 100% discount code (for live testing without payment)
3. Check Affonso dashboard to verify the conversion was tracked
4. Verify the test affiliate received credit for the sale

**Note:** Dodo Payments test mode transactions won't appear in Affonso - use discount codes or check server logs/element attributes.

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

### Dodo Not Receiving Data
- **Checkout API**: Verify server code reads cookie and passes to metadata
- **Payment Links**: Check that enhancement script runs (look for `affonso_referral_ready` event)
- Inspect Network tab for script loading errors

### Tracking Not Appearing in Dashboard
- Ensure Dodo Payments is connected in Affonso dashboard at https://affonso.io/app/affiliate-program/connect
- Verify the webhook URL is added in Dodo Payments
- For testing, use discount codes (test mode doesn't sync)
- Check that the purchase completed successfully in Dodo Payments

## Additional Resources

For framework-specific implementation guidance, the user can check:
- https://affonso.io/help/installation-guides (Next.js, React, Vue, WordPress, etc.)

## Next Steps

After successful integration:
1. Set up affiliate recruitment and program rules in Affonso dashboard
2. Create affiliate resources (landing pages, promotional materials)
3. Monitor performance and optimize commission structure
