---
name: personal-shopper
description: |
  Smart product research assistant using the Diamond Search methodology (7 specialized agents across 3 layers).
  USE WHEN: user wants a product recommendation, shopping advice, "what's the best X", "compare X vs Y",
  "find me a good", product comparison, buying decision help, "is this a good deal", "should I buy X or Y",
  "أبغى أشتري", "وش أفضل منتج", "قارن لي", "ابحث لي عن", "مقارنة منتجات".
  DON'T USE WHEN: user wants price tracking over time, order placement, returns/refunds help,
  market analysis for business entry (use mckinsey-research), general web search not about purchasing,
  reviewing or troubleshooting a product they already own, comparing companies as businesses.
  EDGE CASES: "what's the best laptop" → this skill. "what's the laptop market size" → mckinsey-research.
  "compare two products by specs" → this skill. "compare two companies competitively" → mckinsey-research.
  "is this a good deal on Amazon" → this skill. "analyze the deals market" → mckinsey-research.
  "أبغى أشتري لابتوب" → this skill. "أبغى أفتح متجر لابتوبات" → mckinsey-research.
  Output: Clear product recommendation with best price, availability, alternatives, and expert validation.
  Success: User gets an actionable buying decision backed by multi-source research, expert analysis, and real pricing.
  Inputs: Product type, budget (optional), use case, preferences.
  Tools involved: sessions_spawn (sub-agents), web_search/camofox/exa (search), web_fetch (page extraction).
---

# Personal Shopper — Diamond Search

## Overview

