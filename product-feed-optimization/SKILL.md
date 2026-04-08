---
name: product-feed-optimization
description: >
  Product feed audit and optimization skill. Reads a TSV feed file downloaded from
  Google Merchant Center, audits all attributes against required and recommended
  standards, fixes product_type and google_product_category, rewrites titles and
  descriptions, suggests custom label strategy, and outputs an optimized TSV file.
  AUTO-ACTIVATE when user says: "feed audit", "optimize feed", "product feed",
  "feed optimization", "fix my feed", "audit feed", or "/product-feed-optimization".
---

# B6 Product Feed Optimization

Audits and optimizes a Google Shopping product feed — attribute coverage, title
quality, description quality, category accuracy, and custom label strategy.

---

## Step 0: Get the Feed File and Confirm Vertical

Ask the user:

> "What's the path to your TSV feed file?
>
> To download it from Google Merchant Center: Products → All products → Download (top right) → Products TSV."

Once you have the path, run the script in **detect-only mode** to auto-detect the vertical:

```bash
python3 "[SKILL_PATH]/scripts/b6_feed_audit.py" \
  --feed="[PATH]" \
  --brand="[BRAND]" \
  --detect-only
```

This will print:
- Product count and column list
- Auto-detected vertical (e.g. `Supplements & Health`, `Apparel`, `Pet`)

Then ask the user to confirm before running the full audit:

> "I loaded **[X] products** across **[Y] columns**.
>
> Based on the product categories and titles, I'm treating this as a **[VERTICAL]** feed.
>
> Does that look right, or should I use a different vertical? (Options: apparel, supplements, pet, electronics, home, food, media, automotive, general)"

If they confirm, proceed. If they specify a different vertical, add `--vertical=[vertical]` to override:

```bash
python3 "[SKILL_PATH]/scripts/b6_feed_audit.py" \
  --feed="[PATH]" \
  --brand="[BRAND]" \
  --output="[OUTPUT_DIR]" \
  --vertical=[vertical]
```

**SKILL_PATH** is the absolute path to this skill's folder:
`.claude/skills/product-feed-optimization`

Note the total product count and the column list — you'll use both in Phase 1.

---

## Step 1: Attribute Audit

Check every product against two lists: required attributes and recommended attributes.

### Required attributes (must be present — missing values cause disapprovals)

```bash
python3 -c "
import pandas as pd
df = pd.read_csv('[PATH]', sep='\t', dtype=str)

required = ['id','title','description','link','image_link','availability',
            'price','brand','gtin','identifier_exists','condition',
            'google_product_category','product_type','item_group_id']

apparel = ['age_group','gender','size','size_system','size_type']

recommended = ['sale_price','additional_image_link','lifestyle_image_link',
               'color','material','pattern','mpn','product_detail',
               'product_highlight','is_bundle','multipack','shipping_label',
               'cost_of_goods_sold']

print('=== REQUIRED ===')
for attr in required:
    if attr in df.columns:
        missing = df[attr].isna().sum() + (df[attr].eq('')).sum()
        pct = round(missing / len(df) * 100)
        print(f'  {attr}: {\"OK\" if missing == 0 else f\"MISSING in {missing} products ({pct}%)\"}')
    else:
        print(f'  {attr}: NOT IN FEED')

print('\n=== RECOMMENDED ===')
for attr in recommended:
    if attr in df.columns:
        missing = df[attr].isna().sum() + (df[attr].eq('')).sum()
        pct = round(missing / len(df) * 100)
        print(f'  {attr}: {pct}% missing')
    else:
        print(f'  {attr}: NOT IN FEED')

print('\n=== APPAREL (only relevant if feed contains apparel) ===')
for attr in apparel:
    if attr in df.columns:
        missing = df[attr].isna().sum() + (df[attr].eq('')).sum()
        pct = round(missing / len(df) * 100)
        print(f'  {attr}: {pct}% missing')
    else:
        print(f'  {attr}: NOT IN FEED')
"
```

Present the results to the user clearly:
- Required attributes missing or incomplete → actively causing disapprovals or suppression
- Recommended attributes with low coverage → limiting visibility and CTR
- Apparel attributes → only flag if the feed contains apparel products

---

## Step 2: Diagnose product_type and google_product_category

These two are almost always wrong or too broad. They serve different purposes:

- **google_product_category** — Google's own taxonomy. Must be the most specific category possible. Example: `Apparel & Accessories > Shoes > Athletic Shoes > Running Shoes` not just `Apparel`
- **product_type** — Your own taxonomy. Used for campaign segmentation. Should follow: Brand > Category > Subcategory. Example: `BrightSkin > Serums > Vitamin C`

```bash
python3 -c "
import pandas as pd
df = pd.read_csv('[PATH]', sep='\t', dtype=str)

if 'google_product_category' in df.columns:
    print('=== GOOGLE PRODUCT CATEGORY (top 10 values) ===')
    print(df['google_product_category'].value_counts().head(10).to_string())
else:
    print('google_product_category: NOT IN FEED')

if 'product_type' in df.columns:
    print('\n=== PRODUCT TYPE (top 10 values) ===')
    print(df['product_type'].value_counts().head(10).to_string())
else:
    print('product_type: NOT IN FEED')
"
```

