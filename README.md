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

### Option 2: Cursor Plugin

After the plugin is listed in the Cursor Marketplace:

1. Open **Customize** in Cursor.
2. Search for **Affonso**, select **Install**, and choose user or project scope.
3. Enter an Affonso API key when Cursor asks for it. Create a restricted key in **Affonso → Settings → API Keys** with only the scopes your workflow needs.
4. Enable the **Affonso MCP** server from Customize and ask Cursor to use an Affonso skill or manage your program.

The plugin installs all Affonso skills and connects Cursor to the hosted MCP server at `https://api.affonso.io/mcp`. Your API key is stored by Cursor as plugin configuration and is never committed to this repository.

To test an unpublished checkout locally, symlink this repository into `~/.cursor/plugins/local/affonso-affiliate`, then run **Developer: Reload Window** in Cursor.

### Option 3: Claude Code Plugin

The `affonso` plugin connects to the hosted MCP server through OAuth and provides read-only program reports, partner reviews, and payout reviews. The `affiliate-skills` plugin contains the existing CLI and integration skills.

```bash
# Add the marketplace
/plugin marketplace add affonso-io/agent-skills

# Install the MCP plugin
/plugin install affonso@affiliate-skills

# Or install the CLI and integration skills
/plugin install affiliate-skills@affiliate-skills
```

After installing `affonso`, open `/mcp` and authenticate its Affonso connection. See the [Affonso plugin guide](./plugins/affonso/README.md) for requirements, example prompts, and data handling. A listing in Anthropic's directory requires a separate review.

### Option 4: Direct Link (No Installation)

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

### AI Affiliate Manager

Operational skills for running a healthy affiliate program. They use the Affonso CLI for program data and always require explicit approval before changing payouts, commissions, partner status, groups, coupons, or program settings.

| Skill | Focus |
|-------|-------|
| `/affiliate-fraud-review` | Evidence-based conversion and pending-payout screening |
| `/affiliate-partner-qualification` | Applicant fit, policy, and risk assessment |
| `/affiliate-performance-manager` | Performance analysis and prioritized account actions |
| `/affiliate-profitability-manager` | Commission, margin, LTV, CAC, and payback decisions |
| `/affiliate-partner-activation` | New-partner activation and dormant-partner re-engagement |
| `/affiliate-campaign-manager` | Campaign planning, execution kit, and retrospective |
| `/affiliate-compliance-monitor` | Brand, traffic, coupon, and disclosure compliance |

```
"Review open payouts for fraud risk before this payout cycle"
"Which pending affiliate applications should I approve?"
"Create an activation plan for our dormant high-potential partners"
"Can we increase this affiliate's commission and still hit our contribution target?"
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
├── .cursor-plugin/plugin.json             # Cursor Marketplace manifest
├── assets/logo_icon_affonso.png            # Cursor Marketplace logo
├── mcp.json                                # Hosted Affonso MCP configuration
├── skills/
│   ├── affonso-setup/                    # Auto-detect & full setup
│   ├── affonso-cli/                      # CLI management (works with Claude Code + OpenClaw)
│   ├── affiliate-manager/                 # AI affiliate manager suite
│   │   ├── affiliate-fraud-review/        # Payout and conversion fraud triage
│   │   ├── affiliate-partner-qualification/ # Applicant assessment
│   │   ├── affiliate-performance-manager/   # Partner performance operations
│   │   ├── affiliate-profitability-manager/ # Unit economics and commission decisions
│   │   ├── affiliate-partner-activation/  # Activation and re-engagement
│   │   ├── affiliate-campaign-manager/    # Campaign operations
│   │   └── affiliate-compliance-monitor/  # Compliance remediation
│   ├── stripe-affiliate-integration/     # Stripe
│   ├── polar-affiliate-integration/      # Polar
│   ├── dodo-affiliate-integration/       # Dodo Payments
│   ├── creem-affiliate-integration/      # Creem
│   ├── paddle-affiliate-integration/     # Paddle
│   ├── custom-affiliate-integration/     # Any payment provider
│   └── embedded-affiliate-dashboard/     # Embed dashboard
```

## Cursor MCP

The Cursor plugin configures the remote Affonso MCP server. It gives Cursor access to your Affonso affiliates, referrals, commissions, and payouts, subject to the scopes of the API key you provide. Cursor asks for approval before running MCP tools by default; review any write or destructive action before approving it.

For a narrower setup without the plugin, add this to Cursor's `mcp.json` and set `AFFONSO_API_KEY` in your environment:

```json
{
  "mcpServers": {
    "affonso": {
      "url": "https://api.affonso.io/mcp",
      "headers": {
        "Authorization": "Bearer ${env:AFFONSO_API_KEY}"
      }
    }
  }
}
```

## License

[MIT](./LICENSE)

## Support

- Affonso: https://affonso.io
- Docs: https://docs.affonso.io
- Help: https://affonso.io/help
- Email: silvestro@affonso.io
