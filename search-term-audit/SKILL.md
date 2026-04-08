---
name: search-term-audit
description: |
  B6 Search Term Profitability Audit. Pulls search terms from a Google Ads account,
  classifies them by profitability tier, intent category, and funnel stage, and
  exports a full Excel report with a Recommendations sheet, Summary, Intent Breakdown,
  Funnel Breakdown, and all terms.
  AUTO-ACTIVATE when user says: "search term audit", "profitability audit",
  "wasted spend", "search term analysis", "audit search terms", or "/search-term-audit".
---

# B6 Search Term Profitability Audit

Identifies where your search term budget is working and where it isn't — using
profitability tiers, intent classification, funnel stage analysis, and spend
thresholds to avoid false positives.

## Philosophy

Most search term tools ask: *"What does this searcher intend?"*
This skill asks: *"Is this search term earning its place in the account?"*

Terms aren't judged on psychology — they're judged on business performance. A term
needs sufficient spend before it's called unprofitable. Every term is also classified
by intent (Brand, Competitors, Product, Generic) and funnel stage (Bottom, Mid, Top)
so you can see not just *what's wasting money* but *where in the funnel the leaks are*.

---

## Step 0: Gather Inputs

Ask the user:

**1. Account**
If not already known from context, ask which account to audit.
Check `.claude/accounts.json` for the alias → customer ID mapping.

**2. Timeframe**
```
Which timeframe?
1. Last 7 days
2. Last 30 days (recommended)
3. Last 90 days
4. Last 180 days
5. Custom range
```
For option 5, ask for start and end dates (YYYY-MM-DD).

**3. Target CPA or ROAS**
```
What is this account's target CPA or target ROAS?
- For CPA: enter a number (e.g. 50)
- For ROAS: enter as a percentage (e.g. 160 for 1.6x)
- Type "auto" to calculate from the data
```
The target drives the spend threshold — a term must reach 2× target CPA in spend
before it can be called unprofitable. Lower the threshold with `--min-spend` if
you want more terms evaluated.

---

## Step 1: Run the Script

```bash
python .claude/skills/search-term-audit/scripts/b6_search_term_audit.py \
  --account=<alias> \
  --days=<days> \
  --target-cpa=<amount>
```

**Using ROAS instead of CPA:**
```bash
python .claude/skills/search-term-audit/scripts/b6_search_term_audit.py \
  --account=<alias> \
  --days=<days> \
  --target-roas=<percentage>
```

**Custom date range:**
```bash
python .claude/skills/search-term-audit/scripts/b6_search_term_audit.py \
  --account=<alias> \
  --start=<YYYY-MM-DD> \
  --end=<YYYY-MM-DD> \
  --target-cpa=<amount>
```

**Optional flags:**
| Flag | Description |
|------|-------------|
| `--min-spend=N` | Override minimum spend threshold (default: 2× target CPA) |
| `--min-impressions=N` | Minimum impressions to include a term (default: 10) |
| `--currency=USD` | Force currency symbol (default: from accounts.json) |
| `--use-cache` | Skip API call and reuse data from previous run |
| `--output=PATH` | Custom output directory (default: data/google-ads/{account}/) |

---

## Step 2: Present the Results

The script saves an Excel report to `data/google-ads/{account}/{date}-{account}-audit.xlsx`
and prints a summary to the terminal.

Tell the user the file path, then summarise the key findings from the terminal output:

1. **Wasted spend** — total from waste + unprofitable terms
2. **Top recommendations** — pull the top 2–3 from the Recommendations sheet
3. **Intent mix** — flag any intent category with ROAS below break-even
4. **Funnel gaps** — highlight if mid or bottom funnel is underperforming

**Example summary:**
```
Audit complete: BrightSkin | 1 Jan – 31 Jan 2026

Total spend: $17,321  |  Revenue: $36,798  |  ROAS: 2.1x
Wasted spend: $255 (1.5% of budget)

Key findings:
• 9 waste terms — add as negatives immediately
• Competitor terms spending $119 at 0.7x ROAS — below break-even
• Generic terms = 59% of spend at 0.9x ROAS — consider shifting budget to Product/Brand
• Bottom funnel terms: 3.9x ROAS — make sure these campaigns are uncapped

Report saved to: data/google-ads/bsk/20260131-bsk-audit.xlsx
```

---

## Step 3: Offer Next Steps

```
What would you like to do next?

1. Walk through the Recommendations sheet
2. Dig into a specific intent category (Brand / Competitors / Product / Generic)
3. Review waste terms before adding negatives
4. Run audit on another account
5. Nothing — I have what I need
```

---

## Profitability Tiers

| Tier | Definition |
|------|------------|
| `profitable` | Spend ≥ threshold AND converting at or below target CPA / above target ROAS |
| `unprofitable` | Spend ≥ threshold with zero conversions, or CPA > 3× target |
| `untested` | Spend < threshold — not enough data to judge |
| `waste` | Structurally irrelevant regardless of spend (jobs, Reddit, DIY, etc.) — negatives recommended |

**Key rule:** `waste` terms bypass the spend threshold entirely.
All other tiers require the threshold to be met before calling something unprofitable.

## Intent Categories

| Category | Definition |
|----------|------------|
| `Brand` | Searches that include your brand name or product names |
| `Competitors` | Searches for competing brands or products |
| `Product` | Category or product-type searches without a specific brand |
| `Generic` | Broad informational or non-specific queries |

## Funnel Stages

| Stage | Definition |
|-------|------------|
| `Bottom Funnel` | High purchase intent — brand + product, "buy", "order" |
| `Mid Funnel` | Comparison or consideration — "best", "vs", "review" |
| `Top Funnel` | Awareness or research — "what is", "how to", broad category |

---

## Output: Excel Report

Saved to `data/google-ads/{account}/{date}-{account}-audit.xlsx`

| Sheet | Contents |
|-------|----------|
| `README` | How to read the file — tier definitions, intent categories, funnel stages |
| `Recommendations` | Prioritised findings with specific actions (High / Medium / Low) |
| `Summary` | Top-level overview: spend, revenue, ROAS, waste health, tier + intent + funnel breakdown |
| `Profitability Summary` | All terms grouped by tier with full metrics |
| `Intent Breakdown` | Performance by intent category: Brand, Competitors, Product, Generic |
| `Funnel Breakdown` | Performance by funnel stage: Bottom, Mid, Top |
| `All Terms` | Every search term with tier, intent, funnel stage, and all metrics |
| `searchTerm_data` | Raw data pulled from the Google Ads API |

**All Terms columns:**
`Search Term, Tier, Intent, Funnel Stage, Channels, Campaigns, Impressions, Clicks, CTR, CPC, Spend, Conversions, Conv Value, CPA, ROAS, Wasted Spend`

---

## Requirements

- Python 3.9+
- `google-ads` — `pip install google-ads`
- `pandas` — `pip install pandas`
- `openpyxl` — `pip install openpyxl`
- Google Ads API credentials at `~/google-ads.yaml`
- Account configured in `.claude/accounts.json`
