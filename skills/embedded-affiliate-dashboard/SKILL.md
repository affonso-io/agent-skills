---
name: embedded-affiliate-dashboard
description: "Embed Affonso's white-label affiliate dashboard into any web application. Use this skill when the user wants to add an embedded referral or affiliate portal to their product, integrate an affiliate dashboard via iframe, generate embed tokens, or customize the partner dashboard appearance. Triggers on requests like 'embed affiliate dashboard', 'add referral portal to my app', 'integrate Affonso dashboard', 'white-label affiliate portal', or 'embedded partner dashboard'."
---

# Embedded Affiliate Dashboard

This skill guides you through embedding Affonso's affiliate dashboard into your web application, giving your partners a white-label referral experience directly inside your product.

## Overview

Affonso's embedded dashboard lets you integrate a fully functional affiliate portal via iframe. Partners can:

1. **Access referral links** - Copy unique tracking links and QR codes
2. **Track performance** - View clicks, leads, sales, and commissions
3. **Monitor payouts** - Check earnings and payout status
4. **Use coupons** - View and create promotional discount codes
5. **Access resources** - Download marketing materials and creatives

## Prerequisites & Information Gathering

Before beginning implementation, gather the following information by asking the user:

### Required Information

1. **API Key**: Ask "What is your Affonso API key? You can find or create one in your Affonso dashboard (https://affonso.io/app/affiliate-program/connect/api). It starts with `sk_live_`."

2. **Program ID**: Ask "What is your Affonso program ID? You can find it at https://affonso.io/app/affiliate-program"

3. **Framework**: Ask "What backend framework does your application use?" Common options:
   - Next.js (App Router or Pages Router)
   - Express / Node.js
   - Python (Django / Flask / FastAPI)
   - Ruby on Rails
   - PHP (Laravel)
   - Go

4. **User Identification**: Ask "How do you identify users in your app? We need the logged-in user's email to generate the embed token."

### Optional Customization

5. **Theme**: Ask "Do you want the dashboard in light mode, dark mode, or matching the user's system preference?"

6. **Language**: Ask "What language should the dashboard display? Supported: English, German, French, Spanish, Italian, Portuguese, Dutch, Polish, Turkish, Japanese, Chinese, Korean, Russian, Arabic."

7. **Feature Toggles**: Ask "Do you want to hide any dashboard sections?" Options:
   - Header with referral link
   - Rewards / commissions section
   - Performance reports
   - Marketing resources
   - QR code generation
   - Tracking ID editing
   - Sub-parameter tracking

8. **Commission Tiers**: Ask "Do you use different commission groups/tiers? If so, which group should new partners be assigned to?"

## Workflow

```
1. Generate Embed Token (Server-Side)
   └─ POST /v1/embed/token with partner email

2. Embed Dashboard (Client-Side)
   └─ Render iframe with token + customization params

3. Customize Appearance
   ├─ Theme & background color
   ├─ Language
   └─ Feature toggles

4. Handle Token Refresh
   └─ Tokens expire after 30 minutes — regenerate on page load

5. Style the iframe
   └─ Framework-specific examples
```

## Step 1: Generate Embed Token (Server-Side)

The embed token authenticates a partner and must always be generated server-side. **Never expose your API key to the client.**

Read the [Token Generation Guide](references/token-generation.md) for framework-specific examples.

### API Request

```
POST https://api.affonso.io/v1/embed/token

Headers:
  Authorization: Bearer sk_live_YOUR_API_KEY
  Content-Type: application/json

Body:
{
  "programId": "YOUR_PROGRAM_ID",
  "partner": {
    "email": "partner@example.com",
    "name": "Partner Name",        // optional
    "image": "https://..."         // optional avatar URL
  },
  "groupId": "GROUP_ID"            // optional commission tier
}
```

### API Response

```json
{
  "success": true,
  "data": {
    "publicToken": "eyJhbGci...",
    "expiresAt": "2025-01-01T01:00:00.000Z",
    "link": "https://yoursite.com?atp=TRACKING_ID",
    "portalUrl": "https://affiliate.yoursite.com",
    "partnershipStatus": "APPROVED"
  }
}
```

