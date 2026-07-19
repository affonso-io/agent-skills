---
name: custom-affiliate-integration
description: Set up Affonso affiliate tracking with any payment provider or custom backend using Affonso's API. Use this skill whenever the user does not use Stripe, Paddle, Polar, Creem, or Dodo Payments, or when they want to bypass native integrations and wire Affonso through their own backend. Handles tracking script installation, referral propagation, choosing between `/conversions`, `/events`, and `/commissions`, webhook processing, refunds, signup tracking, GTM, and GDPR consent.
---

# Custom / API Affiliate Integration

This skill covers the fully custom integration path for Affonso.

Use it when the user:

- uses a provider without a native Affonso integration
- wants to keep their existing billing stack and send trusted backend events to Affonso
- wants AI to implement the integration end-to-end with Affonso's API

## Recommended Path

Prefer Affonso's server-side tracking API when the user's backend already knows what happened.

- Use `POST /conversions` when Affonso should calculate commissions automatically.
- Use `POST /events` when the backend sends normalized lifecycle events such as lead, trial, milestone, or conversion.
- Use `POST /commissions` only when the exact commission amount is already known and must be submitted manually.
- Use `POST /conversions/{id}/refund` for refund handling when the original sale was created through `POST /conversions`.

Read both of these before implementing:

