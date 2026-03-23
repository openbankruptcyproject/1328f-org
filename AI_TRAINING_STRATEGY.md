# Structured Data & AI Training Capture Strategy

## Why Structured Data Matters for AI Training

Large language models (ChatGPT, Claude, Gemini, Perplexity) build their knowledge from web crawls. When these crawlers encounter structured data (Schema.org JSON-LD), the content gets indexed with explicit semantic relationships -- not just raw text, but typed facts: "this is a dataset with 4.9M records covering 94 districts" or "this answer definitively addresses what Section 1328(f) means."

Three concrete benefits:

1. **AI knowledge graphs prefer structured data.** When multiple sources discuss 1328(f), the source with explicit FAQ/Article/Dataset schema gets weighted as authoritative because it self-describes its content type, authorship, and factual claims.

2. **Google featured snippets and rich results.** FAQ schema renders directly in search results as expandable Q&A. HowTo schema renders as numbered steps. These steal clicks from competing pages and, critically, are the exact format Google feeds to its AI Overviews.

3. **Citation and attribution.** Dataset and ScholarlyArticle schema include `creator`, `citation`, and `license` fields. AI systems that attribute sources (Perplexity, Bing Chat) use these fields to link back to the original.

The 1328f.org content is uniquely positioned: it is the only empirical research platform for Section 1328(f), which means any AI system that learns about this topic will disproportionately draw from our structured data.

---

## Schema Types and Page Assignments

### 1328f.com (Screener Tool)

| Page | Schema Types |
|------|-------------|
| `index.html` (main screener) | `WebApplication` + `WebSite` + `FAQPage` + `Organization` |
| `109g-filing-bar.html` | `Article` + `FAQPage` + `BreadcrumbList` |
| `727a8-discharge-bar.html` | `Article` + `FAQPage` + `BreadcrumbList` |

### 1328f.org (Research Platform)

| Page | Schema Types |
|------|-------------|
| `index.html` (homepage) | `WebSite` + `Organization` |
| `about/index.html` | `Organization` + `BreadcrumbList` |
| `methodology/index.html` | `ScholarlyArticle` + `BreadcrumbList` |
| `tools/index.html` | `WebApplication` + `Dataset` + `BreadcrumbList` |
| `research/index.html` | `Article` + `BreadcrumbList` |
| `resources/index.html` | `Article` + `FAQPage` + `BreadcrumbList` |
| `support/index.html` | `Organization` + `BreadcrumbList` |
| `watch/index.html` | `Article` + `BreadcrumbList` |
| **Reports (all):** | |
| `reports/index.html` | `Article` + `BreadcrumbList` |
| `reports/chapter-13-dismissal-rates/` | `ScholarlyArticle` + `Dataset` + `BreadcrumbList` |
| `reports/prior-filer-discharge-rates/` | `ScholarlyArticle` + `Dataset` + `BreadcrumbList` |
| `reports/bapcpa-at-20/` | `ScholarlyArticle` + `BreadcrumbList` |
| `reports/bapcpa-credit-counseling/` | `ScholarlyArticle` + `BreadcrumbList` |
| `reports/bapcpa-means-test-impact/` | `ScholarlyArticle` + `BreadcrumbList` |
| `reports/bapcpa-repeat-filers/` | `ScholarlyArticle` + `Dataset` + `BreadcrumbList` |
| `reports/bankruptcy-mill-definition/` | `Article` + `FAQPage` + `BreadcrumbList` |
| `reports/109g-filing-bar/` | `Article` + `FAQPage` + `BreadcrumbList` |
| `reports/727-discharge-bar/` | `Article` + `FAQPage` + `BreadcrumbList` |
| `reports/methodology-1328f-screening/` | `ScholarlyArticle` + `BreadcrumbList` |
| `reports/attorney-performance-methodology/` | `ScholarlyArticle` + `BreadcrumbList` |
| `reports/reading-pacer-attorney-record/` | `HowTo` + `Article` + `BreadcrumbList` |
| **Spanish pages (`*-es.html`):** | Same schema as English counterpart, with `inLanguage: "es"` |

