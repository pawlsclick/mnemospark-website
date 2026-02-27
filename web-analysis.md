# Website Technology Stack Analysis: keepmyclaw.com

**URL:** https://keepmyclaw.com/  
**Date of Analysis:** February 27, 2026  
**Site Type:** One-page marketing/landing page for an encrypted AI agent backup service

---

## Summary

keepmyclaw.com is a lightweight, hand-crafted static website with **zero frontend frameworks, no CSS libraries, and no JavaScript dependencies**. The entire page is a single HTML file with inline CSS and vanilla JavaScript. The infrastructure runs entirely on the Cloudflare platform (Pages, Workers, R2, DNS, CDN). Stripe handles payments.

---

## Frontend

### HTML & Templating

| Aspect | Detail |
|---|---|
| **Document type** | HTML5 (`<!DOCTYPE html>`) |
| **Framework** | None — raw, hand-written HTML |
| **Build tool** | None detected (no Webpack, Vite, Parcel, or bundler fingerprints) |
| **Templating engine** | None — static HTML files served as-is |

The page is a single `.html` file containing all markup, styles, and scripts inline. There is no evidence of any static site generator (Hugo, Jekyll, Astro, 11ty, etc.), nor any SPA framework (React, Vue, Angular, Svelte, Next.js, Nuxt, etc.).

### CSS

| Aspect | Detail |
|---|---|
| **CSS framework** | None (no Tailwind, Bootstrap, Bulma, etc.) |
| **Delivery method** | Inline `<style>` block in `<head>` (~230 lines) |
| **Layout system** | CSS Grid + Flexbox |
| **Theming** | CSS Custom Properties (variables) on `:root` |
| **Responsive design** | `@media` queries (breakpoint at 768px) + `clamp()` for fluid typography |
| **Font stack** | System fonts: `-apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif` |
| **Monospace font** | `'SF Mono', Monaco, Consolas, monospace` (for code blocks) |

Notable CSS techniques used:

- **CSS Custom Properties** for a dark theme color palette (`--bg`, `--surface`, `--accent`, etc.)
- **`backdrop-filter: blur(12px)`** for a frosted-glass fixed navbar
- **CSS Grid** for the pricing cards, feature grid, and 3-step "How It Works" section
- **`clamp()`** for responsive heading sizes (e.g., `clamp(2.8rem, 6vw, 4.5rem)`)
- **CSS `@keyframes` animations** for the chat typing indicator and message fade-in effects
- **Gradient text** via `background-clip: text` and `-webkit-text-fill-color: transparent`
- **Radial gradient** hero background glow effect

### JavaScript

| Aspect | Detail |
|---|---|
| **JS framework/library** | None — vanilla JavaScript only |
| **Delivery method** | Two inline `<script>` blocks (~100 lines total) |
| **External scripts** | None loaded on the landing page (Stripe JS loaded on-demand via CSP allowlist) |

Interactive features implemented in vanilla JS:

1. **Rotating word animation** — cycles through keywords ("Memory", "Skills", "Cron Jobs", etc.) in the hero heading with CSS fade transitions
2. **Chat simulation** — a timed sequence of chat messages that mimics a conversation between a user and their AI agent, with typing indicators
3. **Demo tab switching** — toggles between a CLI demo view and the chat simulation view

---

## Backend & API

| Aspect | Detail |
|---|---|
| **API endpoint (proxied)** | `https://api.keepmyclaw.com` |
| **API endpoint (direct)** | `https://keepmyclaw-api.rzy.workers.dev` |
| **Runtime** | **Cloudflare Workers** (serverless edge functions) |
| **Response format** | JSON (`application/json`) |
| **Authentication** | Bearer token (Authorization header in CORS config) |
| **CORS** | Configured with `access-control-allow-origin: *`, specific allowed headers (`Authorization`, `Content-Type`, `X-Snapshot-Size`) and methods (`GET`, `POST`, `DELETE`, `OPTIONS`) |

---

## Hosting & Infrastructure

| Component | Technology |
|---|---|
| **Static hosting** | **Cloudflare Pages** |
| **CDN** | **Cloudflare CDN** (HTTP/2, cf-ray headers, cf-cache-status) |
| **DNS provider** | **Cloudflare DNS** (nameservers: `lisa.ns.cloudflare.com`, `mack.ns.cloudflare.com`) |
| **DNS records** | A records pointing to Cloudflare proxy IPs: `172.67.175.108`, `104.21.17.94` |
| **API compute** | **Cloudflare Workers** |
| **Object storage** | **Cloudflare R2** (used for storing encrypted backup blobs) |
| **SSL/TLS** | Cloudflare-managed with HSTS preload |
| **Protocol** | HTTP/2 with HTTP/3 (`alt-svc: h3`) advertised |

The entire infrastructure stack is **100% Cloudflare**: DNS, CDN, static hosting (Pages), serverless compute (Workers), and object storage (R2).

---

## Payments

