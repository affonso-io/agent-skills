# Dodo Checkout API Integration

For users implementing affiliate tracking with the Dodo Payments Checkout API.

## Implementation

Pass the `affonso_referral` cookie as metadata to your Dodo Payments session. This works for both one-time payments and subscriptions.

```javascript
import { cookies } from 'next/headers';
import DodoPayments from 'dodopayments';

// Get Affonso referral cookie (Next.js 15+)
// For Next.js 13-14: const cookieStore = cookies(); (without await)
const cookieStore = await cookies();
const affonsoReferral = cookieStore.get('affonso_referral')?.value || '';

const client = new DodoPayments({
  bearerToken: process.env.DODO_PAYMENTS_API_KEY,
});

// For one-time payments
const payment = await client.payments.create({
  // ... your existing config
  metadata: {
    affonso_referral: affonsoReferral, // Add this line
  },
});

// For subscriptions
const subscription = await client.subscriptions.create({
  // ... your existing config
  metadata: {
    affonso_referral: affonsoReferral, // Add this line
  },
});
```

## Testing

1. Visit your site with `?atp=test` (test affiliate parameter)
2. Check browser DevTools → Application → Cookies → look for `affonso_referral`
3. Trigger your checkout flow locally
4. Check your server logs/terminal - the Dodo session should show:
   ```
   metadata: { affonso_referral: "cmdhq6ayf..." }
   ```
5. If the metadata appears in your logs, you're all set!

**Note:** Use 100% discount codes for live testing without payments - Dodo Payments test mode won't appear in Affonso.
