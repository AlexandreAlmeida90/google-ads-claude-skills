# Google Ads Claude Skills

Five Claude Code skills for Google Ads and eCommerce. Each skill is a self-contained folder you drop into `.claude/skills/` in your Claude Code project — Claude activates it automatically when the task matches.

---

## Skills

### [Search Term Profitability Audit](./search-term-audit/)
Pulls search terms from a Google Ads account, classifies every term by profitability tier, intent, and funnel stage, and exports a full Excel report with prioritised recommendations.

### [Product Feed Optimization](./product-feed-optimization/)
Audits a Google Shopping product feed against required and recommended attributes, fixes `google_product_category` and `product_type`, rewrites titles and descriptions, and outputs a ready-to-upload TSV.

### [Shopify Product Page Audit](./product-page-audit/)
Audits any Shopify product page against a 32-item CRO checklist across 7 categories. Grades the page A–F and generates a branded HTML report with the top 5 highest-impact fixes.

### [RSA Copywriter](./rsa-copywriter/)
Interviews you about the product, ad group, and competitors, then generates a complete RSA — 15 headlines and 4 descriptions — within Google's character limits.

### [Skill Creator](./skill-create/)
A guide for building new Claude Code skills — anatomy, trigger conditions, progressive disclosure, and bundled resources.

---

## Requirements

- [Claude Code](https://claude.ai/code) installed
- Python 3.9+ (for skills that run scripts)
- Google Ads API credentials at `~/google-ads.yaml` — required for Search Term Audit only

## Installation

Copy any skill folder into your Claude Code project:

```
.claude/skills/search-term-audit/
.claude/skills/product-feed-optimization/
.claude/skills/product-page-audit/
.claude/skills/rsa-copywriter/
.claude/skills/skill-create/
```

Claude activates each skill automatically when the task matches — no slash commands needed.
