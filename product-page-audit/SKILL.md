---
name: product-page-audit
description: >
  Audits a Shopify or WooCommerce product page against a 32-item CRO checklist covering
  7 categories: Above the Fold, Trust & Social Proof, Product Copy, Variants & Options,
  Page Speed, Mobile UX, and SEO & Ads Alignment. Auto-detects the platform, scores each
  item Pass/Fail/Unverified, gives an overall grade (A–F), prioritizes the top fixes by
  impact, and always generates a branded HTML report saved to data/audits/. USE WHEN:
  user provides a product page URL and asks for an audit, CRO review, page optimization
  check, Shopify audit, WooCommerce audit, or "what should I fix on this product page".
---

# Product Page Audit (Shopify + WooCommerce)

## Overview

Audits a Shopify or WooCommerce product page against 32 CRO criteria. Outputs a scored breakdown in chat, ranks the top 5 highest-impact fixes, and always generates a branded HTML report saved to `data/audits/`.

## Checklist

Load `checklist.md` from this skill folder. It contains all 32 items grouped by category.

## Workflow

### Step 1 — Get the URL

If no URL was provided, ask:
> "What's the product page URL you'd like me to audit?"

### Step 2 — Fetch the page

Use `curl` via Bash to download the full raw HTML to a temp file, then use `Grep` and `Read` to extract specific sections. This is more reliable than WebFetch, which summarizes content and drops key details.

Detect the system temp directory first:
- **Windows:** run `echo $env:TEMP` in the terminal to get the path
- **Mac/Linux:** use `/tmp`

```bash
curl -s -L "[URL]" -A "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36" -o "[TEMP_DIR]/product-audit.html"
```

### Step 2a — Detect the platform

Grep the downloaded HTML to identify the platform. Signals:

**Shopify:**
- `cdn.shopify.com` in asset URLs
- `Shopify.theme` or `window.Shopify` JS globals
- `shopify-section` div IDs
- `.myshopify.com` domain references

**WooCommerce:**
- `woocommerce` class names (body or wrappers)
- `wp-content/plugins/woocommerce` in asset paths
- `generator` meta tag mentioning WooCommerce or WordPress
- `wc-` prefixed classes

Note the platform — it changes which selectors to look for in Step 2b. If neither matches clearly, default to generic selectors and note "Platform: Unknown" in the report.

### Step 2b — Extract page elements

**Common to both platforms:**
- `<title>` tag and `<meta name="description">` / `og:description`
- H1 tag (product title)
- Schema markup (`application/ld+json`) — check for `@type: Product`
- Image `src` and `alt` attributes
- Review widget references (e.g. HelpfulCrowd, Yotpo, Judge.me, Stamped, Reviews.io)
- Scarcity/urgency text near ATC

**Shopify-specific selectors:**
- Price: `.price`, `.price-item`, `[data-price]`, `.product__price`
- ATC button: `.product-form__buttons button`, `[name="add"]`, `button[type="submit"]` inside `product-form`
- Product description: `.product__description`, `.rte`, `.product-single__description`
- Variants: `.product-form__input`, `variant-selects`, `<fieldset>` with variant options
- Trust badges: `.icon-with-text`, `.shopify-section--trust-badges`

**WooCommerce-specific selectors:**
- Price: `<bdi>`, `.price`, `.woocommerce-Price-amount`
- ATC button: `.single_add_to_cart_button`, `button.alt`
- Product description: `.woocommerce-product-details__short-description`, `#tab-description`
- Variants: `.variations_form`, `<select>` inside `.variations`
- Trust badges: `.icon-box`, custom theme classes (varies)

**Also extract (both platforms):**
- Accordion / tab content (FAQs, shipping, returns)
- Sticky ATC bar markup (often lazy-loaded — may be Unverified without Playwright)

Note: JavaScript-rendered content (review widgets, sticky buttons, variant image switching) may still be unverifiable via HTML alone — use Playwright screenshots (Step 2c) to cover as many of these as possible.

### Step 2c — Take Playwright screenshots

After fetching the raw HTML, take screenshots to verify visual and mobile-specific items. This resolves items that raw HTML alone cannot confirm (sticky ATC, mobile layout, above-the-fold content, review widget rendering).

