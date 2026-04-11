# Affiliate Skills

A collection of Claude Code skills to simplify affiliate program management and integration.

## Available Skills

### stripe-affiliate-integration

Set up Affonso affiliate program tracking with Stripe integration.

**Features:**
- Tracking script installation (direct or via Google Tag Manager)
- GDPR/cookie consent compliance
- Stripe integration (Checkout API, Payment Links, Buy Buttons, Pricing Tables)
- Optional signup tracking
- Framework-specific code examples

**Use cases:**
- Setting up a new affiliate program with Stripe
- Integrating Affonso tracking into existing Stripe implementations
- Adding GDPR-compliant affiliate tracking
- Implementing signup conversion tracking

## Installation

### Option 1: CLI Install (Recommended)
Use add-skill to install skills directly:

```bash
# Install all skills
npx add-skill zilvestro/affiliate-skills

# Install specific skills
npx add-skill zilvestro/affiliate-skills --skill stripe-affiliate-integration

# List available skills
npx add-skill zilvestro/affiliate-skills --list
```

This automatically installs to your `.claude/skills/` directory.

### Option 2: Claude Code Plugin
Install via Claude Code's built-in plugin system:

```bash
# Add the marketplace
/plugin marketplace add zilvestro/affiliate-skills

# Install all affiliate skills
/plugin install affiliate-skills
```

### Option 3: Clone and Copy
Clone the entire repo and copy the skills folder:

```bash
git clone https://github.com/zilvestro/affiliate-skills.git
cp -r affiliate-skills/skills/* .claude/skills/
```

### Option 4: Git Submodule
Add as a submodule for easy updates:

```bash
git submodule add https://github.com/zilvestro/affiliate-skills.git .claude/affiliate-skills
```

Then reference skills from `.claude/affiliate-skills/skills/`.

### Option 5: Fork and Customize
1. Fork this repository
2. Customize skills for your specific needs
3. Clone your fork into your projects

### Using Skills

Once installed, trigger the skill by asking Claude to help with related tasks:

```
"Set up my Stripe affiliate program"
"Integrate Affonso tracking with my Stripe checkout"
"Add affiliate tracking to my website"
```

Claude will automatically recognize the task and use the appropriate skill.

## Repository Structure

```
affiliate-skills/
├── README.md                          # This file
├── marketplace.json                   # Skill registry for npx installation
└── skills/                            # Source code for each skill
    └── stripe-affiliate-integration/
        ├── SKILL.md                   # Main skill instructions
        └── references/                # Detailed integration guides
            ├── stripe-checkout-api.md
            ├── stripe-payment-links.md
            ├── stripe-buy-button.md
            ├── stripe-pricing-tables.md
            ├── gtm-integration.md
            └── gdpr-consent.md
```

## Development

### Creating New Skills

Use the skill-creator tool to develop new skills:

```bash
python3 .agents/skills/skill-creator/scripts/init_skill.py <skill-name> --path ./skills
```

### Validating Skills

Validate your skill before committing:

```bash
python3 .agents/skills/skill-creator/scripts/quick_validate.py ./skills/<skill-name>
```

## Contributing

When adding new skills:

1. Create the skill in `skills/<skill-name>/`
2. Follow the skill creation best practices from skill-creator
3. Add the skill to `marketplace.json`
4. Update this README with the new skill information
5. Validate the skill before committing

## License

[Add your license information here]

## Support

For issues or questions:
- Affonso platform: https://affonso.io
- Affonso help: https://affonso.io/help
