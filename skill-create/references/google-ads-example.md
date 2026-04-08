# Worked Example: Search Term Audit Skill

A complete, copy-paste-ready Google Ads skill built using the skill-creator pattern.

## Folder Structure

```
.claude/skills/search-term-audit/
├── SKILL.md
└── scripts/
    └── pull_search_terms.py   ← optional, if using the Google Ads API
```

## SKILL.md

```markdown
---
name: search-term-audit
description: Audits a Google Ads search term report to identify wasted spend, classify
  query intent, and recommend negative keywords. Use when the user shares a search
  term CSV, asks to find irrelevant queries, wants to reduce wasted ad spend, or asks
  to audit search terms.
---

# Search Term Audit

## Inputs

Accept one of:
- A CSV export from Google Ads (Search Terms report)
- A pasted list of search terms with metrics

Minimum required columns: search term, cost.
Ideal: cost, conversions, conversion value, impressions, clicks.

If inputs are missing, ask before proceeding.

## Step 1: Classify Each Term by Intent

| Category | Definition |
|----------|------------|
| Brand | Includes your brand name or product names |
| Competitors | Searches for competing brands or products |
| Product | Category or product-type searches without a specific brand |
| Generic | Broad informational or non-specific queries |

## Step 2: Flag Waste

Terms to flag regardless of spend:
- Job searches (e.g., "careers", "jobs", "hiring")
- DIY / how-to queries unrelated to buying
- Competitor brand names (if not intentionally targeting them)
- Navigational queries for unrelated brands

For all other terms: flag as unprofitable only if spend ≥ 2× target CPA with zero conversions.
If the user hasn't provided a target CPA, ask before flagging.

## Step 3: Identify Top Performers

Surface terms converting at or below target CPA with meaningful spend.
These are candidates to add as exact match keywords if not already present.

## Step 4: Output Recommendations

Produce a markdown report with three sections:

### Summary
- Total spend analyzed
- Estimated wasted spend (amount + % of total)
- Top 3 performing terms

### Recommended Negatives
| Search Term | Match Type | Reason |
|-------------|------------|--------|
| [term] | Exact / Phrase | [why it's irrelevant or unprofitable] |

Group similar irrelevant terms under a theme (e.g., "job-related queries") rather
than listing every variation individually.

### Recommended Exact Match Additions
| Search Term | Spend | Conversions | CPA |
|-------------|-------|-------------|-----|
| [term] | $X | X | $X |

## Edge Cases

- No conversion data: flag terms by cost alone, note the limitation in the summary
- Ambiguous terms: do not recommend negating without asking first
- Small dataset (<50 terms): note that findings are directional, not conclusive
```

---

## Why This Works

- **Description** names the input (CSV), names the output (negatives + recommendations), and lists trigger phrases
- **Body** is a clear sequential workflow — classify → flag → surface → output
- **Output format** is defined so Claude never guesses what the deliverable looks like
- **Edge cases** are handled inline, not left to Claude's judgment

## Adapting This Pattern to Other Google Ads Skills

Same structure every time: inputs → workflow steps → output format → edge cases.

| Skill | What It Does |
|-------|-------------|
| `rsa-copy` | Interview user → scrape landing page → write 15 headlines + 4 descriptions |
| `pmax-audit` | Review Performance Max asset groups, campaign structure, and budget allocation |
| `campaign-structure-review` | Evaluate campaign/ad group organization against best practices |
| `negative-keyword-audit` | Cross-reference search terms with existing negatives to find gaps |
| `product-feed-audit` | Review Shopping feed for missing attributes, poor titles, and category issues |
