---
name: affonso-cli
description: "Manage affiliate marketing programs via the Affonso CLI — create affiliates, track referrals, handle commissions, process payouts, and configure program settings. Use this skill when the user wants to manage their affiliate program from the terminal, automate affiliate operations, or interact with the Affonso API via CLI. Triggers on requests like 'list my affiliates', 'approve this affiliate', 'check commissions', 'process payouts', 'create a coupon', 'update program settings', or any Affonso CLI usage."
version: 1.0.0
homepage: "https://affonso.io"
metadata:
  openclaw:
    requires:
      env:
        - AFFONSO_API_KEY
      bins:
        - affonso
    install:
      - package: "@affonso/cli"
        manager: node
---

# Affonso CLI

Manage affiliate marketing programs from the command line using the `@affonso/cli` package.

## Prerequisites

1. **Install the CLI**: `npm install -g @affonso/cli`
2. **API Key**: Ask "What is your Affonso API key? Find or create one at https://affonso.io/app/affiliate-program/connect/api (starts with `sk_live_`)"
3. **Set the key**: `affonso config set api-key sk_live_YOUR_KEY` or set `AFFONSO_API_KEY` environment variable

## Authentication

Verify auth is working:
```bash
affonso whoami --json
```

Never use `affonso login` — it requires a browser and won't work in headless/agent environments.

## Rules

- **Always use `--json`** on every command — without it, the CLI outputs ANSI tables that can't be parsed.
- **Never delete without user confirmation** — all delete operations are destructive and irreversible.
- **Pagination** — referrals use cursor-based pagination (`--starting-after`, `--ending-before`); all other resources use page-based (`--page`, `--limit`).

## Commands

| Command | Description |
|---------|-------------|
| `affiliates` | List, get, create, update, delete affiliates |
| `referrals` | List, get, create, update, delete referrals (cursor-based pagination) |
| `commissions` | List, get, create, update, delete commissions |
| `payouts` | List, get, update payouts (no create/delete) |
| `coupons` | List, get, create, delete coupons (no update) |
| `clicks` | Create click events with UTM and sub-tracking |
| `embed-tokens` | Create embed tokens for partner portals |
| `program` | Get/update program settings + 8 sub-resources |
| `marketplace` | Browse/search public programs (no auth required) |
| `config` | Get/set CLI config (api-key, base-url) |
| `whoami` | Show authentication status |

For full command details and all options, see [references/COMMAND_REFERENCE.md](references/COMMAND_REFERENCE.md).
For program sub-resources (payment-terms, tracking, restrictions, fraud-rules, portal, notifications, groups, creatives), see [references/PROGRAM_SETTINGS_GUIDE.md](references/PROGRAM_SETTINGS_GUIDE.md).

## Quick Start

1. **Verify auth:** `affonso whoami --json`
2. **List resources:** `affonso affiliates list --json --limit 5`
3. **Take action:** approve, create, update, or configure as needed

For multi-step workflow recipes, see [references/WORKFLOWS.md](references/WORKFLOWS.md).