| Aspect | Detail |
|---|---|
| **Payment processor** | **Stripe** |
| **Integration method** | Stripe.js (`https://js.stripe.com`) loaded via iframe (CSP `frame-src` and `script-src` allowlist) |
| **Billing portal** | Stripe-hosted (mentioned in FAQ for subscription cancellation) |

---

## Security Headers

The site has a thorough set of security headers:

| Header | Value |
|---|---|
| **Content-Security-Policy** | Restrictive policy: `default-src 'self'`; scripts limited to self, inline, and Stripe; styles limited to self, inline, and Google Fonts; connections limited to self and the two API endpoints |
| **Strict-Transport-Security** | `max-age=31536000; includeSubDomains; preload` |
| **X-Content-Type-Options** | `nosniff` |
| **X-Frame-Options** | `ALLOW-FROM https://rankinpublic.xyz` |
| **Referrer-Policy** | `strict-origin-when-cross-origin` |
| **Permissions-Policy** | `camera=(), microphone=(), geolocation=()` (all disabled) |

---

## SEO & Structured Data

| Aspect | Detail |
|---|---|
| **Meta tags** | Title, description, canonical URL, robots directive |
| **Open Graph** | Full OG tags (title, description, URL, type, site_name, locale, image with dimensions) |
| **Twitter Cards** | `summary_large_image` card with title, description, image, and site handle (`@benrzy`) |
| **JSON-LD** | Four schema types: `Organization`, `Product` (with pricing offers), `FAQPage` (8 Q&A pairs), `SoftwareApplication` |
| **Sitemap** | XML sitemap at `/sitemap.xml` (15 URLs including blog posts, docs, terms, privacy) |
| **robots.txt** | Cloudflare Managed Content Signals; blocks AI training crawlers (GPTBot, ClaudeBot, Google-Extended, CCBot, Bytespider, etc.); allows search indexing |
| **Favicon** | SVG favicon (`/favicon.svg`) with an emoji fallback via data URI |

---

## Site Structure (from Sitemap)

| Page | Purpose |
|---|---|
| `/` | Main landing page (this analysis) |
| `/docs` | Product documentation |
| `/blog/` | Blog index |
| `/blog/*` | 10 blog posts on AI agent topics |
| `/terms` | Terms of service |
| `/privacy` | Privacy policy |
| `/success.html` | Post-payment success page (blocked in robots.txt) |

---

## What is NOT Used

For clarity, these common web technologies are **absent**:

- **No frontend framework** (React, Vue, Angular, Svelte, Solid, etc.)
- **No meta-framework** (Next.js, Nuxt, Astro, Remix, SvelteKit, etc.)
- **No CSS framework** (Tailwind, Bootstrap, Bulma, etc.)
- **No CSS preprocessor** evidence (Sass, Less, PostCSS)
- **No JavaScript libraries** (jQuery, Alpine.js, GSAP, etc.)
- **No build tooling** (Webpack, Vite, Rollup, esbuild, Parcel)
- **No analytics/tracking scripts** (Google Analytics, Plausible, Fathom, etc.)
- **No cookie consent banners**
- **No external fonts loaded** (Google Fonts is allowed in CSP but not loaded on the landing page; system fonts are used)
- **No images** (the page uses only emoji characters for icons)

---

## Architecture Diagram

```
┌─────────────────────────────────────────────────────┐
│                    Cloudflare Edge                   │
│                                                     │
│  ┌──────────────┐  ┌──────────────┐  ┌───────────┐ │
│  │  Cloudflare   │  │  Cloudflare   │  │ Cloudflare│ │
│  │    Pages      │  │   Workers     │  │    R2     │ │
│  │  (Static HTML)│  │  (API Logic)  │  │ (Storage) │ │
│  └──────┬───────┘  └──────┬───────┘  └─────┬─────┘ │
│         │                  │                │       │
│  ┌──────┴──────────────────┴────────────────┘       │
│  │         Cloudflare CDN + DNS                     │
│  └──────────────────────┬───────────────────────────┘
│                         │
└─────────────────────────┼───────────────────────────┘
                          │
              ┌───────────┴───────────┐
              │     Stripe (Payments) │
              └───────────────────────┘
```

---

## Key Takeaways

1. **Extreme minimalism** — The entire landing page is ~530 lines of self-contained HTML/CSS/JS with zero external dependencies loaded at page render time. This results in exceptional performance: no framework overhead, no render-blocking resources, no layout shifts.

2. **Cloudflare-native stack** — Every infrastructure component lives on Cloudflare's platform (Pages, Workers, R2, DNS, CDN), creating a unified, low-latency, globally-distributed architecture with minimal operational complexity.

3. **Security-conscious** — Comprehensive CSP, HSTS with preload, disabled browser APIs, restrictive CORS, and AI-crawler blocking via robots.txt content signals.

4. **No analytics** — Notably absent is any client-side tracking or analytics script, aligning with the product's privacy-focused branding.

5. **Pure CSS design system** — The dark theme, animations, responsive layout, and all visual effects are achieved through modern CSS features (custom properties, grid, clamp, backdrop-filter, gradient text) without any preprocessor or utility framework.
