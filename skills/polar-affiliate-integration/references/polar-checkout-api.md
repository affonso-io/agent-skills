# Polar Checkout API Integration

For users implementing affiliate tracking with the Polar Checkout API.

## Implementation

Pass the `affonso_referral` cookie as metadata to your Polar checkout session:

```javascript
import { cookies } from 'next/headers';

// Get Affonso referral cookie (Next.js 15+)
// For Next.js 13-14: const cookieStore = cookies(); (without await)
const cookieStore = await cookies();
const affonsoReferral = cookieStore.get('affonso_referral')?.value || '';

// Add to your Polar checkout
const checkout = await polar.checkouts.create({
  // ... your existing config
  metadata: {
    affonso_referral: affonsoReferral, // Add this line
  },
});
```

## Testing

1. Visit your site with `?atp=test` (test affiliate parameter)
2. Check browser DevTools -> Application -> Cookies -> look for `affonso_referral`
3. Trigger your checkout flow locally
4. Check your server logs/terminal - the Polar checkout should show:
   ```
   metadata: { affonso_referral: "cmdhq6ayf..." }
   ```
5. If the metadata appears in your logs, you're all set!

**Note:** Use 100% discount codes for live testing without payments, or test locally with server logs - Polar test mode won't appear in Affonso.

## Framework-Specific Examples

### Node.js/Express
```javascript
const affonsoReferral = req.cookies.affonso_referral || '';

const checkout = await polar.checkouts.create({
  // ... your config
  metadata: {
    affonso_referral: affonsoReferral,
  },
});
```

### Python/Flask
```python
affonso_referral = request.cookies.get('affonso_referral', '')

checkout = polar.checkouts.create(
    # ... your config
    metadata={
        'affonso_referral': affonso_referral,
    },
)
```

### Python/Django
```python
affonso_referral = request.COOKIES.get('affonso_referral', '')

checkout = polar.checkouts.create(
    # ... your config
    metadata={
        'affonso_referral': affonso_referral,
    },
)
```
