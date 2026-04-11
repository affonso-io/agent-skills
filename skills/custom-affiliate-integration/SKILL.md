---
name: custom-affiliate-integration
description: Set up Affonso affiliate program tracking with any payment provider using the REST API. Use this skill when the user wants to implement an affiliate program with a payment provider that is not Stripe, Paddle, Polar, Creem, or Dodo Payments — such as Lemon Squeezy, Gumroad, Razorpay, Paystack, Xendit, Flutterwave, or any custom billing system. Handles tracking script installation, webhook processing, and manual commission creation via the Affonso API.
---

# Custom Payment Provider Affiliate Integration

This skill guides you through setting up Affonso affiliate tracking with any payment provider using the Affonso REST API.

## Overview

Affonso is an affiliate program platform that tracks referrals and commissions. This skill helps implement:

1. **Tracking script installation** - Monitor affiliate traffic and conversions
2. **Custom payment provider integration** - Pass referral data to your payment provider and create commissions via the Affonso API
3. **Optional features** - Signup tracking, GTM integration, GDPR compliance

**Important:** With custom payment providers, Affonso does NOT auto-calculate commissions. The developer must calculate and submit commission amounts manually via the Affonso API.

## Prerequisites & Information Gathering

Before beginning implementation, gather the following information by asking the user:

### Required Information

1. **Program ID**: Ask "What is your Affonso program ID? You can find it at https://affonso.io/app/affiliate-program"

2. **Cookie Duration**: Ask "How many days should the affiliate tracking cookie persist? (Common values: 30, 60, 90 days)"

3. **Payment Provider**: Ask "What payment provider do you use?" (e.g., Lemon Squeezy, Gumroad, Razorpay, Paystack, Xendit, Flutterwave, or another provider)

4. **Payment Integration Method**: Ask "How do you handle payments?" Provide options:
   - Payment provider SDK (client-side)
   - Payment provider API (server-side)
   - Hosted checkout page (redirect)
   - Embedded checkout widget
   - I don't know / Need help determining this

5. **Google Tag Manager**: Ask "Do you use Google Tag Manager to manage scripts on your website?"

6. **Cookie Consent Banner**: Ask "Do you use a cookie consent banner or need GDPR compliance?" If yes, follow up with "Which cookie consent platform do you use?" (e.g., Cookiebot, OneTrust, CookieYes, custom solution)

### Optional Features

7. **Signup Tracking**: Ask "Do you want to track user signups in addition to purchases? (Recommended - provides better insights on affiliate performance)"

## Workflow Decision Tree

```
1. Install Tracking Script
   ├─ Uses GTM? → See GTM Integration Guide
   ├─ Has Cookie Consent? → See GDPR Consent Guide
   └─ Neither → Direct script installation

2. Track Signups (Optional)
   └─ Add signup tracking code

3. Pass Referral Data to Payment Provider
   ├─ Frontend approach → Use window.affonso_referral
   └─ Backend approach → Read affonso_referral cookie

4. Handle Webhooks
   ├─ Extract referral ID from metadata
   ├─ Update referral status via API
   └─ Create commission via API

5. Handle Refunds
   └─ Update commission for full/partial refunds
```

## Step 1: Install Tracking Script

The tracking script monitors affiliate traffic and creates the `affonso_referral` cookie that gets passed to your payment provider.

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
// After successful user registration — basic usage
window.Affonso.signup(userEmail);

// Or with advanced options for richer tracking
window.Affonso.signup({
  email: userEmail,
  externalUserId: userId,
  name: userName
});
```

**Benefits of signup tracking:**
- See which affiliates drive the most registrations
- Calculate conversion rates from clicks to signups
- Optimize funnel based on affiliate performance

## Step 3: Pass Referral Data to Payment Provider

The tracking script creates an `affonso_referral` cookie that must be passed to your payment provider as metadata. There are two approaches depending on your architecture.

### Frontend Approach

Use `window.affonso_referral` to read the referral ID on the client side and include it when creating a checkout or payment:

```javascript
// Read the referral ID from the global variable
const referralId = window.affonso_referral;

// Pass it to your payment provider's checkout as metadata
createCheckout({
  metadata: { affonso_referral: referralId }
});
```

### Backend Approach

Read the `affonso_referral` cookie from the incoming request and pass it to the payment provider server-side:

```javascript
// Node.js / Express example
const referralId = req.cookies['affonso_referral'] || '';

