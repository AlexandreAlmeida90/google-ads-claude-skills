# RSA Copywriter

A Claude Code skill that writes complete Google Ads Responsive Search Ads. It interviews you about the product, ad group, and competitors, scrapes the landing page to build a research profile, then generates 15 headlines and 4 descriptions — structured by type and within Google's character limits. Exports a formatted Excel file ready to copy into Google Ads.

## What It Does

- Interviews you about the business, product, ad group focus, and any copy restrictions
- Scrapes the landing page and up to 2 sub-pages to extract features, benefits, pricing, and social proof
- Generates a customer avatar based on the product research
- Optionally scrapes up to 3 competitor pages to sharpen positioning
- Generates a complete RSA — 15 headlines and 4 descriptions — structured by type
- Exports a formatted Excel file with character count formulas

## RSA Structure

**15 Headlines (max 30 chars each)**
- Key Features (3)
- Benefits (3)
- Price (1)
- Social Proof (2)
- Target Audience (1)
- USP (2)
- Offer / CTA (2)
- Seasonal / Bonus Offer (1)

**4 Descriptions (max 90 chars each)**
- Key Features / Benefits / USP (2)
- Social Proof / Authority (1)
- Offer / CTA / Seasonal (1)

## Copy Rules

- Every headline must stand alone — Google mixes them dynamically
- Lead with outcomes and specifics, not vague support language
- Use numbers wherever possible: "60 Billion CFU", "1,600+ Reviews", "Save 30%"
- Offer headlines must include a specific number or incentive — never vague CTAs
- Exactly 2 descriptions end with a CTA; the other 2 end with a benefit, feature, or proof statement
- No competitor names in any headline or description
- No em dashes — use periods, commas, or colons instead

## Requirements

- Python 3.9+
- `openpyxl` — `pip install openpyxl`

## Installation

**1. Copy the skill into your project:**

```
.claude/skills/rsa-copywriter/
```

Make sure all files are included:
- `SKILL.md`
- `scripts/`
- `references/`

**2. Install the dependency:**

```bash
pip install openpyxl
```

## Usage

Once installed, tell Claude:

> "Write RSA copy for [brand] — ad group: [keyword theme]"

Claude will guide you through a short interview, scrape the landing page, optionally research competitors, generate the full RSA, and save an Excel file to your Downloads folder.

## Output

An Excel file (`[Client] - RSA Copy - [Ad Group].xlsx`) with:
- All 15 headlines with character counts and type labels
- All 4 descriptions with character counts
- Ad path suggestions
- Empty rows pre-filled with `=LEN()` formulas for manual additions