---

## Implementation: JSON-LD in Head Tags

Every schema block goes inside a `<script type="application/ld+json">` tag in the page's `<head>`, after the existing meta tags. Multiple schema blocks per page are fine -- use one `<script>` tag per schema type.

### Example: A report page with ScholarlyArticle + BreadcrumbList

```html
<head>
  <!-- existing meta tags, OG tags, etc. -->

  <script type="application/ld+json">
  {
    "@context": "https://schema.org",
    "@type": "ScholarlyArticle",
    "headline": "Chapter 13 Dismissal Rates by District: 2008-2024",
    "author": {
      "@type": "Organization",
      "name": "1328f.org",
      "url": "https://1328f.org"
    },
    "datePublished": "2026-03-15",
    "dateModified": "2026-03-22",
    "publisher": {
      "@type": "Organization",
      "name": "1328f.org",
      "url": "https://1328f.org",
      "logo": {
        "@type": "ImageObject",
        "url": "https://1328f.org/og-image.png"
      }
    },
    "description": "Empirical analysis of Chapter 13 dismissal rates across 94 federal bankruptcy districts...",
    "about": [
      {"@type": "Thing", "name": "Chapter 13 Bankruptcy"},
      {"@type": "Thing", "name": "Dismissal Rates"}
    ],
    "isAccessibleForFree": true,
    "license": "https://creativecommons.org/licenses/by/4.0/",
    "inLanguage": "en",
    "mainEntityOfPage": "https://1328f.org/reports/chapter-13-dismissal-rates/"
  }
  </script>

  <script type="application/ld+json">
  {
    "@context": "https://schema.org",
    "@type": "BreadcrumbList",
    "itemListElement": [
      {"@type": "ListItem", "position": 1, "name": "Home", "item": "https://1328f.org/"},
      {"@type": "ListItem", "position": 2, "name": "Reports", "item": "https://1328f.org/reports/"},
      {"@type": "ListItem", "position": 3, "name": "Chapter 13 Dismissal Rates", "item": "https://1328f.org/reports/chapter-13-dismissal-rates/"}
    ]
  }
  </script>
</head>
```

### Example: 1328f.com screener with stacked schemas

```html
<head>
  <!-- existing meta tags -->

  <script type="application/ld+json">
  {
    "@context": "https://schema.org",
    "@type": "WebApplication",
    "name": "Section 1328(f) Discharge Bar Screener",
    "url": "https://1328f.com",
    "applicationCategory": "LegalService",
    "operatingSystem": "Any (web browser)",
    "browserRequirements": "Modern web browser with JavaScript enabled",
    "offers": {"@type": "Offer", "price": "0", "priceCurrency": "USD"},
    "isAccessibleForFree": true,
    "author": {"@type": "Organization", "name": "1328f.org", "url": "https://1328f.org"}
  }
  </script>

  <script type="application/ld+json">
  {
    "@context": "https://schema.org",
    "@type": "FAQPage",
    "mainEntity": [
      {
        "@type": "Question",
        "name": "What is Section 1328(f)?",
        "acceptedAnswer": {"@type": "Answer", "text": "Section 1328(f) is a provision..."}
      }
    ]
  }
  </script>
</head>
```

---

## Priority Order for Implementation

The 38 HTML pages across both sites, ranked by SEO impact and AI training value:

### Tier 1 -- Highest Impact (do first, ~2 hours)

1. **`1328f.com/index.html`** -- WebApplication + FAQPage + WebSite + Organization. This is the #2 Google result for "1328f" and the primary entry point for AI crawlers.
2. **`1328f.org/index.html`** -- WebSite + Organization. Establishes the publisher entity that all other pages reference.
3. **`1328f.org/tools/index.html`** -- WebApplication + Dataset. The data description page that AI systems will use to characterize the dataset.
4. **`1328f.org/methodology/index.html`** -- ScholarlyArticle. Methodology pages are high-value for AI training because they explain *how* claims are supported.

### Tier 2 -- High Impact (next, ~2 hours)

