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
    
---


## CMS for Public Route

## 1) Split the surface: CMS for public pages, React for the app

### A) **Marketing on WordPress (root), private React app on `app.` subdomain**

- **What:** Keep all public, indexable content (home, pricing, features, docs, blog) on WordPress at `example.com/…`. Put the authenticated product at `app.example.com`.
    
- **Why it works for SEO**
    
    - WordPress gives non‑devs fast publishing + mature SEO plugins, sitemaps, and content modeling.
        
    - Google treats **subdomains as valid site partitions**; if you want to see aggregate data, add a **Domain Property** in Search Console to include all subdomains/protocols. ([Google Help](https://support.google.com/webmasters/answer/34592?hl=en&utm_source=chatgpt.com "Add a website property to Search Console - Search Console Help"))
        
    - You can **noindex** app/login/etc. using `meta robots` or the `X‑Robots‑Tag` header (don’t try to “noindex” in robots.txt). ([Google for Developers](https://developers.google.com/search/docs/crawling-indexing/block-indexing?hl=ja&utm_source=chatgpt.com "noindex を使用してコンテンツをインデックスから除外する | Google ..."))
        
- **Integration details**
    
    - Content API: WordPress **REST API** (core + custom endpoints) makes your marketing site future‑proof even if you later replace the WP front end. ([WordPress Developer Resources](https://developer.wordpress.org/rest-api/?utm_source=chatgpt.com "REST API Handbook | Developer.WordPress.org"))
        
    - Navigation/branding: share a design system; keep consistent header/footer, and cross‑link from WP to key app entry points (login/trial).
        
    - Indexing control: **index** root marketing pages; **noindex** app pages (login, dashboards). Use WP plugins for SEO on the root and HTTP headers for the app. ([Google for Developers](https://developers.google.com/search/docs/crawling-indexing/block-indexing?hl=ja&utm_source=chatgpt.com "noindex を使用してコンテンツをインデックスから除外する | Google ..."))
        
    - Analytics: use the root domain as your primary property; create a view/roll‑up for cross‑subdomain sessions.
        

**When to choose this:** you have strong editorial needs, want plugin‑powered SEO, and the app truly is private/auth‑gated.

**Trade‑offs:** split repo/tech stacks; you’ll manage two deploy pipelines. If you later want `/app` (subdirectory) instead of `app.` (subdomain), you can proxy (see next option).

---

### B) **Same domain, different origins: reverse proxy `/app` to your React app**

- **What:** Keep marketing at `example.com/…` (WP or any CMS) and mount the React app at `example.com/app` via **reverse proxy/rewrites** (CDN, Nginx, or platform features).
    
- **Why:** preserves one hostname and **consolidates signals** under the root; avoids cross‑origin login cookie issues.
    
- **How:** if you’re on Vercel, use **external rewrites** to proxy paths to another origin; if you’re using Next.js, **Multi‑Zones** lets you mount separate apps under one domain. ([Vercel](https://vercel.com/docs/rewrites?utm_source=chatgpt.com "Rewrites on Vercel"))
    

**Gotchas:** ensure correct status codes for 404/410 under `/app`, keep canonical links precise, and still **noindex** authenticated routes. ([Google for Developers](https://developers.google.com/search/docs/crawling-indexing/block-indexing?hl=ja&utm_source=chatgpt.com "noindex を使用してコンテンツをインデックスから除外する | Google ..."))

---

### C) **Headless WordPress + React front end (Next.js/Gatsby)**

- **What:** WordPress only for content editing/API; public site is built with a React framework that **pre‑renders** pages.
    
- **Benefits:** HTML is ready on first byte (great for crawl & Core Web Vitals) + you retain WordPress editorial workflow. **Next.js** offers **ISR** (incremental static regeneration) to keep pages fresh without slow builds; **Gatsby** offers **DSG** (defer building lower‑value pages until first request). ([Next.js](https://nextjs.org/docs/pages/guides/incremental-static-regeneration?utm_source=chatgpt.com "Guides: ISR | Next.js"))
    
- **How:** Pull data via WP REST API at build or revalidation time; generate sitemap/robots in the React app (see Next file conventions below). ([WordPress Developer Resources](https://developer.wordpress.org/rest-api/?utm_source=chatgpt.com "REST API Handbook | Developer.WordPress.org"))
    

---

## 2) React‑friendly frameworks that are **SEO‑forward**

|Framework|Why it’s strong for SEO|Signature features you’ll use|
|---|---|---|
|**Next.js (App Router)**|SSR/SSG by default, **Server Components + streaming** ship HTML early; first‑class **metadata**, file‑based **robots/sitemap**; **ISR** updates statics in place.|`generateMetadata`, `app/robots.(ts|
|**React Router v7 (Remix lineage)**|SSR + **pre‑rendering** for static URLs; **progressive enhancement** keeps pages functional before JS; deployable to edge (Cloudflare).|Route‑level `<Meta/>` + **prerender** config; Workers deployment guides. ([React Router](https://reactrouter.com/start/framework/rendering?utm_source=chatgpt.com "Rendering Strategies \| React Router"))|
|**TanStack Start**|Modern full‑stack React with **full‑document SSR**, **streaming**, and **Selective SSR per route** (turn SSR off where you don’t need it).|Per‑route `ssr` controls, streaming SSR examples. ([TanStack](https://tanstack.com/start/community/docs?utm_source=chatgpt.com "TanStack Start Overview \| TanStack Start React Docs"))|
|**Astro (with React islands)**|“**Islands architecture**” → static HTML by default, hydrate only interactive components; add **@astrojs/react** to use React on an ultra‑fast static shell. Great for content/marketing.|React integration, islands (`client:*` directives), built‑in sitemap/adapter ecosystem. ([Astro Docs](https://docs.astro.build/en/concepts/islands/?utm_source=chatgpt.com "Islands architecture - Docs"))|
|**Gatsby**|Mature SSG with **DSG** + optional SSR; huge source plugin ecosystem for CMSs (WP, Contentful, etc.).|DSG/SSR rendering options; WP integration guides. ([Gatsby](https://www.gatsbyjs.com/docs/conceptual/rendering-options/?utm_source=chatgpt.com "Rendering Options - Gatsby"))|
|**Vike (ex vite‑plugin‑ssr)**|Lightweight, Vite‑based SSR with **streaming** and **file‑based routing**—more control, fewer opinions.|SSR/streaming docs and routing primitives. ([vite-plugin-ssr.com](https://vite-plugin-ssr.com/?utm_source=chatgpt.com "vite-plugin-ssr"))|
|**Shopify Hydrogen** _(e‑commerce specific)_|React Router + SSR for storefronts with ready‑made SEO hooks (meta/sitemap/robots).|SEO guide for Hydrogen projects. ([Shopify](https://shopify.dev/docs/api/hydrogen/latest?utm_source=chatgpt.com "Hydrogen - Shopify Developers Platform"))|

> **Why these help SEO:** Google explicitly recommends solving JS‑generated content with **SSR or static rendering**, not “dynamic rendering” as a long‑term solution. The frameworks above make that straightforward. ([Google for Developers](https://developers.google.com/search/docs/crawling-indexing/javascript/dynamic-rendering?utm_source=chatgpt.com "Dynamic rendering as a workaround - Google Developers"))

---

## 3) Three **outside‑the‑box** patterns that work shockingly well

1. **Astro for the entire marketing surface, React where it counts**  
    Build the whole public site in Astro (ship almost zero JS), sprinkle React components for rich widgets (pricing calculator, carousels). Result: tiny payloads, great LCP/INP, and SEO‑ready HTML. ([Astro Docs](https://docs.astro.build/en/concepts/islands/?utm_source=chatgpt.com "Islands architecture - Docs"))
    
2. **Next.js Multi‑Zones to stitch multiple apps into one domain**  
    Put docs, marketing, and app in separate codebases, yet serve them seamlessly on one host (`/docs`, `/blog`, `/app`) with independent deployments. ([Next.js](https://nextjs.org/docs/pages/guides/multi-zones?utm_source=chatgpt.com "Guides: Multi-Zones | Next.js"))
    
3. **Edge‑first SSR**  
    Deploy Next/React Router to edge runtimes (Vercel Edge Functions, Cloudflare Workers) so time‑to‑HTML is minimal worldwide—even for SSR pages. Faster first HTML helps crawling and UX, and frameworks document streaming/edge patterns. ([Vercel](https://vercel.com/docs/functions/runtimes/edge?utm_source=chatgpt.com "Edge Runtime - Vercel"))
    

---

## 4) Implementation playbooks

### A) If you pick **WordPress (root) + React app (subdomain)**

**SEO & crawling**

- Public WP pages: indexable; generate sitemap and manage titles/meta in WP.
    
- App (auth): add `meta name="robots" content="noindex"` or an **X‑Robots‑Tag** header. Remember: robots.txt **cannot** noindex. ([Google for Developers](https://developers.google.com/search/docs/crawling-indexing/block-indexing?hl=ja&utm_source=chatgpt.com "noindex を使用してコンテンツをインデックスから除外する | Google ..."))
    
- Search Console: create a **Domain Property** to see aggregate data across `www`, `app.`, and http/https. ([Google Help](https://support.google.com/webmasters/answer/34592?hl=en&utm_source=chatgpt.com "Add a website property to Search Console - Search Console Help"))
    

**Links & structure**

- Keep canonical **marketing URLs** at the root. Deep links into the app are fine, but don’t try to get private URLs indexed.
    
- If you later go multilingual, subdomains (`fr.example.com`) or subdirectories (`/fr/`) both work—just implement **`hreflang`** correctly. ([Google for Developers](https://developers.google.com/search/docs/specialty/international/managing-multi-regional-sites?utm_source=chatgpt.com "Managing Multi-Regional and Multilingual Sites | Google ..."))
    

**Hardening the app surface**

- Make `/login`, `/account`, `/dashboard` **noindex** by default (header or meta) and return **401/403** where appropriate to avoid “soft‑404” patterns in the index. ([Google for Developers](https://developers.google.com/search/docs/crawling-indexing/block-indexing?hl=ja&utm_source=chatgpt.com "noindex を使用してコンテンツをインデックスから除外する | Google ..."))
    

---

### B) If you pick **Headless WordPress + React front end**

- In **Next.js**, fetch WP content in `generateStaticParams/getStaticProps` and set **ISR** (`revalidate`) for near‑real‑time freshness without full rebuilds. Use the **Metadata API** for titles/OG; ship **`app/robots.ts`** and **`app/sitemap.ts`**. ([Next.js](https://nextjs.org/docs/pages/guides/incremental-static-regeneration?utm_source=chatgpt.com "Guides: ISR | Next.js"))
    
- In **Gatsby**, pick **DSG** to defer low‑value pages and keep builds fast; SSR pages where needed. ([Gatsby](https://www.gatsbyjs.com/docs/how-to/rendering-options/using-deferred-static-generation/?utm_source=chatgpt.com "Using Deferred Static Generation (DSG) - Gatsby"))
    

---

### C) If you pick a **React‑first meta‑framework**

- **Next.js (App Router)**
    
    - Use **Server Components + streaming** to deliver HTML early; add `loading.tsx` for route‑level streams. ([Next.js](https://nextjs.org/docs/14/app/building-your-application/rendering/server-components?utm_source=chatgpt.com "Rendering: Server Components | Next.js"))
        
    - Centralize SEO in `generateMetadata` (per page/layout) + `app/robots.*` + `app/sitemap.*`. ([Next.js](https://nextjs.org/docs/app/getting-started/metadata-and-og-images?utm_source=chatgpt.com "Getting Started: Metadata and OG images | Next.js"))
        
    - For hybrid content sites, mix SSG, **ISR**, and SSR per route. ([Next.js](https://nextjs.org/learn/seo/rendering-strategies?utm_source=chatgpt.com "SEO: Rendering Strategies | Next.js"))
        
- **React Router v7 / Remix‑style**
    
    - SSR + **static pre‑rendering** of known routes gives SEO‑ready HTML even on static hosting. Use route **`meta`** APIs for titles/descriptions. ([React Router](https://reactrouter.com/start/framework/rendering?utm_source=chatgpt.com "Rendering Strategies | React Router"))
        
    - Cloudflare Workers/Pages have official guides for React Router deployments (edge SSR). ([Cloudflare Docs](https://developers.cloudflare.com/workers/framework-guides/web-apps/react-router/?utm_source=chatgpt.com "React Router (formerly Remix) · Cloudflare Workers docs"))
        
- **TanStack Start**
    
    - Turn **SSR on/off per route** (e.g., marketing = SSR, app settings = CSR) to balance cost and SEO. Streaming SSR is built‑in. ([TanStack](https://tanstack.com/start/latest/docs/framework/react/guide/selective-ssr?utm_source=chatgpt.com "Selective Server-Side Rendering (SSR) | TanStack Start ..."))
        
- **Astro (with React islands)**
    
    - Build the marketing site as static HTML; hydrate only the React components that truly need interactivity (e.g., comparison widgets). ([Astro Docs](https://docs.astro.build/en/concepts/islands/?utm_source=chatgpt.com "Islands architecture - Docs"))
        
- **Vike (formerly vite‑plugin‑ssr)**
    
    - If you want maximal control on Vite, Vike provides **SSR + streaming** and file‑based routing without locking you into a big framework. ([vite-plugin-ssr.com](https://vite-plugin-ssr.com/?utm_source=chatgpt.com "vite-plugin-ssr"))
        

---

## 5) Choosing the right path (simple decision rules)

- **Strong editorial + blogs/docs**?  
    **Astro** or **Headless WP + Next/Gatsby** for marketing, **React app** separate. (Astro if you want the smallest JS; Next if you want one React‑only stack.) ([Astro Docs](https://docs.astro.build/en/concepts/islands/?utm_source=chatgpt.com "Islands architecture - Docs"))
    
- **One React codebase for everything (docs + marketing + app)**?  
    **Next.js App Router** with SSR/SSG/ISR + metadata/robots/sitemap conventions. ([Next.js](https://nextjs.org/docs/14/app/building-your-application/rendering/server-components?utm_source=chatgpt.com "Rendering: Server Components | Next.js"))
    
- **Prefer edge runtime + progressive enhancement**?  
    **React Router (v7)** or **Next on Edge**. ([Cloudflare Docs](https://developers.cloudflare.com/workers/framework-guides/web-apps/react-router/?utm_source=chatgpt.com "React Router (formerly Remix) · Cloudflare Workers docs"))
    
- **You want to dial SSR per route**?  
    **TanStack Start** (Selective SSR). ([TanStack](https://tanstack.com/start/latest/docs/framework/react/guide/selective-ssr?utm_source=chatgpt.com "Selective Server-Side Rendering (SSR) | TanStack Start ..."))
    
- **E‑commerce on Shopify**?  
    **Hydrogen** (React + SSR with dedicated SEO guidance). ([Shopify](https://shopify.dev/docs/api/hydrogen/latest?utm_source=chatgpt.com "Hydrogen - Shopify Developers Platform"))
    

---

## 6) Common pitfalls (and how to avoid them)

- **Relying on dynamic rendering** long‑term. It’s a workaround; prefer SSR/static rendering. ([Google for Developers](https://developers.google.com/search/docs/crawling-indexing/javascript/dynamic-rendering?utm_source=chatgpt.com "Dynamic rendering as a workaround - Google Developers"))
    
- **Trying to “noindex” in robots.txt.** Not supported; use meta or HTTP headers. ([Google for Developers](https://developers.google.com/search/docs/crawling-indexing/block-indexing?hl=ja&utm_source=chatgpt.com "noindex を使用してコンテンツをインデックスから除外する | Google ..."))
    
- **Separating marketing/app without Search Console coverage.** Add a **Domain Property** to aggregate signals across subdomains. ([Google Help](https://support.google.com/webmasters/answer/34592?hl=en&utm_source=chatgpt.com "Add a website property to Search Console - Search Console Help"))
    
- **Orphaned paginated pages / infinite scroll.** Ensure crawlable links (real `<a>`) to deeper content in whatever framework you choose. (All frameworks above support generating real anchor tags.)
    

---

## 7) Quick start templates you can adopt this week

- **WordPress + Next.js ISR front end**
    
    - WP as headless CMS (REST API). Next’s **ISR** keeps pages fresh; use **Metadata & file conventions** for OG/robots/sitemap. ([WordPress Developer Resources](https://developer.wordpress.org/rest-api/?utm_source=chatgpt.com "REST API Handbook | Developer.WordPress.org"))
        
- **Astro marketing site + React islands, app on `app.`**
    
    - Astro for public pages (fast static HTML), integrate React components via `@astrojs/react`; point CTAs to `app.` with noindex on auth pages. ([Astro Docs](https://docs.astro.build/en/guides/integrations-guide/react/?utm_source=chatgpt.com "@astrojs/react | Docs"))
        
- **Next.js Multi‑Zones or Vercel rewrites**
    
    - Serve separate Next apps (or an external app) under one domain with rewrites; deploy independently. ([Next.js](https://nextjs.org/docs/pages/guides/multi-zones?utm_source=chatgpt.com "Guides: Multi-Zones | Next.js"))
        

---

### Final recommendation (based on your example)

If your priority is **best‑in‑class SEO on public pages** and a **clean, private React app**:

- **Do:** WordPress (root) for all indexable marketing/docs + **noindex** the React app on `app.` (or proxy it to `/app` if you want one hostname). Manage indexing via meta/header, not robots.txt. Use a **Domain Property** in Search Console to track everything together. ([Google for Developers](https://developers.google.com/search/docs/crawling-indexing/block-indexing?hl=ja&utm_source=chatgpt.com "noindex を使用してコンテンツをインデックスから除外する | Google ..."))
    
- **Or (one‑stack React alternative):** Next.js (App Router) for marketing + app, using **SSR/SSG/ISR**, **generateMetadata**, and file‑based **robots/sitemap**. If your content team still wants WordPress, run it headless behind Next’s ISR. ([Next.js](https://nextjs.org/docs/14/app/building-your-application/rendering/server-components?utm_source=chatgpt.com "Rendering: Server Components | Next.js"))
    

If you share your hosting constraints (Vercel/Cloudflare/self‑hosted) and whether content will be edited in WordPress or a headless CMS, I can map this to a concrete repo structure and the exact files (`app/robots.ts`, `app/sitemap.ts`, prerender lists, and `meta`/`generateMetadata` examples) that fit your stack.