After reviewing:
1. Flag any `google_product_category` values that are too broad
2. Flag any `product_type` values that are missing or inconsistent
3. Suggest corrected values based on the product titles and descriptions in the feed

---

## Step 3: Title Optimisation

### Rules

- **Front-load the most important keywords** — Google reads left to right and truncates on the right
- Include specific attributes customers search for: material, color, size, model number, use case
- Use ` - ` as a separator between major components
- Stay under 150 characters
- No promotional language (Sale!, Best!, Free!, #1, Limited Time)
- Include brand only where it is a real search term (not all brands are searched by name)

### Title pattern by category

| Category | Pattern |
|----------|---------|
| Apparel | Brand + Gender + Product Type + Style/Material + Color + Size |
| Beauty / Supplements | Brand + Product Line + Product Type + Size/Count + Key Benefit |
| Electronics | Brand + Model + Product Type + Key Spec + Capacity |
| Home & Garden | Brand + Product Type + Material + Dimensions + Color |
| Sports & Outdoors | Brand + Sport + Product Type + Key Feature + Size |
| Food & Grocery | Brand + Product Type + Variant/Flavour + Size/Count |

### Audit current titles

```bash
python3 -c "
import pandas as pd
df = pd.read_csv('[PATH]', sep='\t', dtype=str)
lengths = df['title'].str.len().fillna(0)
print(f'Under 25 chars (too short): {(lengths < 25).sum()}')
print(f'25-70 chars (short): {((lengths >= 25) & (lengths < 70)).sum()}')
print(f'70-150 chars (good): {((lengths >= 70) & (lengths <= 150)).sum()}')
print(f'Over 150 chars (too long): {(lengths > 150).sum()}')
print(f'Duplicate titles: {df[\"title\"].duplicated().sum()}')
print()
for i, row in df.head(20).iterrows():
    title = str(row.get('title', ''))
    print(f'[{len(title)}] {title}')
"
```

Rewrite titles that are too short, missing key attributes, or not front-loading keywords.
Present rewrites in a table:

| # | Original | Optimised | Chars |
|---|----------|-----------|-------|

---

## Step 4: Description Optimisation

### Rules

- Include high-volume keywords relevant to the product naturally
- List the most important product attributes — not marketing copy
- No comparisons ("better than X"), no promotional language ("buy now", "limited offer")
- Every description must be unique — do not use the same template for every product
- Minimum 500 characters, maximum 5,000 characters

### Structure

```
[What the product is + primary use case or benefit — 1-2 sentences]
[Key attributes: material, dimensions, technical specs, certifications]
[Secondary attributes: color options, compatibility, variants]
[Ideal customer or use case — 1 sentence]
```

### Audit current descriptions

```bash
python3 -c "
import pandas as pd
df = pd.read_csv('[PATH]', sep='\t', dtype=str)
lengths = df['description'].str.len().fillna(0)
print(f'Under 100 chars: {(lengths < 100).sum()}')
print(f'100-500 chars (short): {((lengths >= 100) & (lengths < 500)).sum()}')
print(f'500-5000 chars (good): {((lengths >= 500) & (lengths <= 5000)).sum()}')
print(f'Over 5000 chars: {(lengths > 5000).sum()}')
print(f'Duplicate descriptions: {df[\"description\"].duplicated().sum()}')
"
```

---

## Step 5: Custom Label Strategy

Custom labels (custom_label_0 through custom_label_4) are used to segment products
in Shopping and PMax campaigns for smarter bidding.

### Recommended framework

| Label | Purpose | Example values |
|-------|---------|----------------|
| custom_label_0 | Margin tier | high / medium / low |
| custom_label_1 | Performance tier | bestseller / regular / new |
| custom_label_2 | Seasonal relevance | evergreen / spring / summer / fall / winter |
| custom_label_3 | Price bracket | under-25 / 25-75 / 75-150 / 150-plus |
| custom_label_4 | Promo eligibility | promo-eligible / excluded |

### Check current usage

```bash
python3 -c "
import pandas as pd
df = pd.read_csv('[PATH]', sep='\t', dtype=str)
for i in range(5):
    col = f'custom_label_{i}'
    if col in df.columns:
        vals = df[col].dropna()
        vals = vals[vals != '']
        print(f'{col} ({len(vals)} populated): {vals.unique()[:10].tolist()}')
    else:
        print(f'{col}: NOT IN FEED')
"
```

After reviewing, suggest which labels to implement based on what data is available
in the feed. Note which labels would require additional data (e.g. Shopify sales
data for margin tier or bestseller status).

---

## Step 6: Output Optimised Feed

After completing all phases, apply the changes and save the optimised feed:

```bash
python3 -c "
import pandas as pd
df = pd.read_csv('[ORIGINAL_PATH]', sep='\t', dtype=str)

# Apply all changes:
# df.loc[df['id'] == 'X', 'title'] = 'New title'
# df['product_type'] = df['product_type'].map(corrections)
# etc.

import os
os.makedirs('data/feeds', exist_ok=True)
df.to_csv('[OUTPUT_PATH]', sep='\t', index=False)
print(f'Saved: [OUTPUT_PATH]')
"
```

**Output file name:** `[YYYY-MM-DD]-[brand]-feed-optimised.tsv`
**Save to:** `data/feeds/`

---

## Step 7: Audit Summary

After saving the file, present a summary:

```
Feed Audit: [Brand] | [Date]
[X] products | [X] columns

ATTRIBUTE COVERAGE
  Required:    [X]/14 present and complete
  Recommended: [X]/14 present

TOP ISSUES FIXED
  • [e.g. google_product_category updated to specific subcategories across all products]
  • [e.g. 47 titles rewritten — keywords front-loaded, material and color added]
  • [e.g. 23 duplicate descriptions replaced with unique copy]

STILL NEEDS ATTENTION
  • [e.g. cost_of_goods_sold — requires Shopify COGS data to populate]
  • [e.g. custom_label_0 (margin tier) — map from Shopify sales data]
  • [e.g. gtin missing on 12 products — add manually if GTINs are available]

Optimised feed saved to: data/feeds/[filename].tsv
```

---

## Feed Size Guidelines

| Feed size | Approach |
|-----------|----------|
| Under 200 products | Process all products fully |
| 200–1,000 products | Full attribute audit on all; rewrite top 200 titles/descriptions (by impressions if available, otherwise first 200) |
| 1,000+ products | Full attribute audit on all; group by category — create one optimised title/description pattern per category, apply across all products in that category, spot-check 10 samples per category |

---

## Requirements

- Python 3.9+
- `pandas` — `pip install pandas`
- TSV feed file downloaded from Google Merchant Center (Products → All products → Download)

---

---

# Part 2: Title Generation

Generates AI-optimised product titles using the Claude API. Run this after Part 1 (the attribute audit) has been completed and reviewed.

**SKILL_PATH** is the absolute path to this skill's folder:
`.claude/skills/product-feed-optimization`

---

## Step 1: Generate Sample Titles for Approval

Run the title optimizer in sample mode. It selects up to 15 representative products (prioritising those flagged in the attribute audit), generates new titles in a single Claude API call, and outputs a preview Excel file.

```bash
python3 "[SKILL_PATH]/scripts/b6_title_optimizer.py" \
  --feed="[PATH]" \
  --brand="[BRAND]" \
  --vertical=[vertical] \
  --mode=sample \
  --output="[OUTPUT_DIR]"
```

The script will print the output path and the token/cost estimates for both scope options.

Present the sample titles Excel to the user and ask:

> "Here are 15 sample titles. Do these look right? Any adjustments to the style, structure, or attributes I'm including?
>
> Once you're happy, I can generate titles for:
> - **Flagged only** — [X] products with title issues (too short / duplicate / too long) — [TOKENS] (~$[COST])
> - **Full feed** — [Y] products — [TOKENS] (~$[COST])
>
> More products = more token usage. Which would you like?"

---

## Step 2: Generate Full Scope

Once the user approves the sample quality and chooses their scope, run in full mode:

```bash
# Flagged products only
python3 "[SKILL_PATH]/scripts/b6_title_optimizer.py" \
  --feed="[PATH]" \
  --brand="[BRAND]" \
  --vertical=[vertical] \
  --mode=full \
  --scope=flagged \
  --output="[OUTPUT_DIR]"

# OR: entire feed
python3 "[SKILL_PATH]/scripts/b6_title_optimizer.py" \
  --feed="[PATH]" \
  --brand="[BRAND]" \
  --vertical=[vertical] \
  --mode=full \
  --scope=all \
  --output="[OUTPUT_DIR]"
```

**Output files:**
- `[YYYY-MM-DD]-[brand]-titles.xlsx` — 2 tabs:
  - **Title Suggestions** — side-by-side comparison (current vs. suggested, char counts, notes)
  - **Updated Feed** — complete feed with new titles applied (green cells = changed)
- `[YYYY-MM-DD]-[brand]-title-feed.tsv` — supplemental feed (id + title only) ready to upload to Merchant Center

---

## Merchant Center Upload

The `.tsv` supplemental feed can be uploaded directly to Merchant Center without re-uploading the primary feed:

1. Go to **Merchant Center → Products → Feeds**
2. Click **+ Add supplemental feed**
3. Upload `[brand]-title-feed.tsv`
4. Map the `id` and `title` columns
5. Allow 24–48 hours for processing

---

## Model Options

| Model | Speed | Cost | Use when |
|-------|-------|------|----------|
| `claude-haiku-4-5-20251001` | Fast | Very low | Default — good for straightforward product feeds |
| `claude-sonnet-4-6` | Medium | Low | Complex products with many attributes or nuanced language needs |

Pass `--model=claude-sonnet-4-6` to use Sonnet.

---

## Requirements

- Python 3.9+
- `pandas`, `openpyxl`, `anthropic` — `pip install pandas openpyxl anthropic`
- `ANTHROPIC_API_KEY` environment variable set
