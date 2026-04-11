# Stripe Payment Links Integration

For users using Stripe Payment Links (URLs starting with `buy.stripe.com`).

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
  function enhanceStripeLinks() {
    if (window.affonso_referral) {
      document.querySelectorAll('a[href*="buy.stripe.com"]').forEach((link) => {
        if (link.href.includes('client_reference_id=')) return;
        const separator = link.href.includes('?') ? '&' : '?';
        link.href =
          link.href +
          separator +
          'client_reference_id=' +
          window.affonso_referral;
      });
    }
  }

  setTimeout(enhanceStripeLinks, 800);
  setTimeout(enhanceStripeLinks, 1500);
  setTimeout(enhanceStripeLinks, 2500);
  window.addEventListener('affonso_referral_ready', enhanceStripeLinks);
</script>
```

## How It Works

1. Visitor clicks affiliate link → Affonso sets tracking cookie
2. Enhancement script runs → Finds all Payment Links on your page
3. Adds referral tracking → Appends `client_reference_id` to each link
4. Customer purchases → Stripe receives tracking data → Affiliate gets credited

## Testing

1. Visit your site with `?atp=test` (test affiliate parameter)
2. Check browser DevTools → Application → Cookies → look for `affonso_referral`
3. Inspect your Payment Links - URLs should now include `client_reference_id=`
4. Make a test purchase to verify tracking in your Affonso dashboard

**Note:** Use 100% discount codes for live testing without payments - Stripe test mode won't appear in Affonso.
