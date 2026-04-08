# Shopify Product Page Audit

A Claude Code skill that audits any Shopify product page against a 32-item CRO checklist. It scores the page across 7 categories, grades it A–F, ranks the top 5 highest-impact fixes, and generates a branded HTML report you can open in any browser.

## What It Does

- Fetches the raw HTML of any product page URL
- Takes desktop and mobile screenshots using Playwright (if installed)
- Scores each of 32 checklist items as Pass / Fail / Unverified
- Grades the page overall (A–F) based on verified items
- Ranks the top 5 fixes by conversion impact
- Generates a formatted HTML report saved to `data/audits/`

## The 7 Categories

| Category | What It Checks |
|----------|----------------|
| Above the Fold | Hero image, product title, price, ATC button — all visible without scrolling |
| Trust & Social Proof | Reviews, ratings, trust badges, guarantees |
| Product Copy | Headline clarity, benefit-led descriptions, objection handling |
| Variants & Options | Selector usability, option labelling, out-of-stock handling |
| Page Speed | Load time and Core Web Vitals (flagged for manual check via PageSpeed Insights) |
| Mobile UX | Layout at 390px, sticky ATC, tap target sizes |
| SEO & Ads Alignment | Title tag, meta description, H1, message match with ads |

## Scoring

- **Pass** — clear evidence in the fetched content or screenshots
- **Fail** — clear evidence it's missing or broken
- **Unverified** — requires live browser or JS rendering; flagged for manual check

Category score: `passes / (passes + fails)` — unverified items excluded from denominator
Overall grade: 90%+ = A, 75%+ = B, 60%+ = C, 45%+ = D, below = F

## Requirements

- `curl` (built into Windows 10+, Mac, and Linux)
- **Optional:** Node.js + Playwright for screenshots (`npm install playwright`)
  - Without Playwright, visual and mobile items are marked Unverified

## Installation

**1. Copy the skill into your project:**

```
.claude/skills/product-page-audit/
```

Make sure all 3 files are included:
- `SKILL.md`
- `checklist.md`
- `report-template.html`

**2. Create the output folder:**

```bash
mkdir -p data/audits
```

**3. (Optional) Install Playwright for screenshots:**

```bash
npm install playwright
npx playwright install chromium
```

## Usage

Once installed, give Claude a product page URL:

> "Audit this product page: [URL]"

Claude will fetch the page, run through all 32 checklist items, grade it, and save an HTML report to `data/audits/`. Open the report in your browser to view the full results.

## Output

An HTML report saved to `data/audits/[YYYY-MM-DD]-[brand]-product-audit.html` containing:
- Overall grade and score
- Category-by-category breakdown
- Top 5 highest-impact fixes
- Full 32-item results with Pass/Fail/Unverified badges
- List of items needing manual verification
