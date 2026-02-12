# Personal Shopper - Diamond Search Skill

Turn any AI agent into a smart shopping research team with 7 specialized agents in 3 layers that find the best products at the best prices.

## What It Does

Uses the **Diamond Search** methodology: starts with your request, expands to 7 agents searching from different angles, then converges to one clear recommendation.

### The 7 Agents

**Search Layer (parallel)**
1. **Mainstream Research** - Reddit, YouTube, Wirecutter, RTINGS
2. **Anti-Bias Research** - Breaks the echo chamber, finds hidden alternatives
3. **Local Market Scanner** - Real prices and availability on local platforms
4. **Niche Community Diver** - Expert opinions from specialized forums

**Expertise Layer (parallel)**
5. **Domain Expert** - Reviews all findings, asks "is this even the right product?"
6. **Latest Tech Tracker** - Checks for newer models, discontinued products, upcoming launches

**Price Layer (sequential)**
7. **Price & Deal Hunter** - Coupons, cashback, installments, cross-platform price comparison

### Key Features

- **Anti-Bias Search**: Expands your options beyond the same 3 brands everyone recommends
- **Expert Validation**: Not just "best rated" but "best for YOUR specific use case"
- **Local Market Awareness**: Saudi market pricing patterns, platform comparison, reseller detection
- **Price Intelligence**: Coupons, trade-in programs, installment options, cashback deals
- **Honest Recommendations**: If there's no real alternative, it says so

## How It Works

1. Asks what you need, your budget, and how you'll use it
2. Spawns 4 search agents in parallel (different search strategies)
3. Spawns 2 expertise agents in parallel (domain expert + latest tech)
4. Merges and filters all results
5. Runs price optimization
6. Delivers a clear recommendation with alternatives

## Installation

### Universal (Works with all platforms)

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

> "ابحث لي عن أفضل مايك للبث المباشر، ميزانيتي 500 ريال"

> "I need a 4K monitor for coding, budget $400"

> "قارن لي بين MacBook Air و Pro للبرمجة"

## Philosophy

> المنتج يرتكز على القيمة، مو على البراند
>
> "We expand the search horizon so decisions are based on value, not just what shows up first on Google."

## Credits

Diamond Search methodology by [NOLai](https://x.com/nolaiai) - February 2026

## License

MIT