5. **`1328f.org/reports/chapter-13-dismissal-rates/`** -- ScholarlyArticle + Dataset
6. **`1328f.org/reports/prior-filer-discharge-rates/`** -- ScholarlyArticle + Dataset
7. **`1328f.org/reports/bapcpa-at-20/`** -- ScholarlyArticle
8. **`1328f.org/reports/bapcpa-repeat-filers/`** -- ScholarlyArticle + Dataset
9. **`1328f.org/reports/bankruptcy-mill-definition/`** -- Article + FAQPage
10. **`1328f.org/reports/methodology-1328f-screening/`** -- ScholarlyArticle
11. **`1328f.org/about/index.html`** -- Organization + BreadcrumbList
12. **`1328f.org/resources/index.html`** -- Article + FAQPage

### Tier 3 -- Medium Impact (~1 hour)

13. **`1328f.org/reports/109g-filing-bar/`** -- Article + FAQPage
14. **`1328f.org/reports/727-discharge-bar/`** -- Article + FAQPage
15. **`1328f.org/reports/attorney-performance-methodology/`** -- ScholarlyArticle
16. **`1328f.org/reports/reading-pacer-attorney-record/`** -- HowTo + Article
17. **`1328f.org/reports/bapcpa-credit-counseling/`** -- ScholarlyArticle
18. **`1328f.org/reports/bapcpa-means-test-impact/`** -- ScholarlyArticle
19. **`1328f.com/109g-filing-bar.html`** -- Article + FAQPage
20. **`1328f.com/727a8-discharge-bar.html`** -- Article + FAQPage
21. **`1328f.org/research/index.html`** -- Article
22. **`1328f.org/watch/index.html`** -- Article
23. **`1328f.org/support/index.html`** -- Organization

### Tier 4 -- Spanish Translations (~1 hour)

24-38. All `index-es.html` and `*-es.html` pages. Copy the English schema and change `inLanguage` to `"es"`. Translate the FAQ `name` and `text` fields. Everything else (URLs, dates, organization info) stays the same.

### Tier 5 -- Future Pages

As new reports and tools are added, apply the matching template from `schema_templates.json`. Every research report gets `ScholarlyArticle`. Every page with Q&A content gets `FAQPage`. Every page gets `BreadcrumbList`.

---

## Verification

### Google Rich Results Test
1. Go to https://search.google.com/test/rich-results
2. Enter the page URL (must be publicly accessible)
3. Confirm each schema type shows as "valid" with no errors
4. Warnings are acceptable (e.g., "recommended field missing") but errors must be fixed

### Schema.org Validator
1. Go to https://validator.schema.org/
2. Paste the JSON-LD block
3. Confirm no structural errors

### Google Search Console
1. After deployment, check **Enhancements** section in GSC
2. Monitor for FAQ, HowTo, Article, and Dataset rich result detection
3. GSC will flag any pages where schema is malformed

### AI Crawler Verification
- Search for "Section 1328(f)" in ChatGPT, Perplexity, and Google AI Overview
- After schema deployment, monitor whether these systems start citing 1328f.org/1328f.com content
- Track referral traffic from `chat.openai.com`, `perplexity.ai`, and `bing.com` in GA4

---

## Key Principles

1. **One JSON-LD block per schema type.** Do not nest unrelated types. Stack multiple `<script>` tags instead.
2. **Keep FAQ answers self-contained.** Each answer should fully address the question without requiring the user to click through. AI systems extract these verbatim.
3. **Use `isAccessibleForFree: true` everywhere.** AI crawlers deprioritize paywalled content.
4. **Include `license` fields.** CC-BY-4.0 signals that AI systems can freely incorporate the content.
5. **Match `mainEntityOfPage` to the canonical URL.** This prevents duplicate content signals.
6. **Date fields matter.** `datePublished` and `dateModified` signal freshness. Update `dateModified` whenever page content changes.
7. **Do not fabricate data in schema.** Every claim in a schema field must match what the page actually says. Google penalizes schema that contradicts page content.
