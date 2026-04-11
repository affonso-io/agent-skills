# Google Tag Manager Integration

For users who manage scripts through Google Tag Manager instead of directly in their website code.

## Why Use GTM?

Google Tag Manager allows you to deploy tracking scripts without modifying your website's code directly. This is useful for teams that want to manage tracking through GTM's interface.

## Installation Steps

### Step 1: Create a New Tag

1. Log in to your Google Tag Manager account
2. Select your container
3. Click **Tags** in the left sidebar
4. Click **New** to create a new tag

### Step 2: Configure the Tag

1. Click **Tag Configuration**
2. Select **Custom HTML** as the tag type
3. Paste the following code:

```html
<script>
  (function() {
    var script = document.createElement('script');
    script.defer = true;
    script.setAttribute('data-affonso', 'YOUR_PUBLIC_PROGRAM_ID');
    script.setAttribute('data-cookie_duration', 'YOUR_COOKIE_DURATION');
    script.src = 'https://cdn.affonso.io/js/pixel.min.js';
    document.head.appendChild(script);
  })();
</script>
```

4. Replace `YOUR_PUBLIC_PROGRAM_ID` with the user's program ID
5. Replace `YOUR_COOKIE_DURATION` with the cookie duration (e.g., `30` for 30 days)

**Why this script format?** GTM requires dynamically created scripts instead of static script tags. This code creates the Affonso tracking script programmatically, which works correctly within GTM's Custom HTML tag.

### Step 3: Set the Trigger

1. Click **Triggering**
2. Select **All Pages** to fire the tag on every page
3. Alternatively, create a custom trigger if tracking is only needed on specific pages

### Step 4: Save and Publish

1. Click **Save** to save the tag
2. Click **Submit** in the top right corner
3. Add a version name (e.g., "Added Affonso affiliate tracking")
4. Click **Publish**

## GDPR/Cookie Consent with GTM

If GDPR compliance is needed, modify the script to include the consent attribute:

```html
<script>
  (function() {
    var script = document.createElement('script');
    script.defer = true;
    script.setAttribute('data-affonso', 'YOUR_PUBLIC_PROGRAM_ID');
    script.setAttribute('data-cookie_duration', 'YOUR_COOKIE_DURATION');
    script.setAttribute('data-requires-consent', 'true');
    script.src = 'https://cdn.affonso.io/js/pixel.min.js';
    document.head.appendChild(script);
  })();
</script>
```

Then see the GDPR consent reference guide for setting up consent-based tracking.

### Using GTM's Consent Mode

If using GTM's built-in Consent Mode:

1. Go to **Triggers** and create a new trigger
2. Select **Consent Initialization - All Pages** or create a custom trigger based on your consent management platform
3. Assign this trigger to your Affonso tag instead of "All Pages"

## Testing Your Integration

### Using GTM Preview Mode

1. In GTM, click **Preview** in the top right corner
2. Enter your website URL
3. Navigate through your site and verify the Affonso tag fires
4. Check that the tag appears under "Tags Fired" on each page

### Verifying Affonso Tracking

1. Visit your website with `?atp=test` appended to the URL
2. Open browser DevTools (F12)
3. Go to **Console** and type `window.affonso_referral`
4. You should see the test referral data
5. Verify the tracking appears in your Affonso dashboard

## Troubleshooting

### Tag Not Firing
- Check that the trigger is set correctly (All Pages)
- Verify the container is published, not just saved
- Use GTM Preview mode to debug

### Script Errors
- Ensure all quotes are straight quotes (`'` or `"`), not curly quotes
- Check that `YOUR_PUBLIC_PROGRAM_ID` is replaced with your actual program ID
- Verify there are no extra spaces or line breaks in the code

### Tracking Not Working
- Check browser console for JavaScript errors
- Verify the Affonso script is loading in the Network tab
- Ensure your domain is correctly configured in Affonso dashboard