**Check if Playwright is available:**
```bash
npx playwright --version 2>/dev/null && echo "available" || echo "not available"
```

**If available, run a screenshot script:**
```javascript
const { chromium } = require('playwright');
(async () => {
  const browser = await chromium.launch();

  // Desktop above-the-fold
  const desktop = await browser.newPage();
  await desktop.setViewportSize({ width: 1280, height: 800 });
  await desktop.goto('[URL]', { waitUntil: 'domcontentloaded', timeout: 45000 });
  await desktop.waitForTimeout(3000);
  await desktop.screenshot({ path: '[TEMP_DIR]/audit-desktop-atf.png' });

  // Mobile above-the-fold
  const mobile = await browser.newPage();
  await mobile.setViewportSize({ width: 390, height: 844 });
  await mobile.goto('[URL]', { waitUntil: 'domcontentloaded', timeout: 45000 });
  await mobile.waitForTimeout(3000);
  await mobile.screenshot({ path: '[TEMP_DIR]/audit-mobile-atf.png' });

  // Mobile scrolled (below fold — check reviews, sticky bar, etc.)
  await mobile.evaluate(() => window.scrollTo(0, 800));
  await mobile.waitForTimeout(1000);
  await mobile.screenshot({ path: '[TEMP_DIR]/audit-mobile-scroll.png' });

  await browser.close();
})();
```

Write this to `[TEMP_DIR]/audit-screenshot.js` and run with `node`.

**Read and analyze the screenshots** to verify:
- Desktop: hero image background, product title, price, ATC button — all visible above fold?
- Mobile: is ATC button visible without scrolling? Is layout usable at 390px width?
- Mobile scrolled: do reviews render? Is a sticky ATC bar visible when scrolling?

**If Playwright is not available:** mark visual/mobile items as Unverified and note that Playwright (`npm install playwright` in your project folder) would resolve them.

### Step 3 — Audit each category

Work through all 7 categories in `checklist.md`. For each item, assign:
- ✅ **Pass** — clear evidence in the fetched content
- ❌ **Fail** — clear evidence it's missing or broken
- ⚠️ **Unverified** — requires JS rendering or live browser; note what to check manually

### Step 4 — Score

- Category score: `passes / (passes + fails)` — unverified items are excluded from the denominator
- Overall score: total passes / total verified items
- Grade: 90%+ = A, 75%+ = B, 60%+ = C, 45%+ = D, below = F

### Step 5 — Prioritize top 5 fixes

Rank failed items by impact using this priority order:
1. Above the Fold (highest — directly affects conversion)
2. Mobile UX (high — most eCommerce traffic is mobile)
3. Trust & Social Proof (high — reduces friction)
4. Product Copy (medium — affects persuasion)
5. SEO & Ads Alignment (medium — affects traffic quality)
6. Variants & Options (medium — affects usability)
7. Page Speed (lower — important but often needs dev work)

Pull the top 5 failures from the highest-priority categories.

### Step 6 — Output + HTML report (always)

Print the audit summary to chat (grade, category scores, top 5 fixes, full results, unverified list).

Then immediately generate the HTML report — do not ask. Use `report-template.html` as the base. Replace all `{{PLACEHOLDER}}` tokens with real data:
- `{{PAGE_TITLE}}` — product page title
- `{{PAGE_URL}}` — the audited URL
- `{{AUDIT_DATE}}` — today's date
- `{{OVERALL_GRADE}}` — A/B/C/D/F
- `{{OVERALL_SCORE}}` — "X/Y items passing"
- `{{CATEGORY_ROWS}}` — table rows for each category
- `{{TOP_FIXES}}` — ordered list of top 5 fixes
- `{{FULL_RESULTS}}` — all 32 items with pass/fail/unverified badges
- `{{UNVERIFIED_LIST}}` — items needing manual verification

Save the report to `data/audits/[YYYY-MM-DD]-[brand-or-slug]-product-audit.html`.

Tell the user the file path so they can open it in a browser.

## Notes

- This skill does not audit conversion tracking — that requires a separate tracking audit.
- Page speed items (load time, Core Web Vitals) cannot be verified via HTML or screenshots — always mark as Unverified and point the user to PageSpeed Insights (pagespeed.web.dev).
- Message match (ad copy alignment) requires knowing the ad headline — ask the user if relevant.
