# SEO Audit Report — sudonex.com

**Date:** 2026-06-03
**Overall Score:** 49/100

---

## Executive Summary

Sudonex.com has significant technical and content issues blocking both search engine indexing and AI citation eligibility. The site scores 49/100 overall, with mobile performance at 51/100. Three issues require immediate action before any content or link-building work: (1) SEO brief metadata is rendering as visible page text on resource pages, (2) the site has no robots.txt or sitemap, making crawling unreliable, and (3) stat counters display "0+" due to broken JS, actively undermining credibility. Core Web Vitals fail on mobile — LCP at 11.5s is well beyond the 2.5s threshold, driven by a redirect chain and render-blocking JavaScript. Schema markup contains placeholder data and a future-dated `dateModified`, which signals low quality to both Google and AI crawlers.

---

## Score Breakdown

| Category | Score | Status |
|---|---|---|
| Technical SEO | ~45/100 | Failing |
| Performance (Mobile) | 51/100 | Failing |
| Content Quality | 58/100 | Borderline |
| Schema Markup | ~35/100 | Failing |
| GEO / AI Readiness | 3.5/10 | Critical |
| SXO (Search Experience) | 54/100 | Borderline |
| **Overall** | **49/100** | **Failing** |

---

## P0 — Critical Issues (Fix Immediately)

### 1. SEO Brief Metadata Rendering as Visible Page Text
**Pages affected:** Resource pages  
**Impact:** Confusing users, diluting content quality signals, likely triggering spam filters.  
**Fix:** Audit the template responsible for resource page rendering. Strip or conditionally hide any SEO brief fields (meta descriptions, keyword targets, internal notes) from the rendered HTML body. These should only exist in `<head>` or be suppressed entirely client-side.

### 2. robots.txt Returns 404
**Impact:** Googlebot and AI crawlers have no crawl directive, and the missing file itself is a crawl error logged in GSC.  
**Fix:** Create `/public/robots.txt` in the Next.js project:
```
User-agent: *
Allow: /
Sitemap: https://sudonex.com/sitemap.xml
```

### 3. sitemap.xml Returns 404
**Impact:** No sitemap means crawlers discover pages only through links. New and updated pages may not be indexed for weeks.  
**Fix:** Generate a dynamic sitemap using `next-sitemap` or a custom `/app/sitemap.ts` route. Verify it covers all service, resource, and blog URLs. Submit to GSC once live.

### 4. Broken Stat Counters Displaying "0+"
**Impact:** Homepage social proof (employee count, projects, years) shows "0+" — actively harmful to conversion and E-E-A-T signals.  
**Fix:** Debug the counter animation JS (likely a IntersectionObserver or scroll-trigger race condition). If the data is static, hardcode it server-side and remove the animation dependency entirely.

### 5. /services/casino-app-development/ Returns 404
**Impact:** This URL likely has internal links and possibly external links pointing to it. A 404 on a service page loses ranking equity and revenue intent traffic.  
**Fix:** Either restore the page or add a 301 redirect to the closest equivalent service page.

---

## P1 — High Priority (Fix Within 2 Weeks)

### 6. 307 Redirect Instead of 301 on Apex Domain
**Current:** HTTP → HTTPS redirect is a 307 (temporary).  
**Impact:** 307 redirects do not pass full PageRank. Search engines will not consolidate link equity onto the canonical HTTPS URL.  
**Fix:** Change the redirect type to 301 at the CDN/hosting layer (Vercel, Cloudflare, or Hostinger config). Do not handle this in Next.js middleware.

### 7. Noindex Risk on Apex Domain
**Impact:** If the apex (`sudonex.com`) has a `noindex` tag or X-Robots-Tag set, the homepage is excluded from the index.  
**Fix:** Audit the `<head>` output and server response headers for `noindex`. Confirm the canonical URL is `https://sudonex.com/` and that no conflicting directives exist in `next.config.js` or middleware.

### 8. LCP 11.5s — Render-Blocking JS and Redirect Chain
**Root causes:**
- Redirect chain penalty: -770ms
- Render-blocking JS: -1,350ms
- JS execution time: 5.6s

