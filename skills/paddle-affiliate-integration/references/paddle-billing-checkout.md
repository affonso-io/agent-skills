# Paddle Billing Checkout Integration

For users who use Paddle.js to open checkout sessions via `Paddle.Checkout.open()`.

## How It Works

1. The Affonso tracking script sets `window.affonso_referral` when a visitor arrives via an affiliate link
2. When checkout opens, the referral ID is passed through the `customData` parameter
3. Paddle includes `customData` in webhook events sent to your server
4. Affonso receives the webhook and credits the affiliate

## Implementation

When opening a Paddle checkout, include the `affonso_referral` value in `customData`:

```javascript
Paddle.Checkout.open({
  product: 12345,
  email: "customer@example.com",
  customData: {
    affonso_referral: window.affonso_referral
  },
});
```

This works entirely client-side — there is no need to read cookies on the server.

### With Existing customData

If you already pass `customData` in your checkout, add the `affonso_referral` field alongside your existing data:

```javascript
Paddle.Checkout.open({
  product: 12345,
  email: "customer@example.com",
  customData: {
    plan: "pro",
    source: "pricing-page",
    affonso_referral: window.affonso_referral
  },
});
```

### Dynamic Checkout Example

If you build the checkout configuration dynamically:

```javascript
function openCheckout(productId, userEmail) {
  const checkoutConfig = {
    product: productId,
    email: userEmail,
    customData: {
      affonso_referral: window.affonso_referral
    },
  };

  Paddle.Checkout.open(checkoutConfig);
}
```

## Testing

### 1. Verify Tracking Script

1. Visit your website with the test parameter: `yoursite.com?atp=test`
2. Open browser DevTools (F12)
3. Go to **Console** and type `window.affonso_referral`
4. You should see the test referral ID

### 2. Verify Checkout Integration

1. With `window.affonso_referral` set, trigger a checkout
2. Check your server logs for the webhook payload
3. Verify `customData` contains `affonso_referral` with the correct value

### 3. End-to-End Test

1. Visit site with `?atp=test` parameter
2. Complete a purchase using a discount code (for live testing without full payment)
3. Check Affonso dashboard to verify the conversion was tracked

**Important:** Paddle sandbox mode transactions won't appear in Affonso. Use discount codes on live mode for testing, or verify by checking server logs and webhook payloads.

## Troubleshooting

### `window.affonso_referral` is undefined
- Ensure the Affonso tracking script is installed in the `<head>` tag
- Visit the page with `?atp=test` to trigger tracking
- Check browser console for JavaScript errors related to the Affonso script

### customData Not Appearing in Webhooks
- Verify the `customData` parameter is included in your `Paddle.Checkout.open()` call
- Check that the checkout is opening successfully
- Inspect the Network tab in DevTools for the checkout request

### Conversions Not Showing in Affonso
- Ensure your Paddle account is connected at https://affonso.io/app/affiliate-program/connect/paddle
- Verify the webhook URL from Affonso is configured in Paddle's webhook settings
- Confirm the purchase completed successfully in your Paddle dashboard
- Remember that sandbox mode transactions won't sync to Affonso
