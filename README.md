<div align="center">

# 🚀 Ultimate SEO Guide for Vibe Coding

### *Master React + TypeScript SEO with Production-Ready Code Examples*

[![Vibe Coding](https://img.shields.io/badge/Vibe-Coding-purple?style=for-the-badge&logo=react)](https://github.com/markfulton/VibeCodingSEO)
[![React](https://img.shields.io/badge/React-18+-61DAFB?style=for-the-badge&logo=react)](https://reactjs.org/)
[![TypeScript](https://img.shields.io/badge/TypeScript-5+-3178C6?style=for-the-badge&logo=typescript)](https://www.typescriptlang.org/)
[![SEO](https://img.shields.io/badge/SEO-Optimized-green?style=for-the-badge&logo=google)](https://developers.google.com/search)

---

### 🎯 **Transform Your React Apps Into SEO Powerhouses**

This comprehensive guide provides **practical, production-ready blueprints** for maximizing organic visibility in React + TypeScript applications. From SSR strategies to Core Web Vitals optimization, get **drop-in code patterns** you can implement today.

### 🌟 **Join the Vibe Coding Community**

<table>
<tr>
<td align="center">
<a href="https://www.skool.com/vibe-coding-is-life/about">
<img src="https://img.shields.io/badge/🎓_Join_Skool-Vibe_Coding_Courses-FF6B35?style=for-the-badge" alt="Join Vibe Coding Skool">
</a>
<br>
<sub><b>Premium Courses & Live Sessions</b></sub>
</td>
<td align="center">
<a href="https://www.facebook.com/groups/vibecodinglife">
<img src="https://img.shields.io/badge/👥_Facebook_Group-71K+_Members-1877F2?style=for-the-badge&logo=facebook" alt="Join Facebook Group">
</a>
<br>
<sub><b>Largest Vibe Coding Community</b></sub>
</td>
</tr>
</table>

**🎓 [Vibe Coding is Life Skool](https://www.skool.com/vibe-coding-is-life/about)** - Access exclusive courses, training sessions, live builds, and project downloads

**👥 [Facebook Community](https://www.facebook.com/groups/vibecodinglife)** - Connect with 71,000+ developers in the largest Vibe Coding group

</div>

---

## 📋 Table of Contents

- [🎯 Core SEO Decisions That Move the Needle](#-core-seo-decisions-that-move-the-needle)
- [🏗️ Making React Indexable](#️-making-react-indexable)
- [🔗 URL Architecture & Routing](#-url-architecture--routing)
- [📊 Route-Level Meta Management](#-route-level-meta-management)
- [🏷️ Structured Data & JSON-LD](#️-structured-data--json-ld)
- [🤖 Crawling, Indexing & Sitemaps](#-crawling-indexing--sitemaps)
- [📄 Pagination & Infinite Scroll](#-pagination--infinite-scroll)
- [⚡ Core Web Vitals Optimization](#-core-web-vitals-optimization)
- [🌍 Internationalization](#-internationalization)
- [📱 Social Sharing Previews](#-social-sharing-previews)
- [🚀 Deployment & Headers](#-deployment--headers)
- [🔧 CI/CD SEO Guardrails](#-cicd-seo-guardrails)
- [📝 Quick Start Checklist](#-quick-start-checklist)
- [⚠️ Common Pitfalls & Solutions](#️-common-pitfalls--solutions)
- [🏢 CMS Integration Strategies](#-cms-integration-strategies)
- [🛠️ Framework Recommendations](#️-framework-recommendations)
- [📚 Additional Resources](#-additional-resources)

---

## 🎯 Core SEO Decisions That Move the Needle

> **React-specific decisions that maximize organic visibility**

### 1. 🏗️ **Render Content as HTML**
- **Prefer SSR/SSG or pre-rendering** so bots and users see real HTML fast
- Google explicitly recommends SSR/static rendering over "dynamic rendering" user-agent workarounds
- If you must ship a pure SPA, ensure every indexable view has a **unique URL that returns HTML**

### 2. 🔗 **Use Crawlable Links and Stable URLs**
- Google finds pages via `<a href="…">` — it **doesn't click buttons** to load more results
- Google **ignores `#` fragments** for unique content
- Use real `<a>` links and real URLs

### 3. 📄 **Canonicalization, Pagination & Parameters**
- Set the canonical for each page (absolute URL)
- Don't force every paginated URL to canonicalize to page 1
- For sorted/filtered variants, either prevent indexing or keep them crawl-eligible but non-indexable

### 4. 🏷️ **Structured Data (JSON-LD)**
- Use JSON-LD and follow Google's structured-data guidelines
- Support Organization/LocalBusiness, Product/Service, FAQ, Breadcrumbs
- If you generate it with JS, that's supported—just test with Rich Results

### 5. ⚡ **Core Web Vitals Are Ranking-Relevant**
- As of **March 12, 2024, INP replaced FID** as a Core Web Vital
- Monitor **LCP, CLS, INP** and optimize images, fonts, JS, and caching

### 6. 🤖 **Crawl Controls & Sitemaps**
- Use **meta robots / X-Robots-Tag** to control indexing
- **robots.txt cannot "noindex"** a page
- Keep **JS/CSS unblocked**
- Supply an XML sitemap (50k URLs / 50 MB max per file)

---

## 🏗️ Making React Indexable

### A. 🌟 **Best-Case: SSR/SSG (React 18 Streaming)**

If your platform allows Node on the edge/origin, React 18's `renderToPipeableStream` reduces time-to-HTML and improves LCP. Combine with route-level meta and HTTP caching.

### B. 📱 **Pure SPA (Vite + React Router) with Pre-render**

If SSR is truly impossible, **pre-render** public routes at build time:

- ✅ Each page has a **static HTML shell** (title/description/LD+JSON)
- ✅ Navigation uses real `<a href>` links (no button-driven routing)

---

## 🔗 URL Architecture & Routing

### 🎯 **Essential Rules**

- ❌ **No hash routing** — bots ignore `#` for uniqueness. Use History API URLs
- ✅ Every indexable view must have a **unique, shareable URL**
- ✅ Choose one hostname (e.g., `https://www.example.com`), enforce **HTTPS + 301s**
- ✅ Normalize trailing slashes/case; produce a single canonical per page
- ✅ **Use absolute canonicals**

---

## 📊 Route-Level Meta Management

### 🛠️ **Robust React + TypeScript Pattern**

Use **React Router v6 "data routers"** with a `handle.seo` contract and **react-helmet-async** to generate unique metadata per route.

```tsx
// src/seo/SEO.tsx
import { Helmet } from 'react-helmet-async';

export type SEOConfig = {
  title?: string;
  description?: string;
  canonical?: string;           // absolute or site-relative
  robots?: string;              // e.g., "index,follow" or "noindex"
  og?: { type?: string; image?: string | null };
  twitter?: { card?: 'summary' | 'summary_large_image'; site?: string; creator?: string };
  schema?: Record<string, unknown> | Record<string, unknown>[];
};

const SITE_URL = 'https://www.example.com';

export function SEO(cfg: SEOConfig) {
  const {
    title, description, robots = 'index,follow', og = {}, twitter = {}, schema
  } = cfg;
  const canonicalUrl =
    cfg.canonical?.startsWith('http')
      ? cfg.canonical
      : `${SITE_URL}${cfg.canonical || (typeof window !== 'undefined' ? window.location.pathname : '/')}`;

  return (
    <Helmet prioritizeSeoTags>
      {title && <title>{title}</title>}
      {description && <meta name="description" content={description} />}
      <link rel="canonical" href={canonicalUrl} />
      <meta name="robots" content={robots} />

      {/* Open Graph / Twitter */}
      <meta property="og:type" content={og.type ?? 'website'} />
      <meta property="og:url" content={canonicalUrl} />
      {title && <meta property="og:title" content={title} />}
      {description && <meta property="og:description" content={description} />}
      {og.image && <meta property="og:image" content={og.image} />}
      <meta name="twitter:card" content={twitter.card ?? 'summary_large_image'} />
      {twitter.site && <meta name="twitter:site" content={twitter.site} />}
      {twitter.creator && <meta name="twitter:creator" content={twitter.creator} />}

      {/* JSON-LD */}
      {schema && (Array.isArray(schema) ? schema : [schema]).map((obj, i) => (
        <script key={i} type="application/ld+json"
                dangerouslySetInnerHTML={{ __html: JSON.stringify(obj) }} />
      ))}
    </Helmet>
  );
}
```

### 🔧 **Router Configuration Example**

```tsx
// src/routes.tsx (React Router v6 data router)
import { createBrowserRouter, useMatches, Outlet } from 'react-router-dom';
import type { SEOConfig } from './seo/SEO';
import { SEO } from './seo/SEO';

type SEOHandle = { seo: SEOConfig | ((data: any) => SEOConfig) };

function RootLayout() {
  const matches = useMatches() as Array<{ handle?: SEOHandle; data?: unknown }>;
  const last = matches.at(-1);
  const cfg = typeof last?.handle?.seo === 'function'
    ? last?.handle?.seo(last?.data)
    : last?.handle?.seo;

  return (
    <>
      {cfg && <SEO {...cfg} />}
      <Outlet />
    </>
  );
}

// Example page-specific configs
export const router = createBrowserRouter([
  {
    path: '/',
    element: <RootLayout />,
    children: [
      {
        index: true,
        element: <HomePage />,
        handle: { 
          seo: { 
            title: 'Acme Co. — Enterprise Widgets', 
            description: 'High-reliability widgets.', 
            canonical: '/' 
          } 
        }
      },
      {
        path: '/services',
        element: <ServicesPage />,
        handle: { 
          seo: { 
            title: 'Services | Acme Co.', 
            description: 'Advisory & implementation.', 
            canonical: '/services' 
          } 
        }
      },
      {
        path: '/products/:slug',
        loader: productLoader,
        element: <ProductPage />,
        handle: {
          seo: (data: Awaited<ReturnType<typeof productLoader>>) => ({
            title: `${data.product.name} | Acme Co.`,
            description: data.product.seoDescription,
            canonical: `/products/${data.product.slug}`,
            og: { type: 'product', image: data.product.ogImage ?? null },
            schema: productJsonLd(data.product)
          })
        }
      }
    ]
  }
]);
```

> 💡 **Why react-helmet-async?** It's the maintained, SSR-safe way to manage `<head>` in React apps. [GitHub Repository](https://github.com/staylor/react-helmet-async)

---

## 🏷️ Structured Data & JSON-LD

### 🎯 **Typed, Reusable JSON-LD Helpers**

Use TypeScript helpers so each page can attach the right schema object(s). Validate with the Rich Results Test.

```ts
// src/seo/schema.ts
export type Thing = Record<string, unknown>;

export const orgJsonLd = (name: string, url: string, logo: string): Thing => ({
  '@context': 'https://schema.org',
  '@type': 'Organization',
  name, url, logo
});

export const localBusinessJsonLd = (opts: {
  name: string; url: string; phone?: string;
  address: { 
    streetAddress: string; 
    addressLocality: string; 
    addressRegion: string; 
    postalCode: string; 
    addressCountry: string 
  };
}): Thing => ({
  '@context': 'https://schema.org',
  '@type': 'LocalBusiness',
  ...opts
});

export const breadcrumbJsonLd = (items: Array<{ name: string; url: string }>): Thing => ({
  '@context': 'https://schema.org',
  '@type': 'BreadcrumbList',
  itemListElement: items.map((it, i) => ({
    '@type': 'ListItem', 
    position: i + 1, 
    name: it.name, 
    item: it.url
  }))
});

export const productJsonLd = (p: {
  name: string; description: string; url: string; image: string;
  sku?: string; brand?: string;
  offers?: { price: number; priceCurrency: string; availability: string; url: string };
}): Thing => ({
  '@context': 'https://schema.org',
  '@type': 'Product',
  ...p
});
```

> 🏢 **Local businesses** should also consider the LocalBusiness guide (hours, phone, geo). If you generate JSON-LD with JavaScript, that's supported—just test it.

---

## 🤖 Crawling, Indexing & Sitemaps

### 🎛️ **The Right Levers**

- ✅ **Use robots meta or X-Robots-Tag** to control indexing (`noindex`, `max-image-preview`, etc.)
- ❌ Do **not** try to "noindex" in robots.txt; that's unsupported
- ✅ Keep **JS/CSS accessible** — don't block them in robots.txt
- ✅ **Sitemaps**: one file ≤ 50,000 URLs or 50 MB (uncompressed)

### 📄 **Example robots.txt**

```txt
User-agent: *
Allow: /

# Keep assets crawlable (don't disallow CSS/JS)
# Disallow private or thin areas
Disallow: /admin/
Disallow: /cart
Disallow: /checkout
Disallow: /*?*sort=
Disallow: /*?*filter=

Sitemap: https://www.example.com/sitemap.xml
```

### 🗺️ **TypeScript Sitemap Generator**

```ts
// scripts/generate-sitemap.ts
import { createWriteStream } from 'node:fs';
import { SitemapStream, streamToPromise } from 'sitemap';
import fetch from 'node-fetch';

const HOST = 'https://www.example.com';

async function main() {
  const sm = new SitemapStream({ hostname: HOST });
  sm.pipe(createWriteStream('public/sitemap.xml'));

  // Static pages
  ['/', '/about', '/services', '/contact'].forEach(url =>
    sm.write({ url, changefreq: 'monthly', priority: 0.7 })
  );

  // Dynamic (e.g., from CMS/API)
  const products = await fetch(`${HOST}/api/public/products`).then(r => r.json());
  products.forEach((p: any) => 
    sm.write({ url: `/products/${p.slug}`, lastmod: p.updatedAt })
  );

  sm.end();
  await streamToPromise(sm);
  console.log('sitemap.xml written');
}

main().catch(err => { console.error(err); process.exit(1); });
```

> ⚠️ **Status codes matter**: Unknown routes must return **404**, retired pages can be **410**, and avoid "soft-404s" (HTML saying "not found" with HTTP 200).

---

## 📄 Pagination & Infinite Scroll

### 🔗 **What Google Expects**

- ✅ Use real links (`<a href="?page=2">Next</a>`) so crawlers can discover subsequent pages
- ✅ Give each page a **unique URL** (e.g., `?page=n`) and **its own canonical**
- ❌ Don't canonical all pages to page 1
- ℹ️ Google no longer uses `rel="prev/next"` for indexing—focus on links, canonicals, and sitemaps

---

## ⚡ Core Web Vitals Optimization

### 🎯 **What Changed: INP Replaced FID (March 12, 2024)**

Target passing performance at the 75th percentile for **LCP, CLS, INP**.

### 🖼️ **Images (Often Your Biggest LCP Lever)**

```html
<!-- In <head> -->
<link rel="preload" as="image" href="/images/hero.avif"
      imagesrcset="/images/hero.avif 1x, /images/hero@2x.avif 2x" 
      imagesizes="100vw">

<!-- In body -->
<img src="/images/hero.avif" alt="Acme industrial widgets"
     width="1280" height="720" 
     fetchpriority="high" 
     decoding="async">
```

**Best Practices:**
- ✅ Serve responsive sources via `srcset`/`sizes`
- ✅ Prefer **AVIF / WebP** fallbacks
- ✅ **Preload the hero image** and/or use `fetchpriority="high"`
- ✅ Lazy-load **below-the-fold** images (`loading="lazy"`)
- ❌ **Not** the LCP image

### 🔤 **Fonts**

```html
<link rel="preload" href="/fonts/Inter-var.woff2" as="font" type="font/woff2" crossorigin>
<style>
  @font-face {
    font-family: 'InterVar';
    src: url('/fonts/Inter-var.woff2') format('woff2');
    font-weight: 100 900; 
    font-display: optional;
  }
  body { 
    font-family: 'InterVar', system-ui, -apple-system, Segoe UI, Roboto, sans-serif; 
  }
</style>
```

**Best Practices:**
- ✅ Self-host WOFF2, subset fonts
- ✅ Use `font-display: optional` + `<link rel="preload" as="font">`
- ✅ Avoid FOIT/FOUT and CLS

### 📊 **Measure in the Field (RUM)**

```ts
// src/metrics/web-vitals.ts
import { onLCP, onCLS, onINP, type Metric } from 'web-vitals';

const send = (m: Metric) =>
  navigator.sendBeacon('/vitals', JSON.stringify({ 
    name: m.name, 
    value: m.value, 
    id: m.id, 
    path: location.pathname 
  }));

onLCP(send); 
onCLS(send); 
onINP(send);
```

> 📈 **Google recommends** collecting Web Vitals in the field; the `web-vitals` library is the easiest way to do that.

---

## 🌍 Internationalization

### 🗺️ **Multi-Language Setup**

- ✅ Use **separate URLs per language/region** (e.g., `/en/`, `/fr/`)
- ✅ Add **`hreflang`** cross-links between alternates
- ✅ Include a **self-reference** on each page
- ❌ Don't rely on `lang` alone

---

## 📱 Social Sharing Previews

### 🎨 **Open Graph & Twitter Cards**

Add page-specific `og:title`, `og:description`, `og:image` for high CTR, and Twitter card meta. Use the Open Graph protocol as the baseline. (The `SEO` component above handles this.)

---

## 🚀 Deployment & Headers

### 🔧 **Clean Signals**

- **Redirects**: Permanent site-wide 301 from non-canonical host to canonical, HTTP→HTTPS, trailing slash policy
- **Caching**: 
  - Hashed static assets → `Cache-Control: public, max-age=31536000, immutable`
  - HTML → `no-store` or `max-age=0, must-revalidate`
- **Security**: HSTS (helps consistency on HTTPS)
- **Favicons & Site Names**: Follow Google's favicon and site-name guidelines

---

## 🔧 CI/CD SEO Guardrails

### 🛡️ **Prevent SEO Regression**

- ✅ **Lighthouse CI** budget in CI (fail builds if LCP/CLS/INP regress)
- ✅ **ESLint** with `jsx-a11y` plugin (accessibility helps semantics)
- ✅ **Link checker** (broken internal links)
- ✅ **Tests** that read route configs and assert each public page has title/description/canonical and JSON-LD

---

## 📝 Quick Start Checklist

### 🚀 **Day 1 Implementation**

- [ ] Pick rendering path: SSR/SSG if possible; otherwise pre-render public routes
- [ ] Implement the `SEO` component + `handle.seo` per route
- [ ] Add Organization (and LocalBusiness/Product/etc.) JSON-LD to relevant pages
- [ ] Ship `sitemap.xml`, `robots.txt` (with sitemap directive), and correct status codes/404s
- [ ] Optimize LCP (hero image preload), fonts, and reduce JS
- [ ] Start collecting Web Vitals with `web-vitals`
- [ ] If multilingual, wire `hreflang`

---

## ⚠️ Common Pitfalls & Solutions

### 🚨 **Avoid These Mistakes**

- ❌ **SPA 404s returning HTTP 200** due to "catch-all" rewrites
  - ✅ Configure your host to serve real `404.html` with status 404

- ❌ **Faceted URLs exploding crawl**
  - ✅ Keep pages crawlable for users but **noindex** values you don't want in the index

- ❌ **Dynamic rendering** for bots only
  - ✅ Use SSR/SSG or pre-render for everyone instead

---

## 🏢 CMS Integration Strategies

### A. 🎯 **WordPress (Root) + React App (Subdomain)**

**What:** Keep all public, indexable content on WordPress at `example.com`. Put the authenticated product at `app.example.com`.

**Why it works for SEO:**
- WordPress gives non-devs fast publishing + mature SEO plugins
- Google treats **subdomains as valid site partitions**
- You can **noindex** app/login/etc. using `meta robots` or `X-Robots-Tag` header

### B. 🔄 **Same Domain, Different Origins: Reverse Proxy**

**What:** Keep marketing at `example.com` and mount the React app at `example.com/app` via **reverse proxy/rewrites**.

**Why:** Preserves one hostname and **consolidates signals** under the root.

### C. 🎭 **Headless WordPress + React Front End**

**What:** WordPress only for content editing/API; public site is built with a React framework that **pre-renders** pages.

**Benefits:** HTML is ready on first byte (great for crawl & Core Web Vitals) + you retain WordPress editorial workflow.

---

## 🛠️ Framework Recommendations

### 🏆 **SEO-Forward React Frameworks**

| Framework | Why it's Strong for SEO | Signature Features |
|-----------|------------------------|-------------------|
| **Next.js (App Router)** | SSR/SSG by default, Server Components + streaming ship HTML early | `generateMetadata`, file-based robots/sitemap, ISR |
| **React Router v7** | SSR + pre-rendering for static URLs, progressive enhancement | Route-level `<Meta/>` + prerender config |
| **TanStack Start** | Full-document SSR, streaming, Selective SSR per route | Per-route `ssr` controls, streaming SSR |
| **Astro (with React)** | Islands architecture → static HTML by default, hydrate only interactive components | React integration, islands (`client:*` directives) |
| **Gatsby** | Mature SSG with DSG + optional SSR, huge source plugin ecosystem | DSG/SSR rendering options, CMS integrations |

### 🎯 **Choosing the Right Path**

- **Strong editorial + blogs/docs?** → **Astro** or **Headless WP + Next/Gatsby**
- **One React codebase for everything?** → **Next.js App Router**
- **Prefer edge runtime + progressive enhancement?** → **React Router (v7)** or **Next on Edge**
- **Want to dial SSR per route?** → **TanStack Start**
- **E-commerce on Shopify?** → **Hydrogen**

---

## 📚 Additional Resources

### 🔗 **Key Google Documentation**

- [JavaScript SEO Basics](https://developers.google.com/search/docs/crawling-indexing/javascript/javascript-seo-basics) - Google's official JS SEO guidance
- [Dynamic Rendering as a Workaround](https://developers.google.com/search/docs/crawling-indexing/javascript/dynamic-rendering) - Why to prefer SSR/static
- [Pagination Best Practices](https://developers.google.com/search/docs/specialty/ecommerce/pagination-and-incremental-page-loading) - Crawlable links, unique URLs
- [Robots Meta Tag](https://developers.google.com/search/docs/crawling-indexing/robots-meta-tag) - Control indexing properly
- [Sitemaps Guide](https://developers.google.com/search/docs/crawling-indexing/sitemaps/build-sitemap) - Build and submit sitemaps

### 🚀 **Performance Resources**

- [INP Became a Core Web Vital](https://web.dev/blog/inp-cwv-march-12) - March 12, 2024 update
- [Optimize LCP](https://web.dev/articles/optimize-lcp) - Preload critical images
- [Optimize Web Fonts](https://web.dev/learn/performance/optimize-web-fonts) - Preload + font-display: optional
- [Measure Web Vitals](https://web.dev/articles/vitals-field-measurement-best-practices) - Field measurement best practices

### 🏗️ **Technical Implementation**

- [Open Graph Protocol](https://ogp.me/) - Social preview reference
- [Structured Data Guidelines](https://developers.google.com/search/docs/appearance/structured-data/sd-policies) - JSON-LD best practices
- [React Helmet Async](https://github.com/staylor/react-helmet-async) - SSR-safe head management

---

<div align="center">

### 🌟 **Ready to Level Up Your Vibe Coding Skills?**

Join thousands of vibe coders mastering coding and deploying with AI!

[![Join Vibe Coding is Life Skool](https://img.shields.io/badge/🎓_Join_Skool-Premium_Courses-FF6B35?style=for-the-badge)](https://www.skool.com/vibe-coding-is-life/about)
[![Join Facebook Group](https://img.shields.io/badge/👥_Facebook-71K+_Developers-1877F2?style=for-the-badge&logo=facebook)](https://www.facebook.com/groups/vibecodinglife)

**Made with ❤️ by [Mark Fulton](https://markfulton.com/)**

</div>