**Fix (in priority order):**
1. Resolve the redirect chain (fix the 307 → 301 and eliminate multi-hop redirects).
2. Audit `<head>` script tags. Move non-critical scripts to `defer` or `async`. Use `next/script` with `strategy="lazyOnload"` for analytics and chat widgets.
3. Code-split large JS bundles. Run `next build --analyze` (`@next/bundle-analyzer`) to identify oversized chunks.
4. Target LCP element (likely hero image or heading) should be server-rendered, not hydrated.

### 9. H1 Renders as Run-On Word (No Spaces)
**Impact:** Affects both UX readability and how search engines parse the primary heading.  
**Fix:** Locate the H1 component and identify whether the issue is CSS (`word-break`, `letter-spacing`) or a data concatenation bug (missing spaces between tokens in the string). Fix the underlying source string or CSS.

### 10. Homepage Word Count ~780 (Thin Content)
**Impact:** For a software development agency targeting competitive terms, 780 words is below threshold. Topical authority requires depth.  
**Fix:** Expand homepage content with a brief methodology section, technology stack overview, or case study snippets. Target 1,200–1,500 words of genuine value. Do not pad.

---

## P2 — Medium Priority (Fix Within 30 Days)

### 11. Schema: Placeholder Addresses in Organization Markup
**Impact:** Google's structured data testing flags placeholder values. AI crawlers use Organization schema for entity disambiguation — fake addresses undermine trust.  
**Fix:** Replace placeholder address data in the Organization schema with real or legitimately abbreviated company details. If the address is sensitive, use city/country only rather than a placeholder string.

### 12. Schema: Organization Duplicated 3x on /resources/
**Impact:** Duplicate schema confuses parsers and may cause Google to ignore all three instances.  
**Fix:** Render Organization schema once, in the site layout (e.g., `_app.tsx` or root layout), not on individual page templates. Remove page-level Organization injection from the resources template.

### 13. Schema: numberOfEmployees Wrong Type
**Current:** Plain integer.  
**Required:** `QuantitativeValue` object.  
**Fix:**
```json
"numberOfEmployees": {
  "@type": "QuantitativeValue",
  "value": 50
}
```

### 14. Schema: dateModified "2026-08-15" is a Future Date
**Impact:** A future modification date signals fabricated metadata to both Google and AI systems, reducing trustworthiness of all schema on the site.  
**Fix:** Set `dateModified` dynamically from the actual file or CMS last-modified timestamp. Never hardcode future dates.

### 15. No Service Schema on Service Pages
**Impact:** Service pages lack `Service` or `Product` schema, missing rich result eligibility.  
**Fix:** Add `@type: "Service"` schema to each service page with `name`, `description`, `provider` (linked to Organization), and `areaServed`.

### 16. CTA Hierarchy Inverted
**Impact:** Secondary CTAs are more visually prominent than primary conversion CTAs, reducing click-through to key conversion points.  
**Fix:** Audit CTA button styles across homepage and service pages. Primary CTA (e.g., "Get a Quote") should have the highest visual weight. Secondary CTA (e.g., "View Portfolio") should be outlined or text-only.

---

## P3 — Low Priority (Backlog)

### 17. No Named Authors on Content
**Impact:** Google's E-E-A-T guidelines reward author attribution. AI citation systems (Perplexity, ChatGPT) strongly prefer content with identifiable human authors.  
**Fix:** Add author bylines to blog posts and resources. Create author profile pages with name, role, and credentials. Reference authors in Article schema via `author.@type: "Person"`.

### 18. AI Citation Readiness: 42/100
**Missing:** `llms.txt`, `ai.txt`, no structured "About" data for entity recognition.  
**Fix:**
1. Create `/public/llms.txt` listing key company facts, services, and contact info in plain text for LLM crawlers.
2. Add a comprehensive About page with named leadership, founding year, and service specializations.
3. Ensure all factual claims (team size, years in operation) are consistent across schema, on-page content, and directory listings.

### 19. Blog 404s
**Impact:** Blog URLs returning 404 waste crawl budget and lose any inbound links.  
**Fix:** Audit which blog URLs are linked internally or externally. Restore content or 301-redirect to the blog index or a relevant article.

### 20. No Third-Party Trust Badges Above the Fold
**Impact:** B2B software buyers look for social proof signals (Clutch, GoodFirms, Google Reviews) early in the page.  
**Fix:** Add 2–3 verified review badges to the hero or immediately below it. Only include badges from platforms where the company has actual reviews.

---

*End of audit. Priority order: P0 → P1 → P2 → P3. Re-score after P0 and P1 fixes are deployed.*