**Key fields:**
- `publicToken` — Use this in the iframe URL (valid for 30 minutes)
- `partnershipStatus` — `APPROVED` or `PENDING` (show appropriate UI)
- `link` — The partner's referral link

## Step 2: Embed the Dashboard

Render an iframe using the `publicToken`:

```html
<iframe
  src="https://affonso.io/embed/referrals?token=PUBLIC_TOKEN"
  style="width: 100%; height: 100%; min-height: 600px; border: none;"
  allow="clipboard-write"
  title="Affiliate Dashboard"
></iframe>
```

**Important:** The `allow="clipboard-write"` attribute is required so partners can copy their referral link.

## Step 3: Customize the Dashboard

Add URL parameters to customize appearance and features. Read the [Customization Reference](references/customization.md) for all options.

### Quick Customization Example

```
https://affonso.io/embed/referrals
  ?token=PUBLIC_TOKEN
  &theme=dark
  &bg=1a1a2e
  &lang=de
  &showResources=false
  &enableQRCode=true
```

### Available Parameters

| Parameter | Values | Default | Description |
|-----------|--------|---------|-------------|
| `theme` | `light`, `dark`, `system` | `light` | Color theme |
| `bg` | HEX without `#` (e.g. `1a1a2e`) | — | Background color |
| `lang` | `en`, `de`, `fr`, `es`, `it`, `pt`, `nl`, `pl`, `tr`, `ja`, `zh`, `ko`, `ru`, `ar` | `en` | Dashboard language |
| `showHeader` | `true` / `false` | `true` | Referral link section |
| `showRewards` | `true` / `false` | `true` | Commissions section |
| `showReports` | `true` / `false` | `true` | Performance reports |
| `showResources` | `true` / `false` | `true` | Marketing resources |
| `enableQRCode` | `true` / `false` | `true` | QR code generation |
| `enableTrackingIdEdit` | `true` / `false` | `true` | Tracking ID editing |
| `enableSubParams` | `true` / `false` | `true` | Sub-parameter tracking |
| `padding` | `true` / `false` | `true` | Inner padding |

### Branding

The dashboard inherits your primary and secondary brand colors from **Portal Settings** in your Affonso dashboard. Configure those at https://affonso.io/app/affiliate-program/portal.

## Step 4: Handle Token Refresh

Tokens expire after **30 minutes**. Generate a fresh token on every page load — do not cache tokens.

```
User loads page → Server generates token → Client renders iframe
```

If the user navigates away and returns, generate a new token. Old tokens are automatically invalidated.

## Step 5: Framework Integration

Read the [Framework Examples](references/framework-examples.md) for complete implementation examples in:

- Next.js (App Router)
- Next.js (Pages Router)
- Express / Node.js
- Python (FastAPI)
- Ruby on Rails
- PHP (Laravel)

## Testing & Verification

1. **Token generation**: Call the API and verify you receive a `publicToken`
2. **iframe rendering**: Load the iframe and confirm the dashboard appears
3. **Partnership status**: Check `partnershipStatus` in the response — new partners may show `PENDING` until approved (or auto-approved if enabled)
4. **Clipboard**: Click the referral link copy button — verify it works (requires `allow="clipboard-write"`)
5. **Customization**: Test theme, language, and feature toggle parameters
6. **Token expiry**: Verify the dashboard still works after page refresh (new token generated)

## Troubleshooting

### Dashboard Not Loading
- Verify the `publicToken` is valid and not expired (30-minute window)
- Check that the iframe `src` URL is correct
- Ensure no Content Security Policy (CSP) headers block `affonso.io`

### Clipboard Not Working
- The iframe must have `allow="clipboard-write"` attribute
- The page must be served over HTTPS

### Partner Shows as PENDING
- Check if auto-approve is enabled in your program settings
- Manually approve the partner in the Affonso dashboard
- Or use the CLI: `affonso affiliates update <id> --status approved`

### Styling Issues
- Use `padding=false` if embedding in a container that already has padding
- Set `bg` to match your app's background color
- Configure brand colors in Portal Settings for consistent theming