const checkout = await paymentProvider.createCheckout({
  metadata: { affonso_referral: referralId }
});
```

### Provider-Specific Metadata Fields

Different payment providers use different field names for custom metadata:

| Provider | Metadata Field |
|---|---|
| Lemon Squeezy | `custom_data` or `checkout_data.custom` |
| Gumroad | `custom_fields` |
| Razorpay | `notes` |
| Paystack | `metadata` |
| Xendit | `metadata` |
| Flutterwave | `meta` |

Adapt the examples above to use the correct field name for the user's payment provider.

For detailed API examples and complete webhook handler code, read the [Custom Payment Provider API Guide](references/custom-payment-provider-api.md).

## Step 4: Handle Webhooks

Set up a webhook endpoint to receive payment events from your payment provider. When a payment succeeds, extract the referral ID from the metadata and use the Affonso API to update the referral and create a commission.

### Webhook Handler

```javascript
app.post('/webhooks/payment', async (req, res) => {
  const event = req.body;

  // Extract the referral ID from metadata (adjust field name for your provider)
  const referralId = event.data.metadata?.affonso_referral;
  if (!referralId) return res.json({ received: true });

  // 1. Update referral status to "customer"
  await fetch(`https://api.affonso.io/v1/referrals/${referralId}`, {
    method: 'PUT',
    headers: {
      'Authorization': 'Bearer sk_live_your_api_key',
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      email: event.data.customer_email,
      customer_id: event.data.customer_id,
      status: 'customer',
      name: event.data.customer_name,
    }),
  });

  // 2. Create a commission (you must calculate the amount yourself)
  await fetch('https://api.affonso.io/v1/commissions', {
    method: 'POST',
    headers: {
      'Authorization': 'Bearer sk_live_your_api_key',
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      referral_id: referralId,
      sale_amount: event.data.amount,
      commission_amount: event.data.amount * 0.20, // Example: 20% commission
      sale_amount_currency: 'USD',
      commission_currency: 'USD',
      payment_intent_id: event.data.payment_id,
      is_subscription: false,
      sales_status: 'complete',
    }),
  });

  res.json({ received: true });
});
```

**Available referral statuses:** lead, trialing, customer, active, canceled, rejected

For complete API reference and cURL examples, read the [Custom Payment Provider API Guide](references/custom-payment-provider-api.md).

## Step 5: Handle Refunds

When a refund occurs, update the commission via the Affonso API.

### Full Refund

```javascript
await fetch(`https://api.affonso.io/v1/commissions/${commissionId}`, {
  method: 'PUT',
  headers: {
    'Authorization': 'Bearer sk_live_your_api_key',
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    sale_amount: 0,
    commission_amount: 0,
    sales_status: 'refunded',
  }),
});
```

### Partial Refund

```javascript
await fetch(`https://api.affonso.io/v1/commissions/${commissionId}`, {
  method: 'PUT',
  headers: {
    'Authorization': 'Bearer sk_live_your_api_key',
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    sale_amount: 49.50, // Remaining amount after refund
    commission_amount: 9.90, // Recalculated commission
    sales_status: 'partial_refunded',
  }),
});
```

## Testing & Verification

### Test Tracking Script

1. Visit the website with test parameter: `yoursite.com?atp=test`
2. Open browser DevTools → Application → Cookies
3. Verify `affonso_referral` cookie is set (unless consent mode is enabled and consent not given)
4. Check Affonso dashboard for test entry

### Test Webhook Integration

1. Set up a test webhook endpoint (use tools like ngrok for local development)
2. Trigger a test payment through your payment provider
3. Verify the webhook receives the event with `affonso_referral` in metadata
4. Check Affonso dashboard for the referral status update and commission creation

### End-to-End Test

1. Visit site with `?atp=test` parameter
2. Complete a test purchase through your payment provider
3. Check Affonso dashboard to verify the referral was updated and the commission was created
4. Verify the test affiliate received credit for the sale

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

### Payment Provider Not Receiving Referral Data
- **Frontend**: Check that `window.affonso_referral` is defined before creating checkout
- **Backend**: Verify the cookie is being read correctly from the request
- Inspect Network tab for script loading errors
- Ensure you are using the correct metadata field name for your provider

### Commission Not Created
- Verify your Affonso API key is correct
- Check that the referral ID is valid and exists in Affonso
- Ensure the commission amount is calculated correctly
- Check webhook logs for API response errors

### Tracking Not Appearing in Dashboard
- Verify your API key has the correct permissions
- Check that the webhook endpoint is publicly accessible
- Ensure the payment completed successfully in your payment provider

## Additional Resources

For framework-specific implementation guidance, the user can check:
- https://affonso.io/help/installation-guides (Next.js, React, Vue, WordPress, etc.)

For detailed API documentation:
- Read the [Custom Payment Provider API Guide](references/custom-payment-provider-api.md)

## Next Steps

After successful integration:
1. Set up affiliate recruitment and program rules in Affonso dashboard
2. Create affiliate resources (landing pages, promotional materials)
3. Monitor performance and optimize commission structure