Diamond Search is a multi-agent product research methodology. It starts narrow (user's request), expands wide (7 agents searching from different angles), then converges to one clear recommendation.

```
         [BRIEF]
            |
    --------+--------
    |   |    |   |
   [1] [2]  [3] [4]     ← Search Layer (parallel)
    |   |    |   |
    --------+--------
            |
        [5]   [6]        ← Expertise Layer (parallel)
            |
      [CONVERGENCE]
            |
           [7]            ← Price Layer
            |
        [OUTPUT]
```

## Golden Product Criteria

Every recommendation is evaluated against these 5 criteria:

| # | Criterion | Description |
|---|-----------|-------------|
| 1 | Performance | Delivers what's needed for the specific use case |
| 2 | Value | Price justified by actual (not theoretical) performance |
| 3 | Availability | In stock locally with warranty and service |
| 4 | Reliability | Genuine positive reviews from real long-term users |
| 5 | Timing | Not about to be replaced by a new generation or discontinued |

## The 7 Agents

| # | Agent | Role | Layer |
|---|-------|------|-------|
| 1 | Mainstream Research | Top sources: Reddit, YouTube, Wirecutter, RTINGS | Search |
| 2 | Anti-Bias Research | Reverse search, alternative brands, breaks echo chambers | Search |
| 3 | Local Market Scanner | Saudi platforms: Amazon.sa, noon, jarir, extra | Search |
| 4 | Niche Community Diver | Specialized forums, Facebook groups, Discord, small subreddits | Search |
| 5 | Domain Expert | Judges results with technical expertise (does NOT search) | Expertise |
| 6 | Latest Tech Tracker | New launches, upcoming models, discontinuations | Expertise |
| 7 | Price & Deal Hunter | Coupons, cashback, installments, cross-platform price comparison | Price |

> For full agent prompts and methodology details: read `references/diamond-methodology.md`

## Tool Detection & Assignment

Before spawning search agents, detect which search tools are available in the current environment. Assign tools to agents for maximum coverage.

### Available Tool Categories

| Tool | Best For | Detection |
|------|----------|-----------|
| `camofox_*` (Camoufox) | Retailer sites, Amazon, Google Shopping — bypasses bot detection | Check if `camofox_create_tab` is available |
| `web_search` | Quick broad web search (Brave API) | Check if `web_search` is available |
| `web_fetch` | Extracting content from specific URLs | Check if `web_fetch` is available |
| `browser` | General browser automation | Check if `browser` is available |
| Exa (via MCP/mcporter) | AI-powered semantic search, great for finding expert content | Check if `mcporter` or exa MCP tool is available |

### Tool Assignment Strategy

Distribute tools across agents to avoid redundancy and maximize coverage:

- **Agent 1 (Mainstream):** `web_search` for broad queries + `web_fetch` for review sites + `camofox` for YouTube/Reddit
- **Agent 2 (Anti-Bias):** `web_search` with reverse queries + Exa for semantic discovery + `camofox` for niche sites
- **Agent 3 (Local Market):** `camofox` for Saudi retailer sites (Amazon.sa, noon, jarir, extra) — best for bot-protected stores. Fallback: `web_fetch`
- **Agent 4 (Niche):** Exa for forum/community search + `camofox` for Discord/Facebook + `web_search` for subreddits
- **Agent 6 (Latest Tech):** `web_search` for recent launches + `web_fetch` for tech news sites
- **Agent 7 (Price):** `camofox` for live pricing on retailer sites + `web_search` for coupon codes

If a tool is not available, the agent falls back to the next best option. Every agent can use `web_search` + `web_fetch` as the universal baseline.

**Include this tool availability context in every sub-agent prompt:**

```
Available search tools: {list all available tools}
Preferred tools for your role: {assigned tools}
Fallback: web_search + web_fetch
```

## Nested Sub-Agent Pattern

For complex products with many competing options, search-layer agents (1-4) may spawn their own sub-agents to parallelize across sources. This is the **nested sub-agent** pattern.

**When to use nested sub-agents:**
- Product category has 10+ viable options (e.g., laptops, headphones, monitors)
- Multiple distinct source types need deep scraping (e.g., Reddit threads + YouTube reviews + Wirecutter articles)
- Local market requires checking 4+ retailer sites with live pricing

**Example:** Agent 1 (Mainstream) might spawn:
- Sub-agent 1a: Search Reddit for "{product} recommendations" via `camofox` or `web_search`
- Sub-agent 1b: Search YouTube reviews via `camofox` or `web_search`
- Sub-agent 1c: Check Wirecutter/RTINGS via `web_fetch`

**When NOT to nest:** Simple products (cables, basic accessories) or when only 2-3 options exist. Over-parallelizing wastes resources.

The orchestrating agent decides based on product complexity whether to enable nesting or run flat.

## Workflow (5 Phases)

### Phase 1: BRIEF (sequential)

Gather from the user:
- What product they need
- Budget (or "open/flexible")
- Primary use case
- Any preferences or hard requirements
- Whether they have initial options in mind
- **Preferred output language** (default: match the language the user wrote in)

If the user hasn't clarified something important, ask. Do not assume.

**Language handling:**
- All internal agent work and analysis runs in English for consistency
- The final output (Phase 5) is delivered in the user's preferred language
- If the user writes in Arabic, output in Arabic. If English, output in English. If they specify a language, use that.

### Phase 2a: Search Layer (parallel — 4 sub-agents)

Spawn 4 sub-agents in parallel using `sessions_spawn`. Each agent gets:
1. Team context (its role within the 7-agent team)
2. Product details from the brief
3. Available tools and preferred tools for its role
4. Clear output format requirements

```python
# Spawn all 4 search agents in parallel
sessions_spawn(
  task="""You are Agent 1 (Mainstream Research) in a 7-agent product research team. 
Your teammates cover other angles — focus strictly on YOUR role.

Product: {product}
Budget: {budget}
Use case: {use_case}
User preferences: {preferences}

YOUR ROLE: Search mainstream, well-known sources for the best options.
Sources: Reddit, YouTube (detailed reviews), Wirecutter, RTINGS, Tom's Guide.

Available search tools: {available_tools}
Preferred tools: web_search for broad queries, web_fetch for review articles, camofox for Reddit/YouTube
Fallback: web_search + web_fetch

INSTRUCTIONS:
- Focus on reviews from the last 12 months
- Prefer comparative reviews over single-product reviews
- Note any clear consensus (same product recommended by multiple sources)
- If using camofox, navigate to specific review sites and extract key findings

OUTPUT FORMAT:
For each recommended product (3-5 max):
- Product name and model
- Why it's recommended
- Source(s) with URLs
- Key specs relevant to the use case
- Any noted drawbacks
""",
  label="agent-1-mainstream"
)

# Similarly spawn agents 2, 3, 4 in parallel (see reference file for full prompts)
```

**Agent 2 (Anti-Bias):** Reverse search strategies — negative search, lesser-known brands, origin-based search, price-point search, professional community search. Goal: break the echo chamber.

**Agent 3 (Local Market):** Scan Saudi platforms (Amazon.sa, noon.com, jarir.com, extra.com). Check actual prices, availability, seller type, shipping, warranty. Use `camofox` for live pricing if available.

**Agent 4 (Niche Community):** Deep-dive into specialized forums, small subreddits, Facebook groups, Discord servers. Find opinions from power users and professionals, not just reviewers.

> For complete agent prompts: read `references/diamond-methodology.md`

Wait for all 4 to complete before proceeding.

### Phase 2b: Expertise Layer (parallel — 2 sub-agents)

Spawn 2 sub-agents in parallel. These receive the combined results from Phase 2a.

**Agent 5 (Domain Expert):** Does NOT search. Analyzes the search results as an expert and answers 5 critical questions:
1. Do the specs actually serve the user's real-world use case?
2. What's the real (not on-paper) difference between the options?
3. Are any specs overkill for the intended use?
4. What do reviews typically miss that actually matters?
5. If buying for yourself, what would you choose and why?

**Agent 6 (Latest Tech):** Searches for:
- Products launched in the last 6 months
- Recent CES/MWC/IFA announcements
- Upcoming next-gen launches within 3 months
- Discontinued or end-of-life products
- Current vs. previous generation comparison

> For complete prompts and the expert-vs-latest-tech priority rule: read `references/domain-expertise.md`

### Phase 3: Convergence (sequential)

Merge all results and apply convergence rules:

1. **3/4 Consensus:** If 3 of 4 search agents recommended the same product, that's a strong signal — but verify they didn't all rely on the same original source
2. **Expert Overrides:** Agent 5's analysis overrides Agent 6 when they conflict (expertise > novelty)
3. **Honesty Rule:** If there's no real alternative with better value, say so plainly
4. **Timing Override:** If Agent 6 found a new generation launching within weeks at the same price, present this clearly and let the user decide
5. **Local Price Rules:** A product might be the global best but locally overpriced. Value is determined by the local price.

Apply the 5 Golden Product Criteria to each remaining option. Eliminate anything that fails 2+ criteria.

> For detailed convergence rules: read `references/anti-bias-playbook.md`

### Phase 4: Price Layer (sequential — 1 sub-agent)

Spawn a single sub-agent for price optimization:

**Agent 7 (Price & Deal Hunter):** For each finalist product:
1. Current price on every local platform
2. Active coupon codes and discounts
3. Cashback offers (bank cards, cashback apps)
4. Interest-free installment options (Tamara, Tabby)
5. Trade-in programs if applicable
6. Seller verification: official, authorized distributor, or third party
7. Shipping cost, tax inclusion, delivery time, return policy
8. **Price Inversion check:** Compare local price vs. international + shipping + customs. Alert if local is 30%+ more expensive.

> For Saudi market pricing patterns and platform details: read `references/market-dynamics.md`

### Phase 5: Output

Deliver the final recommendation in the user's preferred language using this template:

```
## Recommendation: {product_name}

### Why This Product
{Explanation grounded in the 5 Golden Product Criteria}

### Quick Comparison

| Product | Price | Platform | Rating | Note |
|---------|-------|----------|--------|------|
| ...     | ...   | ...      | ...    | ...  |

### Best Available Deal
- Platform: {platform}
- Price: {price}
- Seller: {seller_type}
- Coupon: {coupon_if_any}
- Cashback: {cashback_if_any}
- Installments: {installment_options}

### Alerts
- {timing_advice}
- {price_inversion_warning_if_any}
- {discontinuation_warning_if_any}

### Alternatives
1. {alternative_1} — {why_it's_a_good_second_choice}
2. {alternative_2} — {why_it_serves_a_different_need}

### Sources
- {list key sources with URLs used across all agents}
```

**Platform formatting notes:**
- On Discord/WhatsApp: use bullet lists instead of markdown tables
- On Telegram: tables render fine in monospace but keep them concise
- Adapt formatting to the current channel

## Worked Example

**User:** "I need a USB microphone for podcasting, budget around $150"

**Brief:** Product=USB microphone, Budget=$150, Use=podcasting, Preferences=none stated

**Search Layer results (summarized):**
- Agent 1: Shure MV7+ ($249), Audio-Technica AT2020USB-X ($129), Rode NT-USB Mini ($99)
- Agent 2: Maono PD200X ($79), Fifine K688 ($59) — hidden gems with similar specs
- Agent 3: Local prices checked, AT2020USB-X available at Jarir for 489 SAR
- Agent 4: Reddit r/podcasting consensus: AT2020USB-X best at this range; Maono PD200X great budget pick

**Expertise Layer:**
- Agent 5: For podcasting, dynamic mics (PD200X, MV7+) reject background noise better. AT2020 is condenser — great in treated rooms, problematic otherwise. The PD200X at $79 vs MV7+ at $249 offers 90% of the podcast quality.
- Agent 6: No major launches expected. MV7+ just updated. Market is stable.

**Convergence:** PD200X wins on value. MV7+ if budget allows. AT2020USB-X only for treated rooms.

**Price Layer:** PD200X on Amazon $79, noon.com 299 SAR (price inversion: +15%, acceptable).

**Output:** Recommends Maono PD200X with MV7+ as premium alternative.

## References

| File | Read When |
|------|-----------|
| `references/diamond-methodology.md` | You need full agent prompts, want to customize agent behavior, or need to adjust for product complexity |
| `references/anti-bias-playbook.md` | You need reverse search strategies, brand evaluation frameworks, or echo chamber detection |
| `references/domain-expertise.md` | You need the expert's evaluation framework, real-world examples, or the expert-vs-latest-tech priority rule |
| `references/market-dynamics.md` | You need Saudi market pricing patterns, platform comparison, seller verification, or price inversion detection |

## Implementation Notes

- **Agent count is flexible:** For simple products (USB cable, phone case), use 3 agents: Mainstream + Local Market + Price. Scale agents to match decision complexity.
- **Don't hardcode models:** Use "fast model" for search agents and "reasoning model" for analysis. Let the platform choose.
- **Each sub-agent works independently:** Search-layer agents don't see each other's results. Expertise-layer agents see search results but not each other.
- **Timeout handling:** If a sub-agent takes too long, proceed with available results. Note which agent's data is missing.
- **Nested sub-agents are optional:** Only use when product complexity justifies the extra parallelism. The orchestrator decides.

## Core Principle

> The product is anchored on VALUE, not on BRAND.
> We expand the search horizon so decisions are based on value, not just what shows up first on Google.
