# Dodo Payment Links Integration

For users using Dodo Payment Links (URLs containing `checkout.dodopayments.com`).

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
  function enhanceDodoLinks() {
    if (window.affonso_referral) {
      document.querySelectorAll('a[href*="checkout.dodopayments.com"]').forEach((link) => {
        if (link.href.includes('metadata_affonso_referral=')) return;
        const separator = link.href.includes('?') ? '&' : '?';
        link.href =
          link.href +
          separator +
          'metadata_affonso_referral=' +
          window.affonso_referral;
      });
    }
  }

  setTimeout(enhanceDodoLinks, 800);
  setTimeout(enhanceDodoLinks, 1500);
  setTimeout(enhanceDodoLinks, 2500);
  window.addEventListener('affonso_referral_ready', enhanceDodoLinks);
</script>
```

## How It Works

1. Visitor clicks affiliate link → Affonso sets tracking cookie
2. Enhancement script runs → Finds all Dodo Payment Links on your page
3. Adds referral tracking → Appends `metadata_affonso_referral` to each link
4. Customer purchases → Dodo Payments receives tracking data → Affiliate gets credited

## Testing

1. Visit your site with `?atp=test` (test affiliate parameter)
2. Check browser DevTools → Application → Cookies → look for `affonso_referral`
3. Inspect your Payment Links - URLs should now include `metadata_affonso_referral=`
4. Make a test purchase to verify tracking in your Affonso dashboard

**Note:** Use 100% discount codes for live testing without payments - Dodo Payments test mode won't appear in Affonso.
