# Polar Payment Links Integration

For users using Polar Payment Links (URLs starting with `buy.polar.sh`).

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
  function enhancePolarLinks() {
    if (window.affonso_referral) {
      document.querySelectorAll('a[href*="buy.polar.sh"]').forEach((link) => {
        if (link.href.includes('reference_id=')) return;
        const separator = link.href.includes('?') ? '&' : '?';
        link.href =
          link.href +
          separator +
          'reference_id=' +
          window.affonso_referral;
      });
    }
  }

  setTimeout(enhancePolarLinks, 800);
  setTimeout(enhancePolarLinks, 1500);
  setTimeout(enhancePolarLinks, 2500);
  window.addEventListener('affonso_referral_ready', enhancePolarLinks);
</script>
```

## How It Works

1. Visitor clicks affiliate link -> Affonso sets tracking cookie
2. Enhancement script runs -> Finds all Payment Links on your page
3. Adds referral tracking -> Appends `reference_id` to each link
4. Customer purchases -> Polar receives tracking data -> Affiliate gets credited

## Testing

1. Visit your site with `?atp=test` (test affiliate parameter)
2. Check browser DevTools -> Application -> Cookies -> look for `affonso_referral`
3. Inspect your Payment Links - URLs should now include `reference_id=`
4. Make a test purchase to verify tracking in your Affonso dashboard

**Note:** Use 100% discount codes for live testing without payments - Polar test mode won't appear in Affonso.
