# Affonso Agent Skills

AI agent skills for setting up and managing [Affonso](https://affonso.io) affiliate programs. Works with Claude Code, Cursor, Cline, OpenCode, Codex, Gemini CLI, and 40+ other coding agents.

## Installation

### Option 1: npx (Recommended)

```bash
# Install all skills
npx skills add affonso-io/agent-skills

# Install a specific skill
npx skills add affonso-io/agent-skills --skill affonso-setup

# List available skills
npx skills add affonso-io/agent-skills --list
```

### Option 2: Claude Code Plugin

```bash
# Add the marketplace
/plugin marketplace add affonso-io/agent-skills

# Install the plugin
/plugin install affiliate-skills@affiliate-skills
```

### Option 3: Direct Link (No Installation)

Give any AI agent the raw GitHub URL to follow the instructions directly:

```
Load and implement the instructions from:
https://raw.githubusercontent.com/affonso-io/agent-skills/main/skills/stripe-affiliate-integration/SKILL.md
Replace YOUR_PUBLIC_PROGRAM_ID with: <your-program-id>
```

## Available Skills

### `/affonso-setup` — Auto-Detect & Install

The main entry point. Automatically scans your codebase to detect which payment provider and integration method you use, then performs the full setup.

```
"Set up Affonso affiliate tracking in my project"
```

### Payment Provider Integrations

Each skill installs the Affonso tracking script, handles GTM and GDPR/cookie consent, and passes referral data to your payment provider.

| Skill | Provider | Methods |
|-------|----------|---------|
| `/stripe-affiliate-integration` | Stripe | Checkout API, Payment Links, Buy Button, Pricing Tables |
| `/polar-affiliate-integration` | Polar | Checkout API, Payment Links |
| `/dodo-affiliate-integration` | Dodo Payments | Checkout API, Payment Links |
| `/creem-affiliate-integration` | Creem | Checkout API, Payment Links |
| `/paddle-affiliate-integration` | Paddle | Billing Checkout |
| `/custom-affiliate-integration` | Any provider | REST API (Lemon Squeezy, Gumroad, Razorpay, Paystack, etc.) |

### `/affonso-cli` — Manage Your Program

Use the Affonso CLI to manage affiliates, referrals, commissions, payouts, coupons, and program settings directly from the terminal.

```
"List my pending affiliates"
"Approve affiliate aff_123"
"Show commissions for this month"
```

### `/embedded-affiliate-dashboard`

Embed Affonso's white-label affiliate dashboard into your web app via iframe. Includes framework examples for Next.js, Express, FastAPI, Rails, and Laravel.

```
"Add an embedded affiliate dashboard to my app"
```

## Repository Structure

```
agent-skills/
├── .claude-plugin/marketplace.json
├── skills/
│   ├── affonso-affiliate-program-setup/   # Auto-detect & full setup
│   ├── affonso-affiliate-program-cli/    # CLI management (works with Claude Code + OpenClaw)
│   ├── stripe-affiliate-integration/     # Stripe
│   ├── polar-affiliate-integration/      # Polar
│   ├── dodo-affiliate-integration/       # Dodo Payments
│   ├── creem-affiliate-integration/      # Creem
│   ├── paddle-affiliate-integration/     # Paddle
│   ├── custom-affiliate-integration/     # Any payment provider
│   └── embedded-affiliate-dashboard/     # Embed dashboard
```

## Support

- Affonso: https://affonso.io
- Docs: https://docs.affonso.io
- Help: https://affonso.io/help
- Email: silvestro@affonso.io
