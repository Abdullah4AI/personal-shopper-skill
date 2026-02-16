# Personal Shopper — Diamond Search Skill

Turn any AI agent into a smart shopping research team. 7 specialized agents across 3 layers find the best products at the best prices.

## What It Does

Uses the **Diamond Search** methodology: starts with your request, expands to 7 agents searching from different angles and using all available search tools, then converges to one clear recommendation.

### The 7 Agents

**Search Layer (parallel)**
1. **Mainstream Research** — Reddit, YouTube, Wirecutter, RTINGS
2. **Anti-Bias Research** — Breaks the echo chamber, finds hidden alternatives
3. **Local Market Scanner** — Real prices and availability on local platforms
4. **Niche Community Diver** — Expert opinions from specialized forums

**Expertise Layer (parallel)**
5. **Domain Expert** — Reviews all findings, asks "is this even the right product?"
6. **Latest Tech Tracker** — Checks for newer models, discontinued products, upcoming launches

**Price Layer (sequential)**
7. **Price & Deal Hunter** — Coupons, cashback, installments, cross-platform price comparison

### Key Features

- **Multi-Tool Search:** Automatically detects and uses all available search tools (Camoufox, Exa, web_search, web_fetch, browser) for maximum coverage
- **Nested Sub-Agents:** Search agents can spawn their own sub-agents for deep parallel research on complex products
- **Anti-Bias Search:** Expands options beyond the same brands everyone recommends
- **Expert Validation:** Not just "best rated" but "best for YOUR specific use case"
- **Local Market Awareness:** Saudi market pricing patterns, platform comparison, reseller detection
- **Price Intelligence:** Coupons, trade-in programs, installment options, cashback deals
- **Multi-Language Output:** Research runs in English internally; output delivered in the user's preferred language
- **Honest Recommendations:** If there's no real alternative, it says so

### Design Principles (from [Skills + Shell Tips](https://developers.openai.com/blog/skills-shell-tips))

- **Description as routing logic:** Clear USE WHEN / DON'T USE WHEN / EDGE CASES in the skill description for accurate triggering
- **Negative examples:** Explicit "don't call this when..." cases to prevent misfires
- **Templates inside the skill:** Output templates and worked examples loaded only when needed, keeping other prompts lean
- **Designed for long runs:** Sub-agent pattern with clear convergence points handles large research tasks without context overflow

## How It Works

1. Asks what you need, your budget, how you'll use it, and your preferred language
2. Detects available search tools in the environment
3. Spawns 4 search agents in parallel (each assigned optimal tools)
4. Spawns 2 expertise agents in parallel (domain expert + latest tech)
5. Merges and filters all results using convergence rules
6. Runs price optimization with deal hunting
7. Delivers a clear recommendation with alternatives in your language

## Installation

### Universal (Agent Skills standard)

```bash
npx skills add Abdullah4AI/personal-shopper-skill
```

### OpenClaw

```bash
clawhub install personal-shopper
```

Or manually:

```bash
git clone https://github.com/Abdullah4AI/personal-shopper-skill.git
cp -r personal-shopper-skill ~/.openclaw/skills/personal-shopper
```

### Claude Code

```bash
git clone https://github.com/Abdullah4AI/personal-shopper-skill.git
cp -r personal-shopper-skill ~/.claude/skills/personal-shopper
```

### Codex CLI

```bash
git clone https://github.com/Abdullah4AI/personal-shopper-skill.git
cp -r personal-shopper-skill codex/skills/personal-shopper
```

### Antigravity

```bash
git clone https://github.com/Abdullah4AI/personal-shopper-skill.git
cp -r personal-shopper-skill agent/skills/personal-shopper
```

### Augment

```bash
git clone https://github.com/Abdullah4AI/personal-shopper-skill.git
cp -r personal-shopper-skill ~/.augment/skills/personal-shopper
```

### Cline

```bash
git clone https://github.com/Abdullah4AI/personal-shopper-skill.git
cp -r personal-shopper-skill cline/skills/personal-shopper
```

### CodeBuddy

```bash
git clone https://github.com/Abdullah4AI/personal-shopper-skill.git
cp -r personal-shopper-skill ~/.codebuddy/skills/personal-shopper
```

### Command Code

```bash
git clone https://github.com/Abdullah4AI/personal-shopper-skill.git
cp -r personal-shopper-skill ~/.commandcode/skills/personal-shopper
```

## Example Usage

> "I need a 4K monitor for coding, budget $400"

> "Find me the best noise-cancelling headphones under $300"

> "Compare MacBook Air vs Pro for software development"

> "ابحث لي عن أفضل مايك للبث المباشر، ميزانيتي 500 ريال"

> "وش أفضل لابتوب للبرمجة بميزانية 5000 ريال"

## Philosophy

> The product is anchored on VALUE, not on BRAND.
> We expand the search horizon so decisions are based on value, not just what shows up first on Google.

## Credits

Diamond Search methodology by [NOLai](https://x.com/nolaiai) — February 2026

## License

MIT
