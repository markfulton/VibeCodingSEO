Below is a **practical, React + TypeScript–specific blueprint** you can adopt end‑to‑end to maximize organic visibility while staying within a pure React stack. I’ve organized it by decisions that matter most, then included **drop‑in TypeScript patterns** (meta, structured data, sitemaps/robots, Web Vitals) you can paste into your codebase.

---

## 0) The decisions that move the needle (React‑specific)

1. **Render content as HTML** wherever you can
    
    - Prefer **SSR/SSG or pre‑rendering** so bots and users see real HTML fast. Google explicitly recommends SSR/static rendering over “dynamic rendering” user‑agent workarounds (which they no longer recommend).
    
    - If you must ship a pure SPA, make sure every indexable view has a **unique URL that returns HTML** (see §2 and §6). Google can execute JS, but you must still expose crawlable links and avoid interaction‑gated content. 
        
2. **Use crawlable links and stable URLs**
    
    - Google finds pages via `<a href="…">`—it **doesn’t click buttons** to load more results—and it **ignores `#` fragments** for unique content. Use real `<a>` links and real URLs. 
        
3. **Canonicalization, pagination & parameters**
    
    - Set the canonical for each page (absolute URL); don’t force every paginated URL to canonicalize to page 1. 
        
    - For sorted/filtered variants, either prevent indexing or keep them crawl‑eligible but non‑indexable (see §5). 
        
4. **Structured data** (JSON‑LD) for key page types
    
    - Use JSON‑LD and follow Google’s structured‑data guidelines (Organization/LocalBusiness, Product/Service, FAQ, Breadcrumbs). If you generate it with JS, that’s supported—just test with Rich Results. 
        
5. **Core Web Vitals** are ranking‑relevant quality signals
    
    - As of **March 12, 2024, INP replaced FID** as a Core Web Vital. Monitor **LCP, CLS, INP** and optimize images, fonts, JS, and caching. 
        
6. **Crawl controls & sitemaps (not robots “noindex”)**
    
    - Use **meta robots / X‑Robots‑Tag** to control indexing; **robots.txt cannot “noindex”** a page. Keep **JS/CSS unblocked**. Supply an XML sitemap (50k URLs / 50 MB max per file).
        

---

## 1) Make React indexable reliably (even if you can’t use Next/Remix)

### A. Best‑case: SSR/SSG (React 18 streaming)

If your platform allows Node on the edge/origin, React 18’s `renderToPipeableStream` reduces time‑to‑HTML and improves LCP. Combine with route‑level meta (below) and HTTP caching. 

### B. Pure SPA (Vite + React Router) with pre‑render

If SSR is truly impossible, **pre‑render** public routes at build time (marketing pages, product/category lists, blog posts). Ensure:

- Each page has a **static HTML shell** (title/description/LD+JSON).
    
- Navigation uses real `<a href>` links (no button‑driven routing).
    

---

## 2) URL architecture & routing rules

- **No hash routing**—bots ignore `#` for uniqueness. Use History API URLs. 
    
- Every indexable view must have a **unique, shareable URL**.
    
- Choose one hostname (e.g., `https://www.example.com`), enforce **HTTPS + 301s** from the alternate (www ↔ non‑www).
    
- Normalize trailing slashes/case; produce a single canonical per page. **Use absolute canonicals**. 
    

---

## 3) Route‑level meta: a robust React + TS pattern

Use **React Router v6 “data routers”** with a `handle.seo` contract and **react‑helmet‑async** to generate unique metadata per route (including OG/Twitter and JSON‑LD).

```tsx
// src/seo/SEO.tsx
import { Helmet } from 'react-helmet-async';

export type SEOConfig = {
  title?: string;
  description?: string;
  canonical?: string;           // absolute or site‑relative
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

      {/* JSON‑LD */}
      {schema && (Array.isArray(schema) ? schema : [schema]).map((obj, i) => (
        <script key={i} type="application/ld+json"
                dangerouslySetInnerHTML={{ __html: JSON.stringify(obj) }} />
      ))}
    </Helmet>
  );
}
```

