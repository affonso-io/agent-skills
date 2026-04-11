# Custom Payment Provider API Integration

Detailed guide for integrating Affonso affiliate tracking with any payment provider using the Affonso REST API.

## Overview

When using a payment provider that is not natively supported by Affonso (such as Lemon Squeezy, Gumroad, Razorpay, Paystack, Xendit, Flutterwave, or a custom billing system), you must manually:

1. Pass the referral ID to your payment provider as metadata
2. Handle payment webhooks to extract the referral ID
3. Update referral status via the Affonso API
4. Create commissions via the Affonso API
5. Handle refunds by updating commissions

**Important:** Affonso does NOT auto-calculate commissions for custom providers. You must calculate and submit commission amounts yourself.

## Provider-Specific Metadata Fields

Different payment providers use different field names for custom metadata. Use the correct field for your provider:

| Provider | Metadata Field | Example |
|---|---|---|
| Lemon Squeezy | `custom_data` or `checkout_data.custom` | `custom_data: { affonso_referral: referralId }` |
| Gumroad | `custom_fields` | `custom_fields: { affonso_referral: referralId }` |
| Razorpay | `notes` | `notes: { affonso_referral: referralId }` |
| Paystack | `metadata` | `metadata: { affonso_referral: referralId }` |
| Xendit | `metadata` | `metadata: { affonso_referral: referralId }` |
| Flutterwave | `meta` | `meta: { affonso_referral: referralId }` |

For providers not listed above, consult your payment provider's documentation for how to attach custom metadata to a checkout session or payment.

## Step 1: Pass Referral ID to Payment Provider

### Frontend Approach

Read the referral ID from the global variable set by the Affonso tracking script and include it when creating a checkout:

```javascript
// The Affonso tracking script sets this global variable
const referralId = window.affonso_referral;

// Pass it as metadata when creating a checkout
// (adjust the field name for your provider — see table above)
createCheckout({
  metadata: { affonso_referral: referralId }
});
```

### Backend Approach (Node.js)

Read the `affonso_referral` cookie from the request and pass it to the payment provider server-side:

```javascript
// Node.js / Express example
const referralId = req.cookies['affonso_referral'] || '';

const checkout = await paymentProvider.createCheckout({
  // ... existing config
  metadata: { affonso_referral: referralId }
});
```

### Backend Approach (Python)

```python
# Flask / Django example
referral_id = request.cookies.get('affonso_referral', '')

checkout = payment_provider.create_checkout(
    # ... existing config
    metadata={'affonso_referral': referral_id}
)
```

### Backend Approach (PHP)

```php
$referralId = $_COOKIE['affonso_referral'] ?? '';

$checkout = $paymentProvider->createCheckout([
    // ... existing config
    'metadata' => ['affonso_referral' => $referralId]
]);
```

## Step 2: Handle Webhooks

Set up a webhook endpoint to receive payment events from your provider. When a payment succeeds, extract the referral ID and process it.

### Webhook Handler (Node.js / Express)

```javascript
app.post('/webhooks/payment', async (req, res) => {
  const event = req.body;

  // Extract referral ID from metadata (adjust path for your provider)
  const referralId = event.data.metadata?.affonso_referral;

  // No referral — not an affiliate sale, skip processing
  if (!referralId) return res.json({ received: true });

  // 1. Update referral status
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

  // 2. Create commission (calculate amount yourself)
  const saleAmount = event.data.amount; // in your currency
  const commissionRate = 0.20; // example: 20%
  const commissionAmount = saleAmount * commissionRate;

  await fetch('https://api.affonso.io/v1/commissions', {
    method: 'POST',
    headers: {
      'Authorization': 'Bearer sk_live_your_api_key',
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      referral_id: referralId,
      sale_amount: saleAmount,
      commission_amount: commissionAmount,
      sale_amount_currency: 'USD',
      commission_currency: 'USD',
      payment_intent_id: event.data.payment_id, // unique payment identifier
      is_subscription: false,
      sales_status: 'complete',
    }),
  });

  res.json({ received: true });
});
```

## Step 3: Update Referral Status

Use the Affonso API to update a referral's status when a customer takes action (signs up, starts a trial, makes a purchase, etc.).

### API Endpoint

```
PUT https://api.affonso.io/v1/referrals/{referralId}
```

### cURL Example

```bash
curl -X PUT "https://api.affonso.io/v1/referrals/{referralId}" \
  -H "Authorization: Bearer sk_live_your_api_key" \
  -H "Content-Type: application/json" \
  -d '{"email":"customer@example.com","customer_id":"cust_123","status":"customer","name":"John Doe"}'
```

### Available Statuses