- [Server-Side Tracking Docs](https://docs.affonso.io/api/server-side-tracking)
- [Custom Payment Provider API Guide](references/custom-payment-provider-api.md)

Important: this custom/API path does not require a native payment-provider connection in the Affonso dashboard. The agent should implement the backend flow directly with the user's Affonso API key.

## Information To Gather

Before writing code, gather or infer:

1. **Program ID**: "What is your Affonso program ID? You can find it at https://affonso.io/app/affiliate-program"
2. **Affonso API key**: Confirm the backend can call `https://api.affonso.io/v1/*`
3. **Payment provider / billing system**: Lemon Squeezy, Gumroad, Chargebee, Razorpay, Paystack, Xendit, Flutterwave, custom Stripe wrapper, in-house billing, etc.
4. **Checkout architecture**: Hosted checkout, embedded widget, provider SDK, backend-created sessions, or a fully custom billing flow
5. **Stable identifier**: Whether the backend has `external_user_id`, `customer_id`, or can persist `referral_id`
6. **Webhook/events source**: Which backend event confirms signup, trial, purchase, renewal, or refund
7. **Cookie duration**
8. **Google Tag Manager**
9. **Cookie consent / GDPR**
10. **Signup tracking requirement**
11. **Commission model**: Whether Affonso should calculate commissions from program incentives, or the backend should send exact commission amounts manually

## Workflow

```
1. Install the Affonso tracking script
   |- Uses GTM? -> Read the GTM guide
   |- Has cookie consent? -> Read the GDPR guide
   +- Neither -> Install directly in <head>

2. Decide which identifier the backend will send
   |- external_user_id -> Preferred when available
   |- referral_id -> Best when captured at checkout time
   +- customer_id -> Good when provider customer IDs are stable

3. Pass the referral identifier through checkout metadata
   |- Client-side checkout -> Use window.affonso_referral
   +- Server-side checkout -> Read affonso_referral from cookies

4. Implement backend event ingestion
   |- /conversions -> Affonso calculates commissions
   |- /events -> Lifecycle + normalized conversion events
   +- /commissions -> Manual commission amounts

5. Implement refunds
   |- /conversions/{id}/refund -> Preferred for conversion-based flows
   +- Update commission manually -> Only for manual /commissions flows

6. Test the end-to-end flow
```

## Step 1: Install Tracking Script

The tracking script captures affiliate attribution and exposes it to the app as:

- the `affonso_referral` cookie
- `window.affonso_referral` on the client

### GTM vs Direct Installation

If the user uses Google Tag Manager:

- Read and follow the [GTM Integration Guide](references/gtm-integration.md)

If the user does not use GTM, install this directly in the site's `<head>`:

```html
<script
  async
  defer
  src="https://cdn.affonso.io/js/pixel.min.js"
  data-affonso="YOUR_PUBLIC_PROGRAM_ID"
  data-cookie_duration="YOUR_COOKIE_DURATION"
></script>
```

If the user requires consent mode, read [GDPR Consent Guide](references/gdpr-consent.md) and use:

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

## Step 2: Track Signups When Useful

If the user wants signup attribution in addition to paid conversions and already uses the browser pixel, add:

```javascript
window.Affonso.signup({
  email: userEmail,
  externalUserId: userId,
  name: userName,
});
```

Prefer including `externalUserId` when the product has a durable internal user ID. That same ID can later be reused for server-side events.

If the user wants to record the signup from their backend instead of the browser, use `POST /v1/signups`. This endpoint is the server-side equivalent of `Affonso.signup()` and requires the original `click_id` from `POST /v1/clicks`:

```bash
curl -X POST "https://api.affonso.io/v1/signups" \
  -H "Authorization: Bearer sk_live_your_api_key" \
  -H "Content-Type: application/json" \
  -d '{
    "click_id": "ref_clk_xyz789",
    "email": "jane@example.com",
    "external_user_id": "user_42",
    "name": "Jane Doe"
  }'
```

Use `POST /v1/signups` when the backend owns the signup event. Use `POST /events` only when the team wants a normalized event pipeline and the corresponding identifiers already resolve to the same referral in Affonso.

## Step 3: Pass Referral Data Through Checkout

The backend needs a way to connect the later billing event back to the original referral.

### Client-Side Checkout

```javascript
const referralId = window.affonso_referral || '';

createCheckout({
  metadata: {
    affonso_referral: referralId,
  },
});
```

### Server-Side Checkout

```javascript
const referralId = req.cookies['affonso_referral'] || '';

const checkout = await paymentProvider.createCheckout({
  metadata: {
    affonso_referral: referralId,
  },
});
```

### Common Metadata Field Names

| Provider | Field |
|---|---|
| Lemon Squeezy | `custom_data` or `checkout_data.custom` |
| Gumroad | `custom_fields` |
| Razorpay | `notes` |
| Paystack | `metadata` |
| Xendit | `metadata` |
| Flutterwave | `meta` |

If the provider uses a different field, adapt the code to whatever metadata/custom-data mechanism that provider supports.

## Step 4: Choose the Affonso API Endpoint

### Use `POST /conversions` by default

Choose this when the backend knows a revenue event happened and the user wants Affonso to apply the team's incentive rules automatically.

Include `referral_id` on the first conversion unless the billing customer is already mapped to an existing Affonso referral.

```bash
curl -X POST "https://api.affonso.io/v1/conversions" \
  -H "Authorization: Bearer sk_live_your_api_key" \
  -H "Content-Type: application/json" \
  -d '{
    "referral_id": "ref_123",
    "customer_id": "cust_123",
    "sale_amount": 99.00,
    "sale_amount_currency": "USD",
    "external_event_id": "invoice_123",
    "sales_status": "complete",
    "is_subscription": true,
    "interval": "monthly"
  }'
```

### Use `POST /events` for normalized backend events

Choose this when the backend wants one endpoint for lead, trial, milestone, and conversion tracking.

```bash
curl -X POST "https://api.affonso.io/v1/events" \
  -H "Authorization: Bearer sk_live_your_api_key" \
  -H "Content-Type: application/json" \
  -d '{
    "event_name": "trial_started",
    "event_type": "trial",
    "external_user_id": "usr_104982",
    "external_event_id": "evt_trial_001"
  }'
```

### Use `POST /commissions` only for manual commission flows

Choose this only if the backend must send the exact commission amount instead of letting Affonso calculate it.

```bash
curl -X POST "https://api.affonso.io/v1/commissions" \
  -H "Authorization: Bearer sk_live_your_api_key" \
  -H "Content-Type: application/json" \
  -d '{
    "referral_id": "ref_123",
    "sale_amount": 99.00,
    "commission_amount": 19.80,
    "sale_amount_currency": "USD",
    "commission_currency": "USD",
    "payment_intent_id": "pay_123",
    "is_subscription": false,
    "sales_status": "complete"
  }'
```

### Identifier Choice

Prefer identifiers in this order:

1. `external_user_id` when the product has a stable internal user ID
2. `referral_id` when the checkout flow already captured the Affonso referral
3. `customer_id` when matching against the billing provider is the cleanest option

If multiple identifiers are sent, they must all resolve to the same referral.

### Idempotency

Always send a stable `external_event_id` on retryable backend events such as:

- order ID
- invoice ID
- checkout session ID
- payment event ID

Reuse the exact same `external_event_id` on retries.

## Step 5: Implement Webhooks or Backend Event Handlers

The agent should wire the user's existing payment-success and refund events into Affonso.

For a conversion-based flow:

```javascript
app.post('/webhooks/payment', async (req, res) => {
  const event = req.body;

  await fetch('https://api.affonso.io/v1/conversions', {
    method: 'POST',
    headers: {
      Authorization: `Bearer ${process.env.AFFONSO_API_KEY}`,
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      customer_id: event.data.customer_id,
      referral_id: event.data.metadata?.affonso_referral,
      external_event_id: event.data.id,
      sale_amount: event.data.amount,
      sale_amount_currency: event.data.currency,
      sales_status: 'complete',
      is_subscription: Boolean(event.data.subscription_id),
    }),
  });

  res.json({ received: true });
});
```

Include `referral_id` on the first conversion event unless `customer_id` already maps to an existing Affonso referral. Once the billing customer has been linked, `customer_id` becomes a stable identifier for later renewals and updates.

## Step 6: Handle Refunds

For conversions created via `POST /conversions`, prefer:

```bash
curl -X POST "https://api.affonso.io/v1/conversions/conv_123/refund" \
  -H "Authorization: Bearer sk_live_your_api_key" \
  -H "Content-Type: application/json" \
  -d '{
    "external_event_id": "refund_123",
    "amount": 49.50,
    "currency": "USD"
  }'
```

If the integration uses manual `POST /commissions`, the backend must persist the created commission ID and update that commission manually as part of refund handling.

Example manual commission refund update:

```bash
curl -X PUT "https://api.affonso.io/v1/commissions/com_123" \
  -H "Authorization: Bearer sk_live_your_api_key" \
  -H "Content-Type: application/json" \
  -d '{
    "sale_amount": 49.50,
    "commission_amount": 9.90,
    "sales_status": "partial_refunded"
  }'
```

## Step 7: Verify The Integration

Test in this order:

1. Visit the site with `?atp=test`
2. Confirm `affonso_referral` exists in cookies or `window.affonso_referral`
3. Start checkout and verify the referral value is attached to checkout metadata
4. Trigger the backend purchase event
5. Confirm the Affonso API call succeeds
6. Check the Affonso dashboard for the referral/conversion
7. Trigger a refund and verify the downstream refund state in Affonso

## Troubleshooting

### Referral Never Reaches The Backend

- Check that the tracking script is loaded in `<head>`
- Check consent-mode behavior if cookies are blocked until opt-in
- Confirm the checkout metadata actually includes the referral value

### Affonso Rejects The API Request

- Verify the API key
- Verify the identifier resolves to an existing referral
- Verify `sale_amount_currency` and similar required fields are present
- Verify `external_event_id` stays stable on retries

### Commissions Look Wrong

- If the user wants automatic incentive logic, use `POST /conversions` instead of manual `POST /commissions`
- If the user insists on manual commissions, confirm the backend's commission math and refund math

## Additional Resources

- https://docs.affonso.io/api/server-side-tracking
- https://affonso.io/help/integrations/custom/custom-payment-provider-api
- https://affonso.io/help/installation-guides
