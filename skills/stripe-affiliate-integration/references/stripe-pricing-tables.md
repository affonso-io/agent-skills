# Stripe Pricing Tables Integration

For users using Stripe Pricing Tables on their website.

## Enhancement Script

Add this script to your website's `<head>` section, right after your Affonso tracking script:

```html
<!-- Your existing Affonso tracking script -->
<script
  async
  defer
  src="https://cdn.affonso.io/js/pixel.min.js"
  data-affonso="YOUR_PUBLIC_PROGRAM_ID"
  data-cookie_duration="YOUR_COOKIE_DURATION"
></script>

<!-- Add this enhancement script right after -->
<script>
  function enhanceStripePricingTables() {
    if (window.affonso_referral) {
      document.querySelectorAll('stripe-pricing-table').forEach((table) => {
        table.setAttribute('client-reference-id', window.affonso_referral);
      });
    }
  }

  setTimeout(enhanceStripePricingTables, 800);
  setTimeout(enhanceStripePricingTables, 1500);
  setTimeout(enhanceStripePricingTables, 2500);
  window.addEventListener('affonso_referral_ready', enhanceStripePricingTables);
</script>
```

That's it! Your Pricing Table will now automatically include affiliate tracking data when customers make purchases.

## Testing

1. Visit your site with `?atp=test` (test affiliate parameter)
2. Check browser DevTools → Application → Cookies → look for `affonso_referral`
3. Inspect the `<stripe-pricing-table>` element - it should have the `client-reference-id` attribute
4. Make a test purchase to verify tracking in your Affonso dashboard

**Note:** Use 100% discount codes for live testing without payments - Stripe test mode won't appear in Affonso.
