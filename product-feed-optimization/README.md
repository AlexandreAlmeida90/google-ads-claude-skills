# Product Feed Optimization

A Claude Code skill that audits and optimizes a Google Shopping product feed. It checks every attribute against required and recommended standards, fixes `product_type` and `google_product_category`, rewrites titles and descriptions following Shopping best practices, suggests a custom label strategy, and outputs a ready-to-upload TSV file.

## What It Does

- Audits all attributes against Google's required and recommended feed specifications
- Flags missing or incomplete fields that cause disapprovals or limit visibility
- Diagnoses and fixes `google_product_category` (too broad is one of the most common feed issues) and `product_type`
- Rewrites titles to front-load keywords and include the attributes customers actually search for
- Rewrites descriptions to be keyword-rich and attribute-focused (not marketing copy)
- Suggests a custom label strategy for smarter campaign segmentation
- Outputs an optimized TSV file ready to upload to Merchant Center

## Attributes Covered

**Required** (missing values cause disapprovals):
`id`, `title`, `description`, `link`, `image_link`, `availability`, `price`, `brand`, `gtin`, `identifier_exists`, `condition`, `google_product_category`, `product_type`, `item_group_id`

**Apparel** (required for clothing/footwear):
`age_group`, `gender`, `size`, `size_system`, `size_type`

**Recommended** (improve visibility and CTR):
`sale_price`, `additional_image_link`, `lifestyle_image_link`, `color`, `material`, `pattern`, `mpn`, `product_detail`, `product_highlight`, `is_bundle`, `multipack`, `shipping_label`, `cost_of_goods_sold`

## Title Rules

- Front-load the most important keywords — Google reads left to right and truncates on the right
- Include specific attributes customers search for: material, color, size, model number, use case
- Use ` - ` as a separator between major components
- Under 150 characters
- No promotional language (Sale!, Best!, Free!, #1)

## Requirements

- Python 3.9+
- `pandas` — `pip install pandas`
- TSV feed file from Google Merchant Center

## How to Get Your Feed File

1. Go to **Google Merchant Center**
2. Click **Products** → **All products**
3. Click **Download** (top right)
4. Select **Products TSV**

## Installation

**1. Copy the skill into your project:**

```
.claude/skills/product-feed-optimization/
```

Make sure all files are included:
- `SKILL.md`
- `README.md`
- `scripts/`
- `google-product-taxonomy.txt`

**2. Install the dependency:**

```bash
pip install pandas
```

## Usage

Once installed, tell Claude:

> "Run a feed audit for [brand] — here's the file: [path to TSV]"

Claude will audit all attributes, flag issues, rewrite titles and descriptions, suggest custom labels, and save an optimized TSV to `data/feeds/`.
