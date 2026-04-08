# Search Term Profitability Audit

A Claude Code skill that audits Google Ads search terms by profitability. It classifies every term into a tier, breaks performance down by intent and funnel stage, generates prioritised recommendations, and exports everything into a clean Excel report.

## What It Does

- Pulls search term data directly from your Google Ads account via the API
- Classifies every term into one of 4 profitability tiers
- Tags each term by intent (Brand, Competitors, Product, Generic) and funnel stage (Bottom, Mid, Top)
- Generates a Recommendations sheet with prioritised, data-driven actions
- Exports a full Excel report with 8 sheets — ready to share with a client or act on yourself

## Example Output

```
Audit complete: BrightSkin | 1 Jan – 31 Jan 2026

Total spend: $17,321  |  Revenue: $36,798  |  ROAS: 2.1x
Wasted spend: $255 (1.5% of budget)

Key findings:
• 9 waste terms — add as negatives immediately
• Competitor terms spending $119 at 0.7x ROAS — below break-even
• Generic terms = 59% of spend at 0.9x ROAS — shift budget toward Product/Brand
• Bottom funnel terms: 3.9x ROAS — make sure these campaigns are uncapped
```

## Profitability Tiers

| Tier | Definition |
|------|------------|
| `profitable` | Spend ≥ threshold AND converting at or below target CPA / above target ROAS |
| `unprofitable` | Spend ≥ threshold with zero conversions, or CPA > 3× target |
| `untested` | Spend < threshold — not enough data to judge |
| `waste` | Structurally irrelevant (jobs, Reddit, DIY, etc.) — negatives recommended regardless of spend |

A term must reach 2× your target CPA in spend before it can be called unprofitable. This avoids false positives on low-data terms.

## Excel Report Structure

| Sheet | Contents |
|-------|----------|
| README | Tier definitions, intent categories, funnel stages |
| Recommendations | Prioritised findings with specific actions (High / Medium / Low) |
| Summary | Spend, revenue, ROAS, waste health, breakdown by tier / intent / funnel |
| Profitability Summary | All terms grouped by tier with full metrics |
| Intent Breakdown | Performance by Brand, Competitors, Product, Generic |
| Funnel Breakdown | Performance by Bottom, Mid, Top funnel stage |
| All Terms | Every search term with tier, intent, funnel stage, and all metrics |
| searchTerm_data | Raw data from the Google Ads API |

## Requirements

- Python 3.9+
- Google Ads API access (Developer Token + OAuth credentials)
- `~/google-ads.yaml` configured with your credentials
- Account alias configured in `.claude/accounts.json`

## Installation

**1. Copy the skill into your project:**

```
.claude/skills/search-term-audit/
```

**2. Install dependencies:**

```bash
pip install -r .claude/skills/search-term-audit/scripts/requirements.txt
```

**3. Configure your Google Ads API credentials** at `~/google-ads.yaml`.

**4. Add your account to `.claude/accounts.json`:**

```json
{
  "accounts": {
    "my-client": {
      "customer_id": "1234567890",
      "currency": "USD"
    }
  }
}
```

## Usage

Once installed, tell Claude:

> "Run a search term audit for [account] — last 30 days, target CPA $50"

Claude will ask for the account, timeframe, and target CPA or ROAS, run the audit, summarise the key findings, and save the Excel report to `data/google-ads/{account}/`.
