# Product Page Audit Checklist

32 items across 7 categories. Tracking is excluded — use a dedicated tracking audit for that.

---

## 1. Above the Fold (6 items)

| # | Item | Notes |
|---|------|-------|
| 1.1 | Product title is clear, keyword-rich, and matches search intent | Check H1 text |
| 1.2 | Hero image is high-res, white background, product centered | Check first image src/alt |
| 1.3 | Multiple images present: lifestyle, angles, detail shots, scale reference | Count image elements |
| 1.4 | Price is prominent and immediately visible | Check for price element in page source |
| 1.5 | "Add to Cart" button is high-contrast and above the fold on mobile | Check button text and placement |
| 1.6 | Stock urgency shown ("Only 3 left") if applicable | Check for inventory/scarcity text |

**Impact: HIGHEST — directly affects whether visitors engage**

---

## 2. Trust & Social Proof (5 items)

| # | Item | Notes |
|---|------|-------|
| 2.1 | Star rating and review count displayed near the title | Check for review widget near H1 |
| 2.2 | Reviews visible without scrolling far | Check review section position |
| 2.3 | UGC photos present in reviews (not just text) | Check review image elements |
| 2.4 | Trust badges present: free returns, secure checkout, guarantee | Check for badge/icon elements |
| 2.5 | Social proof urgency present (e.g. "X people bought this") if real data backs it | Check for live social proof widgets |

**Impact: HIGH — reduces purchase anxiety**

---

## 3. Product Copy (5 items)

| # | Item | Notes |
|---|------|-------|
| 3.1 | Description headline is benefit-first, not feature-first | Read first sentence of description |
| 3.2 | Bullet points answer: What is it? Who is it for? Why does it matter? | Check bullet structure |
| 3.3 | Copy addresses top 3 objections or has an FAQ section | Check for FAQ or objection handling |
| 3.4 | No walls of text — content is scannable (bullets, headers, short paragraphs) | Visual scan of description length/format |
| 3.5 | No spelling errors, placeholder text, or broken formatting | Spot check copy quality |

**Impact: MEDIUM — affects persuasion and confidence**

---

## 4. Variants & Options (3 items)

| # | Item | Notes |
|---|------|-------|
| 4.1 | Variant selectors are clear (color swatches vs. confusing dropdowns) | Check variant input types |
| 4.2 | Out-of-stock variants are visually disabled, not hidden | Check disabled/sold-out state |
| 4.3 | Selecting a variant updates the product image | Requires live browser — mark Unverified |

**Impact: MEDIUM — affects usability and reducing friction**

---

## 5. Page Speed (3 items)

| # | Item | Notes |
|---|------|-------|
| 5.1 | Images are compressed (WebP preferred, <200KB each) | Check image file extensions in src attributes |
| 5.2 | No excessive third-party app scripts loading on product pages | Requires live browser/DevTools — mark Unverified |
| 5.3 | Mobile load time under 3 seconds | Cannot verify via fetch — always Unverified; direct user to PageSpeed Insights |

**Impact: LOWER in audit (needs dev work), but critical for ad performance**
**Note: All page speed items typically require a live browser or PageSpeed Insights.**

---

## 6. Mobile UX (4 items)

| # | Item | Notes |
|---|------|-------|
| 6.1 | ATC button is sticky on scroll (mobile) | Requires live mobile browser — mark Unverified |
| 6.2 | Images are swipeable on mobile | Requires live mobile browser — mark Unverified |
| 6.3 | No text smaller than 14px | Check inline font-size styles if present |
| 6.4 | Tap targets (buttons, links) are at least 44x44px | Requires live browser — mark Unverified |

**Impact: HIGH — majority of eCommerce traffic is mobile**
**Note: Most mobile UX items require live device testing. Flag clearly.**

---

## 7. SEO & Ads Alignment (6 items)

| # | Item | Notes |
|---|------|-------|
| 7.1 | Title tag includes primary keyword + brand | Check `<title>` tag |
| 7.2 | Meta description has a benefit hook and CTA | Check `<meta name="description">` |
| 7.3 | URL is clean: `/products/keyword-name`, no auto-generated IDs | Check URL structure |
| 7.4 | Product schema markup present (`application/ld+json`) | Search for `ld+json` script tag |
| 7.5 | Page H1 matches the keyword ads are sending traffic to | Compare H1 to ad copy (ask user for ad headline if unknown) |
| 7.6 | Landing page copy mirrors ad messaging (message match) | Ask user for ad headline if reviewing for ads |

**Impact: MEDIUM — affects traffic quality and Quality Score**

---

## Summary Totals

| Category | Items |
|---|---|
| Above the Fold | 6 |
| Trust & Social Proof | 5 |
| Product Copy | 5 |
| Variants & Options | 3 |
| Page Speed | 3 |
| Mobile UX | 4 |
| SEO & Ads Alignment | 6 |
| **Total** | **32** |