```tsx
// src/routes.tsx  (React Router v6 data router)
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
        handle: { seo: { title: 'Acme Co. — Enterprise Widgets', description: 'High‑reliability widgets.', canonical: '/' } }
      },
      {
        path: '/services',
        element: <ServicesPage />,
        handle: { seo: { title: 'Services | Acme Co.', description: 'Advisory & implementation.', canonical: '/services' } }
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

> Why `react-helmet-async`? It’s the maintained, SSR‑safe way to manage `<head>` in React apps. ([GitHub](https://github.com/staylor/react-helmet-async "GitHub - staylor/react-helmet-async: Thread-safe Helmet for ..."))  
> For OG markup, follow the Open Graph spec; for Twitter summaries, use the standard card meta. ([Open Graph Protocol](https://ogp.me/ "The Open Graph protocol"))

---

## 4) Structured data: typed, reusable JSON‑LD helpers

Use TS helpers so each page can attach the right schema object(s). Validate with the Rich Results Test and follow Google’s structured‑data policies. 

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
  address: { streetAddress: string; addressLocality: string; addressRegion: string; postalCode: string; addressCountry: string };
}): Thing => ({
  '@context': 'https://schema.org',
  '@type': 'LocalBusiness',
  ...opts
});

export const breadcrumbJsonLd = (items: Array<{ name: string; url: string }>): Thing => ({
  '@context': 'https://schema.org',
  '@type': 'BreadcrumbList',
  itemListElement: items.map((it, i) => ({
    '@type': 'ListItem', position: i + 1, name: it.name, item: it.url
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

> Local businesses should also consider the LocalBusiness guide (hours, phone, geo). If you generate JSON‑LD with JavaScript, that’s supported—just test it. 

---

## 5) Crawling, indexing, and sitemaps (the right levers)

- **Use robots meta or X‑Robots‑Tag** to control indexing (`noindex`, `max-image-preview`, etc.). Do **not** try to “noindex” in robots.txt; that’s unsupported. 
    
- Keep **JS/CSS accessible**—don’t block them in robots.txt; many JS‑related search issues come from blocked resources.
    
- **Sitemaps**: one file ≤ 50,000 URLs or 50 MB (uncompressed). Use multiple sitemaps plus an index when needed. Add your sitemap URL to `robots.txt`. 
    

**Example `robots.txt` (adjust paths):**

```
User-agent: *
Allow: /

