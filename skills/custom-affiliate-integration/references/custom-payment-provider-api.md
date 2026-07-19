# Custom Payment Provider API Integration

This reference file is the implementation layer for custom/backend-driven Affonso integrations.

Use it after reading:

- https://docs.affonso.io/api/server-side-tracking
- `../SKILL.md`

## Endpoint Chooser

| Use case | Endpoint | Why |
|---|---|---|
| Purchase or renewal where Affonso should calculate commissions | `POST /conversions` | Best default for backend-driven revenue events |
| Lead, trial, milestone, or conversion through one normalized event pipeline | `POST /events` | One ingestion path for multiple lifecycle events |
| Exact commission known in advance | `POST /commissions` | Manual override or import-style flow |
| Refund for a conversion created by Affonso | `POST /conversions/{id}/refund` | Preferred refund handling |

## Passing Referral Data Into Checkout

### Browser-created checkout

```javascript
const referralId = window.affonso_referral || '';

createCheckout({
  metadata: {
    affonso_referral: referralId,
  },
});
```

### Server-created checkout

```javascript
const referralId = req.cookies['affonso_referral'] || '';

const checkout = await paymentProvider.createCheckout({
  metadata: {
    affonso_referral: referralId,
  },
});
```

### Common metadata field names

| Provider | Field |
|---|---|
| Lemon Squeezy | `custom_data` or `checkout_data.custom` |
| Gumroad | `custom_fields` |
| Razorpay | `notes` |
| Paystack | `metadata` |
| Xendit | `metadata` |
| Flutterwave | `meta` |

## Preferred Identifier Strategy

Prefer these in order:

1. `external_user_id`
2. `referral_id`
3. `customer_id`

If the provider webhook exposes only its own customer or subscription identifiers, store the mapping when checkout is created.

## Example: Conversion-Based Backend Flow

Use this when Affonso should apply program incentives and calculate commissions.

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
      interval: event.data.interval ?? undefined,
      product_ids: event.data.product_ids ?? undefined,
      price_ids: event.data.price_ids ?? undefined,
    }),
  });

  res.json({ received: true });
});
```

## Example: Normalized Events Flow

Use this when the backend wants one unified pipeline.

```bash
curl -X POST "https://api.affonso.io/v1/events" \
  -H "Authorization: Bearer sk_live_your_api_key" \
  -H "Content-Type: application/json" \
  -d '{
    "event_name": "signup_completed",
    "event_type": "lead",
    "external_user_id": "usr_123",
    "external_event_id": "evt_signup_123",
    "metadata": {
      "email": "customer@example.com"
    }
  }'
```

```bash
curl -X POST "https://api.affonso.io/v1/events" \
  -H "Authorization: Bearer sk_live_your_api_key" \
  -H "Content-Type: application/json" \
  -d '{
    "event_name": "invoice_paid",
    "event_type": "conversion",
    "customer_id": "cust_123",
    "external_event_id": "evt_invoice_123",
    "sale_amount": 99,
    "sale_amount_currency": "USD",
    "is_subscription": true
  }'
```

## Example: Manual Commission Flow

Use this only when the backend owns the commission math.

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

## Refund Handling

### Refund a conversion

```bash
curl -X POST "https://api.affonso.io/v1/conversions/conv_123/refund" \
  -H "Authorization: Bearer sk_live_your_api_key" \
  -H "Content-Type: application/json" \
  -d '{
    "external_event_id": "refund_123",
    "refund_amount": 49.50,
    "refund_currency": "USD"
  }'
```

### Refund a manual commission

If the original sale used `POST /commissions`, update that commission record manually in the user's refund handler.

## Idempotency

Use a stable upstream identifier for `external_event_id`:

- order ID
- invoice ID
- payment event ID
- refund event ID

Reuse the exact same value when retrying.

## Testing Checklist

1. Load the site with `?atp=test`
2. Confirm `affonso_referral` exists
3. Confirm checkout metadata includes the referral value
4. Trigger a real or test backend event
5. Confirm the Affonso API request succeeds
6. Confirm the dashboard shows the expected referral or conversion
7. Trigger a refund event and verify downstream state
