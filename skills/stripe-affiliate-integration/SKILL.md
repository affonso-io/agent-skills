---
name: stripe-affiliate-integration
description: Set up Affonso affiliate program tracking with Stripe integration. Use this skill when the user wants to implement or configure an affiliate program using Affonso with Stripe for payment processing. Handles tracking script installation, Google Tag Manager integration, GDPR/cookie consent compliance, and integration with Stripe Checkout API, Payment Links, Buy Buttons, or Pricing Tables. Triggers on requests like "set up affiliate program with Stripe", "integrate Affonso tracking", "add affiliate tracking to Stripe checkout", or similar affiliate program implementation tasks.
---

# Stripe Affiliate Integration

This skill guides you through setting up Affonso affiliate tracking with Stripe payment integration.

## Overview

Affonso is an affiliate program platform that tracks referrals and commissions. This skill helps implement:

1. **Tracking script installation** - Monitor affiliate traffic and conversions
2. **Stripe integration** - Pass referral data to Stripe for commission tracking
3. **Optional features** - Signup tracking, GTM integration, GDPR compliance

## Prerequisites & Information Gathering

Before beginning implementation, gather the following information by asking the user:

### Required Information

1. **Program ID**: Ask "What is your Affonso program ID? You can find it at https://affonso.io/app/affiliate-program"

2. **Cookie Duration**: Ask "How many days should the affiliate tracking cookie persist? (Common values: 30, 60, 90 days)"

3. **Stripe Integration Method**: Ask "How do you communicate with Stripe?" Provide options:
   - Stripe Checkout API
   - Stripe Payment Links
   - Stripe Buy Button
   - Stripe Pricing Tables
   - I don't know / Need help determining this

4. **Google Tag Manager**: Ask "Do you use Google Tag Manager to manage scripts on your website?"

5. **Cookie Consent Banner**: Ask "Do you use a cookie consent banner or need GDPR compliance?" If yes, follow up with "Which cookie consent platform do you use?" (e.g., Cookiebot, OneTrust, CookieYes, custom solution)

### Optional Features

6. **Signup Tracking**: Ask "Do you want to track user signups in addition to purchases? (Recommended - provides better insights on affiliate performance)"

## Connect Payment Provider

If the user's Stripe account is not already connected in Affonso, tell them to do this before or alongside the code integration:

1. Go to https://affonso.io/app/affiliate-program/connect
2. Select Stripe
3. Enter the Stripe API key
4. Copy the webhook URL that Affonso provides
5. Add that webhook URL in the Stripe dashboard

If Stripe is already connected, continue with the code changes and keep the existing webhook connection in place.

## Workflow Decision Tree

```
1. Connect Stripe to Affonso if it is not connected yet

2. Install Tracking Script
   ├─ Uses GTM? → See GTM Integration Guide
   ├─ Has Cookie Consent? → See GDPR Consent Guide
   └─ Neither → Direct script installation

3. Track Signups (Optional)
   └─ Add signup tracking code

4. Pass Data to Stripe
   ├─ Checkout API → See Stripe Checkout API Guide
   ├─ Payment Links → See Payment Links Guide
   ├─ Buy Button → See Buy Button Guide
   ├─ Pricing Tables → See Pricing Tables Guide
   └─ Don't Know → Help identify integration method

5. Test Integration
   └─ Verify tracking works end-to-end
```

## Step 1: Install Tracking Script

The tracking script monitors affiliate traffic and creates the `affonso_referral` cookie that gets passed to Stripe.

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

## Step 3: Pass Referral Data to Stripe

The tracking script creates an `affonso_referral` cookie that must be passed to Stripe. Implementation varies by Stripe integration method.

### Identify Stripe Integration Method

If the user selected "I don't know", help them identify their integration method:

- **Checkout API**: They create Stripe sessions programmatically in backend code (Node.js, Python, PHP, etc.)
- **Payment Links**: They use links starting with `buy.stripe.com` that redirect to Stripe's hosted checkout
- **Buy Button**: They embed `<stripe-buy-button>` HTML elements on their website
- **Pricing Tables**: They embed `<stripe-pricing-table>` HTML elements on their website

### Integration Guides

Based on the identified method, read and implement the appropriate guide:

#### Stripe Checkout API
**When to use**: User creates checkout sessions programmatically in backend code

Read [Stripe Checkout API Guide](references/stripe-checkout-api.md) which includes:
- How to pass `affonso_referral` cookie as metadata
- Framework-specific examples (Next.js, Node.js, Python, Ruby, PHP)
- Testing with server logs

**Quick implementation** (Next.js example):
```javascript
import { cookies } from 'next/headers';

const cookieStore = await cookies();
const affonsoReferral = cookieStore.get('affonso_referral')?.value || '';

const session = await stripe.checkout.sessions.create({
  // ... existing config
  metadata: {
    affonso_referral: affonsoReferral,
  },
});
```

#### Stripe Payment Links
**When to use**: User has links starting with `buy.stripe.com`

Read [Payment Links Guide](references/stripe-payment-links.md) which includes:
- Enhancement script that automatically appends `client_reference_id` to links
- How the script works and when it runs
- Testing procedures

#### Stripe Buy Button
**When to use**: User has `<stripe-buy-button>` elements

Read [Buy Button Guide](references/stripe-buy-button.md) which includes:
- Enhancement script that sets `client-reference-id` attribute
- Installation instructions
- Testing procedures

#### Stripe Pricing Tables
**When to use**: User has `<stripe-pricing-table>` elements

Read [Pricing Tables Guide](references/stripe-pricing-tables.md) which includes:
- Enhancement script that sets `client-reference-id` attribute
- Installation instructions
- Testing procedures

## Step 4: Testing & Verification

### Test Tracking Script

1. Visit the website with test parameter: `yoursite.com?atp=test`
2. Open browser DevTools → Application → Cookies
3. Verify `affonso_referral` cookie is set (unless consent mode is enabled and consent not given)
4. Check Affonso dashboard for test entry

### Test Stripe Integration

**For Checkout API:**
- Trigger checkout flow and check server logs for `metadata: { affonso_referral: "..." }`

**For Payment Links, Buy Buttons, Pricing Tables:**
- Inspect the enhanced elements/URLs to verify `client_reference_id` is added
- DevTools → Elements tab → Find the Stripe element and check attributes/URL

### End-to-End Test

1. Visit site with `?atp=test` parameter
2. Complete a purchase using a 100% discount code (for live testing without payment)
3. Check Affonso dashboard to verify the conversion was tracked
4. Verify the test affiliate received credit for the sale

**Note:** Stripe test mode transactions won't appear in Affonso - use discount codes on live mode or check server logs/element attributes.

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

### Stripe Not Receiving Data
- **Checkout API**: Verify server code reads cookie and passes to metadata
- **Payment Links/Buy Button/Pricing Tables**: Check that enhancement script runs (look for `affonso_referral_ready` event)
- Inspect Network tab for script loading errors

### Tracking Not Appearing in Dashboard
- Ensure Stripe is connected in Affonso dashboard. If not, connect it first at https://affonso.io/app/affiliate-program/connect and add the Affonso webhook in Stripe.
- For testing, use live mode with discount codes (test mode doesn't sync)
- Check that the purchase completed successfully in Stripe

## Additional Resources

For framework-specific implementation guidance, the user can check:
- https://affonso.io/help/installation-guides (Next.js, React, Vue, WordPress, etc.)

## Next Steps

After successful integration:
1. Set up affiliate recruitment and program rules in Affonso dashboard
2. Create affiliate resources (landing pages, promotional materials)
3. Monitor performance and optimize commission structure