# Keep assets crawlable (don’t disallow CSS/JS)
# Disallow private or thin areas
Disallow: /admin/
Disallow: /cart
Disallow: /checkout
Disallow: /*?*sort=
Disallow: /*?*filter=

Sitemap: https://www.example.com/sitemap.xml
```

**Minimal TypeScript sitemap generator** (drop in a build step):

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
  products.forEach((p: any) => sm.write({ url: `/products/${p.slug}`, lastmod: p.updatedAt }));

  sm.end();
  await streamToPromise(sm);
  console.log('sitemap.xml written');
}

main().catch(err => { console.error(err); process.exit(1); });
```

**Status codes matter**: unknown routes must return **404**, retired pages can be **410**, and avoid “soft‑404s” (HTML saying “not found” with HTTP 200).

---

## 6) Pagination, “Load more,” infinite scroll—what Google expects

- Use real links (`<a href="?page=2">Next</a>`) so crawlers can discover subsequent pages.
    
- Give each page a **unique URL** (e.g., `?page=n`) and **its own canonical**. Don’t canonical all pages to page 1.
    
- Google no longer uses `rel="prev/next"` for indexing—focus on links, canonicals, and sitemaps.
    

---

## 7) Performance: win Core Web Vitals (LCP, CLS, INP)

**What changed**: INP replaced FID as a Core Web Vital on **March 12, 2024**. Target passing performance at the 75th percentile. 

**Implementation checklist**

- **Images (often your biggest LCP lever)**
    
    - Serve responsive sources via `srcset`/`sizes`, prefer **AVIF / WebP** fallbacks.
        
    - **Preload the hero image** and/or use `fetchpriority="high"` on the LCP image; consider `decoding="async"`.
        
    
    ```html
    <!-- In <head> -->
    <link rel="preload" as="image" href="/images/hero.avif"
          imagesrcset="/images/hero.avif 1x, /images/hero@2x.avif 2x" imagesizes="100vw">
    <!-- In body -->
    <img src="/images/hero.avif" alt="Acme industrial widgets"
         width="1280" height="720" fetchpriority="high" decoding="async">
    ```
    
    - Lazy‑load **below‑the‑fold** images (`loading="lazy"`), but **not** the LCP image.
        
- **Fonts**
    
    - Self‑host WOFF2, subset, and **`font-display: optional` + `<link rel="preload" as="font">`** to avoid FOIT/FOUT and CLS. 
        
    
    ```html
    <link rel="preload" href="/fonts/Inter-var.woff2" as="font" type="font/woff2" crossorigin>
    <style>
      @font-face {
        font-family: 'InterVar';
        src: url('/fonts/Inter-var.woff2') format('woff2');
        font-weight: 100 900; font-display: optional;
      }
      body { font-family: 'InterVar', system-ui, -apple-system, Segoe UI, Roboto, sans-serif; }
    </style>
    ```
    
- **JavaScript & CSS**
    
    - Code‑split and lazy‑load non‑critical bundles; minimize third‑party JS.
        
    - Inline tiny critical CSS; defer the rest.
        
    - Keep CSS/JS **unblocked for crawlers**. 
        
- **Measure in the field** (RUM)
    
    ```ts
    // src/metrics/web-vitals.ts
    import { onLCP, onCLS, onINP, type Metric } from 'web-vitals';
    const send = (m: Metric) =>
      navigator.sendBeacon('/vitals', JSON.stringify({ name: m.name, value: m.value, id: m.id, path: location.pathname }));
    onLCP(send); onCLS(send); onINP(send);
    ```
    
    > Google recommends collecting Web Vitals in the field; the `web‑vitals` library is the easiest way to do that. 
    

---

## 8) Internationalization (if you have multiple languages)

- Use **separate URLs per language/region** (e.g., `/en/`, `/fr/`).
    
- Add **`hreflang`** cross‑links between alternates and include a **self‑reference** on each page; don’t rely on `lang` alone. 
    

---

## 9) Social sharing previews (OG/Twitter)

- Add page‑specific `og:title`, `og:description`, `og:image` for high CTR, and Twitter card meta. Use the Open Graph protocol as the baseline. (The `SEO` component above handles this.)
    

---

## 10) Deployment & headers (clean signals)

- **Redirects**: permanent site‑wide 301 from the non‑canonical host to canonical, HTTP→HTTPS, trailing slash policy.
    
- **Caching**: hashed static assets → `Cache-Control: public, max-age=31536000, immutable`; HTML → `no-store` or `max-age=0, must-revalidate`.
    
- **Security**: HSTS (helps consistency on HTTPS).
    
- **Favicons & site names**: follow Google’s favicon and site‑name guidelines so results show your correct icon and name.
    

---

## 11) CI/CD guardrails so SEO can’t regress

- **Lighthouse CI** budget in CI (fail builds if LCP/CLS/INP regress).
    
- **ESLint** with `jsx-a11y` plugin (accessibility helps semantics).
    
- **Link checker** (broken internal links).
    
- **Tests** that read route configs and assert each public page has title/description/canonical and, where appropriate, JSON‑LD.
    

---

## 12) Quick reference — what to do on day 1

-  Pick rendering path: SSR/SSG if possible; otherwise pre‑render public routes (and keep links crawlable).
    
-  Implement the `SEO` component + `handle.seo` per route (above).
    
-  Add Organization (and LocalBusiness/Product/etc.) JSON‑LD to the relevant pages.
    
-  Ship `sitemap.xml`, `robots.txt` (with sitemap directive), and correct status codes/404s. 
    
-  Optimize LCP (hero image preload), fonts, and reduce JS; start collecting Web Vitals with `web‑vitals`. 
    
-  If multilingual, wire `hreflang`.
    

---

### Notes on a few common pitfalls (and fixes)

- **SPA 404s returning HTTP 200** due to “catch‑all” rewrites: configure your host to serve real `404.html` with status 404. 
    
- **Faceted URLs exploding crawl**: keep pages crawlable for users but **noindex** values you don’t want in the index; expose a **View All** (or a canonical base) where appropriate.
    
- **Dynamic rendering** for bots only: don’t—use SSR/SSG or pre‑render for everyone instead. 
    

---

Adopting the patterns above—**route‑level meta + JSON‑LD, crawlable links/URLs, correct canonicals/sitemaps, and Core Web Vitals discipline**—will give your React + TypeScript site an excellent foundation for search.

---

### Sources (key guidance)

- Google on **JS SEO basics**, rendering & indexing. ([Google for Developers](https://developers.google.com/search/docs/crawling-indexing/javascript/javascript-seo-basics?rd=1&visit_id=638120923109115860-3890602092 "Understand JavaScript SEO Basics | Google Search Central ..."))
    
- Google: **Dynamic rendering is a workaround** (prefer SSR/static). ([Google for Developers](https://developers.google.com/search/docs/crawling-indexing/javascript/dynamic-rendering "Dynamic Rendering as a workaround | Google Search Central ..."))
    
- web.dev: **Rendering on the web** (prefer SSR/static rendering). ([web.dev](https://web.dev/articles/rendering-on-the-web "Rendering on the Web | Articles | web.dev"))
    
- Google: **Pagination/infinite scroll**—crawlable links, unique URLs; Google **no longer uses `rel=prev/next`**. ([Google for Developers](https://developers.google.com/search/docs/specialty/ecommerce/pagination-and-incremental-page-loading "Pagination Best Practices for Google | Google Search Central  |  Documentation  |  Google for Developers"))
    
- Google: **Robots meta & X‑Robots‑Tag** (and that robots.txt can’t “noindex”). ([Google for Developers](https://developers.google.com/search/docs/crawling-indexing/robots-meta-tag?hl=ja&utm_source=chatgpt.com "Robots meta タグ、data-nosnippet、X-Robots-Tag の設定 ..."))
    
- Google: **Sitemaps** (50 MB / 50k URLs per file, use index). ([Google for Developers](https://developers.google.com/search/docs/crawling-indexing/sitemaps/build-sitemap "Build and Submit a Sitemap | Google Search Central ..."))
    
- web.dev: **INP became a Core Web Vital** on Mar 12, 2024. ([web.dev](https://web.dev/blog/inp-cwv-march-12 "Interaction to Next Paint becomes a Core Web Vital on March 12  |  Blog  |  web.dev"))
    
- web.dev: **Optimize LCP**; preload critical images; image performance patterns. ([web.dev](https://web.dev/articles/optimize-lcp "Optimize Largest Contentful Paint | Articles | web.dev"))
    
- web.dev: **Optimize web fonts**; **preload + `font-display: optional`**. ([web.dev](https://web.dev/learn/performance/optimize-web-fonts "Optimize web fonts | web.dev"))
    
- web.dev: **Measure Web Vitals in the field** (`web‑vitals` library). ([web.dev](https://web.dev/articles/vitals-field-measurement-best-practices "Best practices for measuring Web Vitals in the field"))
    
- Google: **Structured data** general guidelines; **LocalBusiness** reference; generating LD+JSON with JS. ([Google for Developers](https://developers.google.com/search/docs/appearance/structured-data/sd-policies "General Structured Data Guidelines | Google Search Central ..."))
    
- Open Graph protocol reference (for social previews). ([Open Graph Protocol](https://ogp.me/ "The Open Graph protocol"))
    
