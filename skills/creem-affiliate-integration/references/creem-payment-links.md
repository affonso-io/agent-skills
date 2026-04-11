# Creem Payment Links Integration

For users using Creem Payment Links (URLs containing `creem.io/payment`).

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
  function enhanceCreemLinks() {
    if (window.affonso_referral) {
      document.querySelectorAll('a[href*="creem.io/payment"]').forEach((link) => {
        if (link.href.includes('metadata[affonso_referral]=')) return;
        const separator = link.href.includes('?') ? '&' : '?';
        link.href = link.href + separator + 'metadata[affonso_referral]=' + window.affonso_referral;
      });
    }
  }

  setTimeout(enhanceCreemLinks, 800);
  setTimeout(enhanceCreemLinks, 1500);
  setTimeout(enhanceCreemLinks, 2500);
  window.addEventListener('affonso_referral_ready', enhanceCreemLinks);
</script>
```

## How It Works

1. Visitor clicks affiliate link → Affonso sets tracking cookie
2. Enhancement script runs → Finds all Payment Links on your page
3. Adds referral tracking → Appends `metadata[affonso_referral]` to each link
4. Customer purchases → Creem receives tracking data → Affiliate gets credited

## Testing

1. Visit your site with `?atp=test` (test affiliate parameter)
2. Check browser DevTools → Application → Cookies → look for `affonso_referral`
3. Inspect your Payment Links - URLs should now include `metadata[affonso_referral]=`
4. Make a test purchase to verify tracking in your Affonso dashboard

**Note:** Use 100% discount codes for live testing without payments - Creem test mode won't appear in Affonso.
