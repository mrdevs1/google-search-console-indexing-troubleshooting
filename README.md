# Google Search Console Indexing Troubleshooting Guide

A practical troubleshooting guide for diagnosing Google Search indexing problems, including crawling issues, `noindex`, `robots.txt`, canonical URLs, XML sitemaps, redirects, duplicate URLs, and pages that are discovered or crawled but not indexed.

## Table of Contents

* [Overview](#overview)
* [How Google Search Indexing Works](#how-google-search-indexing-works)
* [Common Indexing Problems](#common-indexing-problems)
* [Step 1: Confirm the URL Is Accessible](#step-1-confirm-the-url-is-accessible)
* [Step 2: Use URL Inspection](#step-2-use-url-inspection)
* [Step 3: Check Robots.txt](#step-3-check-robotstxt)
* [Step 4: Check Noindex](#step-4-check-noindex)
* [Step 5: Check Canonical URLs](#step-5-check-canonical-urls)
* [Step 6: Check XML Sitemaps](#step-6-check-xml-sitemaps)
* [Step 7: Crawled but Not Indexed](#step-7-crawled-but-not-indexed)
* [Step 8: Discovered but Not Indexed](#step-8-discovered-but-not-indexed)
* [Step 9: Troubleshoot HTTP Status Codes](#step-9-troubleshoot-http-status-codes)
* [Step 10: Investigate Redirects](#step-10-investigate-redirects)
* [Step 11: Check Duplicate URLs](#step-11-check-duplicate-urls)
* [Step 12: Check JavaScript Rendering](#step-12-check-javascript-rendering)
* [Step 13: WordPress-Specific Checks](#step-13-wordpress-specific-checks)
* [Step 14: Request Reindexing](#step-14-request-reindexing)
* [Post-Fix Validation](#post-fix-validation)
* [Common Mistakes](#common-mistakes)
* [Indexing Troubleshooting Checklist](#indexing-troubleshooting-checklist)
* [Prevention Best Practices](#prevention-best-practices)
* [Official Resources](#official-resources)
* [Disclaimer](#disclaimer)

## Overview

Google Search needs to be able to discover, crawl, process, and potentially index a webpage before that page can appear in Google Search.

A page can be technically accessible while still not being indexed.

Common causes include:

* `noindex` directives.
* Incorrect canonical URLs.
* Crawling restrictions.
* Poor internal linking.
* Duplicate content.
* Redirects.
* Server errors.
* Soft 404 situations.
* Sitemap problems.
* Rendering problems.
* Content quality or search-system decisions.

Google's Search Console provides tools for investigating indexing and crawling problems, including URL Inspection and sitemap reporting.

> **Important:** Being indexed does not guarantee that a page will rank well or appear for a particular search query.

---

## How Google Search Indexing Works

A simplified search workflow is:

```text
Discovery
   ↓
Crawling
   ↓
Processing
   ↓
Indexing
   ↓
Search serving
```

### 1. Discovery

Google discovers URLs through sources such as:

* Internal links.
* External links.
* XML sitemaps.
* Previously known URLs.
* Other discoverable references.

### 2. Crawling

Googlebot requests the URL and retrieves the page and its resources.

The server must respond appropriately and allow Google to access the content.

### 3. Processing

Google analyzes the page, resources, metadata, canonical signals, structured data, content, and other information.

### 4. Indexing

Google decides whether the page should be included in its search index.

### 5. Search serving

An indexed page may or may not appear for a particular search.

This means:

```text
Accessible ≠ Crawled ≠ Indexed ≠ Ranking
```

---

## Common Indexing Problems

| Problem                    | Possible cause                                                          | First check                          |
| -------------------------- | ----------------------------------------------------------------------- | ------------------------------------ |
| Page is not indexed        | `noindex`                                                               | Inspect page source and HTTP headers |
| Google cannot crawl page   | `robots.txt` or server restriction                                      | Test URL accessibility               |
| Crawled but not indexed    | Content, duplication, canonicalization, or other search-system decision | URL Inspection                       |
| Discovered but not indexed | Google has discovered URL but has not crawled it yet                    | Internal links and sitemap           |
| Wrong URL indexed          | Canonical or duplicate URL signals                                      | Canonical configuration              |
| Old URL remains indexed    | Redirect or removal not processed yet                                   | HTTP status and Search Console       |
| Sitemap has errors         | Invalid URLs or server problems                                         | Sitemap report                       |
| URL redirects              | Incorrect redirect configuration                                        | HTTP response chain                  |
| Page returns 404           | Missing or deleted content                                              | Server response                      |
| Search appearance differs  | Rendering or indexing changes                                           | URL Inspection                       |

---

# Step 1: Confirm the URL Is Accessible

Before changing SEO settings, verify that the URL actually works.

Open the URL in a normal browser and check:

* Does the page load?
* Does it return the expected content?
* Does HTTPS work?
* Does the page require authentication?
* Does it redirect somewhere unexpected?
* Does the server return an error?

### Basic HTTP check

You can use:

```bash
curl -I https://example.com/page/
```

A healthy page commonly returns:

```text
HTTP/2 200
```

However, the correct status depends on the resource.

For example:

```text
200 → Page successfully served
301 → Permanent redirect
302 → Temporary redirect
403 → Access forbidden
404 → Resource not found
410 → Resource permanently gone
500 → Server error
503 → Service temporarily unavailable
```

Do not treat every non-`200` response as an error. Redirects and intentionally unavailable resources can be legitimate.

---

# Step 2: Use URL Inspection

Google Search Console's **URL Inspection** tool is one of the most useful tools for diagnosing an individual URL.

Inspect the exact URL rather than relying only on a general site report.

Review:

* Whether the URL is available to Google.
* Indexing status.
* Crawling information.
* User-declared canonical.
* Google-selected canonical.
* Referring pages where available.
* Enhancements and detected issues.
* Last crawl information where available.

### Investigation workflow

```text
Search Console
      ↓
URL Inspection
      ↓
Enter exact URL
      ↓
Review indexing status
      ↓
Check canonical
      ↓
Check crawl accessibility
      ↓
Fix the underlying problem
      ↓
Request validation/reindexing when appropriate
```

Google recommends using URL Inspection to test how Google sees a page and to request a recrawl after appropriate fixes.

---

# Step 3: Check Robots.txt

The `robots.txt` file can provide crawling instructions to search engine crawlers.

Typical location:

```text
https://example.com/robots.txt
```

Example:

```text
User-agent: *
Disallow:
```

This allows crawling of URLs unless another rule applies.

A restrictive example:

```text
User-agent: *
Disallow: /private/
```

This prevents crawlers from crawling URLs under `/private/`.

## Common mistake

Do not confuse:

```text
Disallow
```

with:

```text
noindex
```

A `robots.txt` rule controls crawling. It is not a general replacement for a `noindex` directive.

### Troubleshooting checklist

* [ ] `robots.txt` exists and is reachable.
* [ ] Important pages are not accidentally blocked.
* [ ] CSS and JavaScript resources are not unnecessarily restricted.
* [ ] Rules are written correctly.
* [ ] A staging or development rule has not been copied to production.
* [ ] CDN or caching systems are not serving an outdated file.

After changing `robots.txt`, allow time for Google to recrawl the affected resources.

---

# Step 4: Check Noindex

A page may contain a `noindex` directive that tells search engines not to include the page in search results.

Example:

```html
<meta name="robots" content="noindex">
```

A page can also receive indexing directives through HTTP headers.

Example:

```text
X-Robots-Tag: noindex
```

## Check the page source

Search the HTML source for:

```text
noindex
```

Also inspect HTTP headers when necessary:

```bash
curl -I https://example.com/page/
```

Look for:

```text
X-Robots-Tag: noindex
```

## Common WordPress causes

* Search engine visibility setting enabled.
* SEO plugin configured with `noindex`.
* Individual post or page set to `noindex`.
* Taxonomy archive configured incorrectly.
* Plugin-generated headers.
* Staging settings copied to production.

Before removing `noindex`, confirm that the page is actually intended to appear in Google Search.

---

# Step 5: Check Canonical URLs

A canonical URL tells search engines which URL should generally represent a set of duplicate or very similar pages.

Example:

```html
<link rel="canonical" href="https://example.com/example-page/">
```

## Common canonical problems

* Canonical points to another page.
* Canonical points to an old URL.
* HTTP canonical on an HTTPS website.
* Canonical points to a redirected URL.
* Canonical points to a 404 page.
* Incorrect canonical generated by a plugin.
* Product variations or parameters create conflicting canonical signals.

### Compare these values

For the affected URL, compare:

```text
Requested URL
       ↓
Canonical declared by page
       ↓
Google-selected canonical
```

If Google selects a different canonical, investigate why before forcing a change.

Possible signals include:

* Internal links.
* Redirects.
* Sitemap URLs.
* Canonical tags.
* Duplicate content.
* Site architecture.

---

# Step 6: Check XML Sitemaps

An XML sitemap helps search engines discover URLs.

A common location is:

```text
https://example.com/sitemap.xml
```

WordPress SEO plugins may generate different sitemap structures, such as:

```text
/sitemap_index.xml
```

or:

```text
/post-sitemap.xml
/page-sitemap.xml
/product-sitemap.xml
```

## Sitemap checklist

* [ ] Sitemap URL works.
* [ ] Sitemap uses HTTPS when appropriate.
* [ ] URLs return valid responses.
* [ ] URLs represent canonical pages.
* [ ] Important indexable pages are included.
* [ ] Deleted URLs are removed.
* [ ] Redirected URLs are not unnecessarily included.
* [ ] `noindex` URLs are not included.
* [ ] The sitemap is submitted to Search Console.

### Example

```xml
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  <url>
    <loc>https://example.com/example-page/</loc>
  </url>
</urlset>
```

Do not manually create a huge sitemap if your CMS or SEO platform already generates one correctly.

---

# Step 7: Crawled but Not Indexed

One common Search Console status is:

```text
Crawled - currently not indexed
```

This means Google has crawled the URL but has not currently included it in the search index.

It does **not** necessarily mean that the site is broken.

## Investigate

Check:

* Is the page useful and unique?
* Is the content substantially similar to another URL?
* Is the canonical correct?
* Does the page have strong internal links?
* Is the content accessible without unnecessary restrictions?
* Is the page generated automatically with little useful content?
* Does the page return the correct HTTP status?
* Is the URL intentionally indexable?

### Avoid this mistake

Do not immediately submit the same URL for indexing repeatedly without fixing an underlying issue.

Instead:

```text
Identify
   ↓
Diagnose
   ↓
Improve
   ↓
Validate
   ↓
Request recrawl when appropriate
   ↓
Monitor
```

Google's systems may take time to recrawl and reprocess changes.

---

# Step 8: Discovered but Not Indexed

Another common status is:

```text
Discovered - currently not indexed
```

This generally means Google knows about the URL but has not crawled it yet.

Possible areas to investigate:

* Internal links.
* Sitemap submission.
* Site architecture.
* URL quality.
* Crawl demand.
* Server availability.
* Large numbers of low-value URLs.
* Duplicate URL generation.

## Improve discoverability

Use clear internal links:

```text
Homepage
   ↓
Category
   ↓
Subcategory
   ↓
Product / Article
```

Important pages should be reachable through the site's normal navigation or contextual internal links.

Do not create thousands of artificial links solely to force crawling.

---

# Step 9: Troubleshoot HTTP Status Codes

HTTP status codes are important during indexing investigations.

## 200 OK

The resource was successfully served.

Check whether:

* The content is correct.
* The page is indexable.
* The canonical is correct.

## 301 / 308 Redirect

A permanent redirect indicates that the requested URL has moved.

Check:

* Redirect destination.
* Redirect chain.
* Redirect loops.
* Whether the destination is the correct canonical URL.

## 302 / 307 Redirect

These generally represent temporary redirects.

Verify that the redirect type matches the intended behavior.

## 404 Not Found

The requested resource does not exist.

Check whether:

* The page was intentionally deleted.
* Internal links still point to it.
* The sitemap still contains it.
* A replacement page exists.

## 410 Gone

Indicates that the resource has been intentionally removed.

Use it only when appropriate.

## 500 / 503

Server-side errors can interfere with crawling.

Investigate:

* Web server logs.
* PHP errors.
* Application errors.
* Database availability.
* Resource exhaustion.
* CDN or proxy errors.
* Hosting-level restrictions.

---

# Step 10: Investigate Redirects

Redirect problems can create indexing and crawling issues.

### Example of a redirect chain

```text
http://example.com/page
        ↓
https://example.com/page
        ↓
https://www.example.com/page
        ↓
https://www.example.com/new-page
```

A simpler redirect structure is generally easier to maintain.

### Check redirects

```bash
curl -I http://example.com/page
```

Follow redirects when appropriate:

```bash
curl -IL http://example.com/page
```

Look for:

* Redirect loops.
* Long redirect chains.
* Incorrect destination URLs.
* HTTP/HTTPS conflicts.
* `www`/non-`www` inconsistencies.
* Redirects to unrelated pages.

---

# Step 11: Check Duplicate URLs

A website can accidentally expose the same content through multiple URLs.

Examples:

```text
https://example.com/product/
https://example.com/product/?ref=123
https://example.com/product/?utm_source=test
```

Other duplication sources include:

* HTTP and HTTPS versions.
* `www` and non-`www`.
* URL parameters.
* Product variations.
* Category pagination.
* Tag archives.
* Search pages.
* Printer-friendly URLs.
* Tracking URLs.

## Investigation

For duplicate URLs, check:

1. Canonical configuration.
2. Internal links.
3. Redirect rules.
4. Sitemap entries.
5. Parameter handling.
6. CMS-generated URLs.

Do not automatically block every parameterized URL in `robots.txt`. Determine how the URLs are actually being used first.

---

# Step 12: Check JavaScript Rendering

Modern websites may depend heavily on JavaScript.

Problems can occur when important content or links are not available as expected during Google's processing.

### Investigate

* JavaScript errors.
* Failed API requests.
* Client-side routing.
* Content loaded only after user interaction.
* Blocked resources.
* Authentication requirements.
* Incorrect server-side rendering.
* Broken hydration.

Check browser developer tools for errors such as:

```text
Failed to load resource
JavaScript error
Network request failed
404
403
500
```

If important content is rendered dynamically, make sure the page remains accessible and understandable to search engines and users.

---

# Step 13: WordPress-Specific Checks

WordPress sites can develop indexing problems because multiple plugins and settings can influence SEO behavior.

## Check WordPress Reading Settings

Navigate to:

```text
Settings → Reading
```

Review:

```text
Search engine visibility
```

Make sure production sites are not accidentally configured to discourage search engine indexing.

## Check SEO plugin settings

Common areas include:

* Global robots settings.
* Post/page `noindex`.
* Taxonomy indexing.
* Canonical URLs.
* Sitemap settings.
* Redirects.
* Attachment pages.
* Product archives.

## Check generated source

View the page source and search for:

```text
noindex
canonical
robots
```

## Check sitemap

Depending on the setup, WordPress may expose:

```text
/wp-sitemap.xml
```

or an SEO-plugin-generated sitemap.

Confirm that the submitted sitemap matches the site's actual SEO configuration.

## WooCommerce checks

For WooCommerce sites, investigate:

* Product URLs.
* Product categories.
* Product tags.
* Variations.
* Filter URLs.
* Search URLs.
* Cart and checkout pages.
* Account pages.
* Internal search results.

Not every WooCommerce-generated URL should be indexed.

---

# Step 14: Request Reindexing

After fixing an indexing problem, you can use Search Console's URL Inspection workflow to request crawling when the option is available.

A typical workflow is:

```text
Fix issue
   ↓
Verify page manually
   ↓
Inspect URL
   ↓
Confirm indexing signals
   ↓
Request indexing
   ↓
Wait for Google to recrawl
   ↓
Monitor status
```

Requesting indexing is not a guarantee that Google will immediately crawl or index the URL.

Google may take time to process changes.

Avoid repeatedly requesting indexing without making meaningful changes.

---

# Post-Fix Validation

After correcting an indexing issue, verify the complete URL rather than checking only one setting.

## Technical validation

* [ ] URL returns the expected HTTP status.
* [ ] HTTPS works correctly.
* [ ] No unexpected redirects occur.
* [ ] `robots.txt` does not unnecessarily block crawling.
* [ ] `noindex` is not present when indexing is intended.
* [ ] Canonical points to the intended URL.
* [ ] Sitemap contains the correct URL.
* [ ] Important internal links exist.
* [ ] Page loads correctly.
* [ ] JavaScript does not prevent important content from being accessed.

## Search Console validation

* [ ] URL inspected.
* [ ] Indexing status reviewed.
* [ ] Canonical information checked.
* [ ] Relevant sitemap status reviewed.
* [ ] Coverage/indexing reports monitored.
* [ ] Validation requested where applicable.

---

# Common Mistakes

## 1. Repeatedly requesting indexing

Submitting the same URL repeatedly does not replace fixing the underlying problem.

## 2. Blocking URLs with robots.txt without understanding the effect

A robots rule can prevent crawling when the actual goal was to prevent indexing or remove a page.

## 3. Using noindex everywhere

`noindex` should be intentional.

Do not add it to important pages simply because they are not ranking.

## 4. Changing canonical URLs blindly

Canonicalization should reflect the preferred URL and the site's actual structure.

## 5. Creating duplicate pages to target keywords

Creating many near-identical pages can create unnecessary URL and indexing problems.

## 6. Removing useful internal links

Internal links help search engines discover and understand pages.

## 7. Treating indexing as a ranking guarantee

A page can be indexed and still receive little or no organic traffic.

## 8. Making many changes simultaneously

Large numbers of simultaneous changes make troubleshooting difficult.

Whenever practical:

```text
Change
→ Test
→ Monitor
→ Document
```

---

# Indexing Troubleshooting Checklist

Use this quick checklist during a technical SEO investigation.

### URL

* [ ] Correct URL confirmed.
* [ ] Page loads normally.
* [ ] HTTP status checked.
* [ ] HTTPS works.
* [ ] Redirects checked.

### Crawling

* [ ] `robots.txt` checked.
* [ ] Important resources accessible.
* [ ] No accidental firewall restriction.
* [ ] Internal links checked.

### Indexing

* [ ] `noindex` checked.
* [ ] Canonical checked.
* [ ] Duplicate URLs investigated.
* [ ] Search Console URL Inspection completed.

### Sitemap

* [ ] Sitemap accessible.
* [ ] Sitemap submitted.
* [ ] Correct canonical URLs included.
* [ ] Broken or redirected URLs removed.

### Website

* [ ] Important content is accessible.
* [ ] JavaScript errors checked.
* [ ] Server errors checked.
* [ ] WordPress SEO settings checked.
* [ ] Plugin-generated SEO directives checked.

### After fixing

* [ ] Page manually tested.
* [ ] URL inspected again.
* [ ] Reindexing requested if appropriate.
* [ ] Search Console monitored.
* [ ] Changes documented.

---

# Prevention Best Practices

## Maintain a clean site architecture

Use:

* Logical categories.
* Descriptive URLs.
* Useful internal links.
* Consistent canonical URLs.
* Clean navigation.

## Maintain accurate sitemaps

Keep only URLs that are intended to be discoverable and indexable.

## Monitor Search Console

Regularly review:

* Indexing reports.
* Sitemap reports.
* URL Inspection.
* Manual actions.
* Security issues.
* Search performance.

## Monitor server health

Indexing problems can sometimes be symptoms of infrastructure problems.

Monitor:

* HTTP errors.
* Server availability.
* PHP errors.
* Database failures.
* CPU and memory usage.
* Disk space.
* CDN/proxy errors.

## Keep documentation

For production websites, maintain a record of:

```text
Date
↓
Problem
↓
Investigation
↓
Change made
↓
Validation
↓
Result
```

This makes future troubleshooting significantly easier.

---

# Quick Diagnostic Flow

```text
             URL not indexed
                    |
                    v
             Open the URL
                    |
             +------+------+
             |             |
           Works         Fails
             |             |
             v             v
       Check status     Fix server/
       and redirects    application issue
             |
             v
       Check robots.txt
             |
             v
        Check noindex
             |
             v
       Check canonical
             |
             v
       Check sitemap
             |
             v
       Check internal links
             |
             v
       Inspect in Search Console
             |
             v
      Fix underlying issue
             |
             v
       Request recrawl when
          appropriate
             |
             v
          Monitor
```

---

# Useful Commands

## Check HTTP headers

```bash
curl -I https://example.com/
```

## Follow redirects

```bash
curl -IL https://example.com/
```

## Check robots.txt

```bash
curl -s https://example.com/robots.txt
```

## Search response headers for indexing directives

```bash
curl -I https://example.com/page/ | grep -i "x-robots-tag"
```

## Check canonical from HTML

A simple approach is to download the page and inspect its source:

```bash
curl -Ls https://example.com/page/ | grep -i canonical
```

These commands are diagnostic examples. Always verify the output before changing production configuration.

---

# Official Resources

* [Google Search Central](https://developers.google.com/search)
* [Google Search Console](https://search.google.com/search-console)
* [URL Inspection Tool](https://support.google.com/webmasters/answer/9012289)
* [Google Search Central — Crawling and Indexing](https://developers.google.com/search/docs/crawling-indexing)
* [Google Search Central — Sitemaps](https://developers.google.com/search/docs/crawling-indexing/sitemaps/overview)
* [Google Search Central — Robots.txt](https://developers.google.com/search/docs/crawling-indexing/robots/intro)
* [Google Search Central — Canonicalization](https://developers.google.com/search/docs/crawling-indexing/consolidate-duplicate-urls)
* [Google Search Central — SEO Starter Guide](https://developers.google.com/search/docs/fundamentals/seo-starter-guide)

Google's documentation recommends using URL Inspection to understand how Google sees a page, and notes that pages may take time to be crawled and reprocessed after changes.

---

# Disclaimer

This repository is an independent technical SEO troubleshooting guide and is not affiliated with or endorsed by Google.

Search Console interfaces, reports, and Google Search behavior may change over time. Always consult Google's current documentation when troubleshooting production websites.

Indexing is not guaranteed. A technically accessible page may still not be indexed or ranked for a particular query.

Always test changes carefully before applying them to production websites.
