# Customization Reference

All available URL parameters for customizing the Affonso embedded dashboard appearance and features.

## Base URL

```
https://affonso.io/embed/referrals?token=PUBLIC_TOKEN
```

Append customization parameters with `&`:

```
https://affonso.io/embed/referrals?token=PUBLIC_TOKEN&theme=dark&bg=1a1a2e&lang=de
```

## Theme

| Parameter | Values | Default |
|-----------|--------|---------|
| `theme` | `light`, `dark`, `system` | `light` |

- `light` — Light background, dark text
- `dark` — Dark background, light text
- `system` — Matches the user's OS preference

```
&theme=dark
```

## Background Color

| Parameter | Values | Default |
|-----------|--------|---------|
| `bg` | HEX color code without `#` | Inherited from theme |

Use this to match your application's background color exactly.

```
&bg=1a1a2e
&bg=ffffff
&bg=0f172a
```

## Language

| Parameter | Values | Default |
|-----------|--------|---------|
| `lang` | Language code | `en` |

Supported languages:

| Code | Language |
|------|----------|
| `en` | English |
| `de` | German |
| `fr` | French |
| `es` | Spanish |
| `it` | Italian |
| `pt` | Portuguese |
| `nl` | Dutch |
| `pl` | Polish |
| `tr` | Turkish |
| `ja` | Japanese |
| `zh` | Chinese |
| `ko` | Korean |
| `ru` | Russian |
| `ar` | Arabic |

```
&lang=de
```

## Feature Toggles

Control which dashboard sections are visible to the partner.

| Parameter | Default | Section |
|-----------|---------|---------|
| `showHeader` | `true` | Top section with referral link and copy button |
| `showRewards` | `true` | Commission earnings and reward details |
| `showReports` | `true` | Performance charts and statistics (clicks, leads, sales) |
| `showResources` | `true` | Marketing materials and creative assets |
| `enableQRCode` | `true` | QR code generation for the referral link |
| `enableTrackingIdEdit` | `true` | Allow partner to customize their tracking ID |
| `enableSubParams` | `true` | Sub-parameter tracking (sub1-sub5) for campaign segmentation |
| `padding` | `true` | Inner padding around the dashboard content |

### Examples

**Minimal dashboard** — only referral link and performance:
```
&showRewards=false&showResources=false&enableSubParams=false
```

**Referral link only** — hide everything except the link:
```
&showRewards=false&showReports=false&showResources=false
```

**Full-featured** (default — no parameters needed):
```
// All features enabled by default
```

**No padding** — when embedding inside a container with its own padding:
```
&padding=false
```

## Brand Colors

The dashboard inherits brand colors from your **Portal Settings** in Affonso:

1. Go to https://affonso.io/app/affiliate-program/portal
2. Set **Primary Color** — used for buttons, links, and accents
3. Set **Secondary Color** — used for secondary UI elements

These colors apply automatically to all embedded dashboards. No URL parameter needed.

## Complete Example

A German-language dark mode dashboard with custom background, showing only the referral link and reports:

```
https://affonso.io/embed/referrals
  ?token=PUBLIC_TOKEN
  &theme=dark
  &bg=0f172a
  &lang=de
  &showRewards=false
  &showResources=false
  &enableSubParams=false
  &padding=false
```

## Content Security Policy (CSP)

If your application uses CSP headers, allow the iframe source:

```
frame-src https://affonso.io;
```

If you also load the tracking script:

```
script-src https://cdn.affonso.io;
frame-src https://affonso.io;
```
