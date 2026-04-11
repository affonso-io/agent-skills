# GDPR & Cookie Consent Integration

For users who need to comply with GDPR and privacy regulations using cookie consent software.

## How Consent Mode Works

When consent mode is enabled, the tracking script operates in two phases:

### Before Consent (Privacy-First)
- Detects affiliate parameters in the URL
- No cookies are set until user gives consent
- No personal data collected (no IP address, browser info, location data, etc.)

### After Consent
- Enables full affiliate tracking with user permission
- Sets the `affonso_referral` cookie for attribution
- Enables discount codes and enhanced functionality

## Installation

Add the consent-enabled tracking script to your website's `<head>` section:

```html
<!-- Standard Affonso script with consent mode enabled -->
<script
  async
  defer
  src="https://cdn.affonso.io/js/pixel.min.js"
  data-affonso="YOUR_PUBLIC_PROGRAM_ID"
  data-cookie_duration="YOUR_COOKIE_DURATION"
  data-requires-consent="true"
></script>
```

**Key attribute:** `data-requires-consent="true"` enables privacy-first mode

The script will wait for `window.affonsoConsentGranted()` before setting cookies.

## Integration with Cookie Consent Software

### Popular Cookie Consent Platforms

The consent mode works with all major cookie consent platforms:
- Cookiebot
- OneTrust
- CookieYes
- Iubenda
- Klaro
- Cookie Notice
- Termly
- Custom consent solutions

### Step-by-Step Integration

1. Install the consent-enabled script in your `<head>` tag (see code above)
2. Add consent callback to your consent platform configuration
3. Handle URL parameter propagation with custom code (see below)
4. Test the integration to ensure proper functionality

### Consent Platform Configuration

**Important:** Only add the consent callback function - NOT the entire script again.

When a user accepts cookies, your consent software should call:

```javascript
// Add this to your consent platform configuration
// DO NOT add the script tag again - it's already in your <head>
window.affonsoConsentGranted();
```

This function must be called:
- When user first accepts cookies (on any page)
- On every page load if user has already given consent
- After page navigation if consent was previously granted

### Platform-Specific Examples

#### Cookiebot

```javascript
// Add this to your consent platform - NOT a new script tag
window.addEventListener('CookiebotOnAccept', function() {
  if (Cookiebot.consent.marketing) {
    window.affonsoConsentGranted();
  }
});

// Also check on page load if consent was already given
document.addEventListener('DOMContentLoaded', function() {
  if (typeof Cookiebot !== 'undefined' && Cookiebot.consent.marketing) {
    window.affonsoConsentGranted();
  }
});
```

#### OneTrust

```javascript
// Add this to your consent platform - NOT a new script tag
function OptanonWrapper() {
  if (OnetrustActiveGroups.includes('C0004')) { // Marketing cookies
    window.affonsoConsentGranted();
  }
}

// Also check on page load
document.addEventListener('DOMContentLoaded', function() {
  if (typeof OnetrustActiveGroups !== 'undefined' && OnetrustActiveGroups.includes('C0004')) {
    window.affonsoConsentGranted();
  }
});
```

#### CookieYes

```javascript
// Add this to your consent platform - NOT a new script tag
document.addEventListener('cookieyes_consent_update', function(event) {
  if (event.detail.accepted.includes('advertisement')) {
    window.affonsoConsentGranted();
  }
});

// Also check on page load if consent exists
document.addEventListener('DOMContentLoaded', function() {
  // Check if marketing cookies are already accepted
  if (localStorage.getItem('cookieyes-consent')) {
    const consent = JSON.parse(localStorage.getItem('cookieyes-consent'));
    if (consent.advertisement) {
      window.affonsoConsentGranted();
    }
  }
});
```

#### Custom Consent Solution

```javascript
// When user clicks "Accept All" or "Accept Marketing"
function handleConsentAccepted() {
  // Your consent logic here...

  // Then notify Affonso
  window.affonsoConsentGranted();
}

// Also check on every page load if consent was already given
document.addEventListener('DOMContentLoaded', function() {
  if (hasUserGivenConsent()) { // Your consent check function
    window.affonsoConsentGranted();
  }
});
```

## URL Parameter Propagation

**Important:** URL parameter propagation does NOT happen automatically. You need to implement this with custom code.

In consent mode, the `affonso_id` parameter is added to the URL for session continuity:

### What happens automatically:
```
yoursite.com?ref=affiliate123
↓ (Affonso script adds affonso_id)
yoursite.com?ref=affiliate123&affonso_id=cm123abc456
```

### What you need to implement:
```
yoursite.com/pricing?affonso_id=cm123abc456  ← You must preserve this
yoursite.com/features?affonso_id=cm123abc456  ← when users navigate
```

**Why this matters:** If the `affonso_id` parameter is lost during navigation, attribution won't work when the user gives consent later.

### URL Propagation Implementation

You need to ensure the `affonso_id` parameter is preserved when users navigate between pages. This requires custom code on your website.

**General approach:**
1. Check if `affonso_id` exists in the current URL
2. Add it to all internal navigation links
3. Preserve it during form submissions and redirects

Implementation depends on your website technology:
- **Static websites**: Update all `<a>` tags with JavaScript
- **Single Page Applications**: Handle in your router
- **Server-side frameworks**: Add parameter to link generation

## Testing Your Integration

### 1. Test Without Consent

1. Visit your site with a test parameter: `yoursite.com?atp=test`
2. Check that no `affonso_referral` cookie is set
3. Verify that `affonso_id` is added to the URL for session continuity
4. Check your Affonso dashboard - you should see a minimal entry without personal data

### 2. Test With Consent

1. On the same page, accept cookies through your consent banner
2. Check that `affonso_referral` cookie is now set
3. In your Affonso dashboard, the entry should now show complete attribution data (IP, location, browser)

### 3. Test Navigation

1. Navigate to another page on your site
2. Verify that `affonso_id` parameter is preserved in the URL
3. If you give consent on a subpage, it should still work correctly

## Troubleshooting

### Cookie Not Set After Consent
- Ensure `window.affonsoConsentGranted()` is called after the user accepts
- Check browser DevTools Console for any JavaScript errors
- Verify the consent function is called only once per page

### Tracking Lost Between Pages
- Ensure `affonso_id` parameter is preserved in navigation for proper attribution
- Check that your SPA router doesn't strip query parameters
- Verify internal links include the `affonso_id` parameter when in consent mode

### Affiliate Attribution Not Working
- Test the complete flow: affiliate link → consent → purchase
- Check that both the script and consent integration are functioning properly
- Verify your payment provider integration includes the referral data

## API Reference

### Available Functions

```javascript
// Check if consent is required
window.Affonso.isConsentRequired(); // returns true/false

// Get current referral ID (works in both modes)
window.Affonso.getReferralId(); // returns ID or null

// Manually trigger consent (if needed)
window.affonsoConsentGranted(); // returns Promise<boolean>
```

## Legal Compliance

This consent mode helps you comply with:
- **GDPR** (General Data Protection Regulation)
- **CCPA** (California Consumer Privacy Act)
- **ePrivacy Directive** (Cookie Law)
- **LGPD** (Brazilian General Data Protection Law)

**Important:** This technical implementation helps with compliance, but you should consult with legal experts to ensure your complete setup meets all regulatory requirements.
