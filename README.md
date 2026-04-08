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

## Installation

**1. Clone this repo:**

```bash
git clone https://github.com/AlexandreAlmeida90/google-ads-claude-skills.git
```

**2. Copy the skill(s) you want into your Claude Code project:**

```bash
cp -r google-ads-claude-skills/search-term-audit your-project/.claude/skills/
cp -r google-ads-claude-skills/product-feed-optimization your-project/.claude/skills/
cp -r google-ads-claude-skills/product-page-audit your-project/.claude/skills/
cp -r google-ads-claude-skills/rsa-copywriter your-project/.claude/skills/
cp -r google-ads-claude-skills/skill-create your-project/.claude/skills/
```

**3. Install dependencies** (see each skill's README for specifics):

```bash
pip install pandas openpyxl
```

Claude activates each skill automatically when the task matches — no slash commands needed.

---

## Requirements

- [Claude Code](https://claude.ai/code) installed
- Python 3.9+
- Google Ads API credentials at `~/google-ads.yaml` — required for Search Term Audit only

---

## Disclaimer

These skills are provided as-is for educational and productivity purposes. Use them at your own risk.

We are not responsible for any changes made to your Google Ads accounts, ad spend, feed submissions, or business outcomes resulting from the use of these skills. Always review Claude's output before applying changes to live accounts or uploading files to Google Merchant Center.

These skills are not affiliated with or endorsed by Google.