| Status | Description |
|---|---|
| `lead` | Visitor clicked an affiliate link |
| `trialing` | Customer started a free trial |
| `customer` | Customer made a purchase |
| `active` | Customer has an active subscription |
| `canceled` | Customer canceled their subscription |
| `rejected` | Referral was rejected (fraud, self-referral, etc.) |

## Step 4: Create Commission

Use the Affonso API to create a commission when a referred customer makes a purchase.

### API Endpoint

```
POST https://api.affonso.io/v1/commissions
```

### cURL Example

```bash
curl -X POST "https://api.affonso.io/v1/commissions" \
  -H "Authorization: Bearer sk_live_your_api_key" \
  -H "Content-Type: application/json" \
  -d '{"referral_id":"{referralId}","sale_amount":99.00,"commission_amount":19.80,"sale_amount_currency":"USD","commission_currency":"USD","payment_intent_id":"pi_unique_id","is_subscription":false,"sales_status":"complete"}'
```

### Request Body Fields

| Field | Type | Required | Description |
|---|---|---|---|
| `referral_id` | string | Yes | The Affonso referral ID |
| `sale_amount` | number | Yes | Total sale amount |
| `commission_amount` | number | Yes | Commission amount to pay the affiliate |
| `sale_amount_currency` | string | Yes | Currency code (e.g., "USD", "EUR") |
| `commission_currency` | string | Yes | Currency code for the commission |
| `payment_intent_id` | string | Yes | Unique payment identifier from your provider |
| `is_subscription` | boolean | Yes | Whether this is a recurring subscription payment |
| `sales_status` | string | Yes | Status of the sale (see below) |

### Sales Status Values

| Status | Description |
|---|---|
| `complete` | Payment was successful |
| `refunded` | Full refund was issued |
| `partial_refunded` | Partial refund was issued |

## Step 5: Handle Refunds

When a refund occurs, update the commission to reflect the refund.

### Full Refund

```bash
curl -X PUT "https://api.affonso.io/v1/commissions/{commissionId}" \
  -H "Authorization: Bearer sk_live_your_api_key" \
  -H "Content-Type: application/json" \
  -d '{"sale_amount":0,"commission_amount":0,"sales_status":"refunded"}'
```

### Partial Refund

```bash
curl -X PUT "https://api.affonso.io/v1/commissions/{commissionId}" \
  -H "Authorization: Bearer sk_live_your_api_key" \
  -H "Content-Type: application/json" \
  -d '{"sale_amount":49.50,"commission_amount":9.90,"sales_status":"partial_refunded"}'
```

### Refund Webhook Handler (Node.js)

```javascript
app.post('/webhooks/refund', async (req, res) => {
  const event = req.body;
  const commissionId = event.data.commission_id; // You need to store this mapping

  if (event.data.refund_type === 'full') {
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
  } else {
    // Partial refund — recalculate amounts
    const remainingAmount = event.data.amount_after_refund;
    const commissionRate = 0.20;

    await fetch(`https://api.affonso.io/v1/commissions/${commissionId}`, {
      method: 'PUT',
      headers: {
        'Authorization': 'Bearer sk_live_your_api_key',
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        sale_amount: remainingAmount,
        commission_amount: remainingAmount * commissionRate,
        sales_status: 'partial_refunded',
      }),
    });
  }

  res.json({ received: true });
});
```

## Authentication

All API requests require a Bearer token in the Authorization header:

```
Authorization: Bearer sk_live_your_api_key
```

You can find your API key in the Affonso dashboard at https://affonso.io/app/settings.

## Error Handling

The Affonso API returns standard HTTP status codes:

| Code | Description |
|---|---|
| `200` | Success |
| `400` | Bad request (check request body) |
| `401` | Unauthorized (check API key) |
| `404` | Resource not found (check referral/commission ID) |
| `422` | Validation error (check required fields) |
| `500` | Server error (retry the request) |

### Recommended Error Handling

```javascript
const response = await fetch('https://api.affonso.io/v1/commissions', {
  method: 'POST',
  headers: {
    'Authorization': 'Bearer sk_live_your_api_key',
    'Content-Type': 'application/json',
  },
  body: JSON.stringify(commissionData),
});

if (!response.ok) {
  const error = await response.json();
  console.error('Affonso API error:', response.status, error);
  // Implement retry logic for 5xx errors
}
```

## Testing

### Test with cURL

1. Create a test referral by visiting your site with `?atp=test`
2. Note the referral ID from the Affonso dashboard
3. Use the cURL examples above to test each API endpoint
4. Verify results in the Affonso dashboard

### Webhook Testing

Use tools like ngrok or localtunnel to expose your local webhook endpoint for testing:

```bash
# Start ngrok
ngrok http 3000

# Use the ngrok URL as your webhook endpoint in your payment provider
# e.g., https://abc123.ngrok.io/webhooks/payment
```
