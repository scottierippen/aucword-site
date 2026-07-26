# Aucword Site — Session Log

**Purpose:** Single source of truth for what's been done to aucword.com. Read this FIRST when starting a new session. Update at the END of every session.

- **Repo:** `github.com/scottierippen/aucword-site`
- **Domain:** `aucword.com`
- **Hosting:** GitHub Pages (migrating to Vercel — see below)
- **Analytics:** PostHog JS (project `phc_zjuDw38ZHVd2KKNT6b6Ln3Rvr8u3YzHUetTBiZ3AMgBP`)
- **Related repos:** `~/Desktop/aucword` (iOS), `~/Desktop/aucword-android` (Android)

---

## Current State

### (2026-07-26) SEO + compliance pass, branch `feat/seo-and-compliance` — committed, NOT pushed, NOT merged

Implemented from `~/Downloads/aucword-seo-plan.md` (§2 items, email-capture form deliberately skipped) plus compliance work the plan did not cover.

**⚠️ NOT DONE — needs a human (see "Manual steps owed" below):** Vercel import, DNS cutover, disabling GitHub Pages, Search Console, Bing, link-preview validation, PageSpeed check, collecting real testimonials.

**⚠️ VISUAL RENDERING NEVER CONFIRMED.** Structure, links, JSON-LD, and tag balance are all validated programmatically, but the browser preview tool timed out three times so nobody has *looked* at the rendered pages. Do this on the first Vercel preview deploy before promoting to production.

---

## What changed

### Pages
| File | Status | Notes |
|---|---|---|
| `index.html` | rewritten | head/meta/schema, copy, new sections |
| `privacy.html` | corrected | Pendo removed, factual fixes — see below |
| `support.html` | **new** | `/support` — App Store + Play support URL target |
| `delete-account.html` | **new** | `/delete-account` — **Play launch blocker** |
| `404.html` | **new** | on-brand |
| `robots.txt`, `sitemap.xml`, `llms.txt` | **new** | sitemap has 4 URLs |
| `vercel.json` | **new** | cleanUrls, cache + security headers |
| `fonts/` | **new** | self-hosted DM Sans + OFL license |

### The privacy policy was factually wrong — this was the most consequential find
The published policy named **Pendo** as the analytics vendor (removed from both apps months ago in the PostHog migration), never mentioned **PostHog** (live, including session replay on this site), and had **zero mention of Android or FCM** while the Android app shipped push via FCM. It also claimed analytics data was "anonymized and aggregated" and that no PII goes to analytics providers — both untrue, since `identify` sends the account email to PostHog.

Fixed: dead Pendo agent script deleted (it was still loading), §7 vendor list corrected to Supabase / APNs / FCM / eBay API / PostHog, §2 rewritten to disclose session replay and post-sign-in identification honestly, §5 covers both push platforms, §1 covers both platforms, date March → July 2026, deletion page linked from §6 and §9.

**These are legal-text edits and have not been reviewed by a human. Read them.**

### SEO / meta
- **The `og:` tags had been silently wiped.** Commit `c668508` added 10 of them; a later "Add files via upload" through the GitHub web UI clobbered the whole file. Restored, and the root cause is the web-upload workflow — **edit in the repo, not through github.com.** Vercel preview deploys are the real fix.
- JSON-LD `@graph`: `MobileApplication` + `Organization` (with real `sameAs` for @aucword on X/Instagram/TikTok) + `WebSite` + `FAQPage`.
- **`aggregateRating` is hand-maintained** — 5.0 from 4 ratings, checked 2026-07-26. Refresh it when the App Store rating moves or delete the block. Stale values here are false structured data.
- `twitter:site` = `@aucword` — safe to ship, handles confirmed held on X, Instagram, TikTok.

**Second commit (`112131c`) came from actually auditing the pass rather than trusting the plan's copy table.** It found: (a) the home page's three feature `h3`s had **no parent `h2`**, nesting them under the problem statement; (b) "countdown", "widget", and "watchlist" appeared in **no `h2` at all**, and "eBay" in none either; (c) the support page's title was **17 characters** — the single biggest wasted SEO element, on the one page that can plausibly rank for troubleshooting long-tail; (d) "auction" appeared **once** on that page; (e) the deletion page title read as a UI label, not the query people type. All fixed. Subpages also gained `twitter:card` and `WebPage` + `BreadcrumbList` JSON-LD.

Audited state: 5/5 pages well-formed, all JSON-LD parses, exactly 1 `h1` per page, 0 skipped heading levels, titles 43–61ch and descriptions 155–167ch on the three indexable content pages, home page 629 words (was 274).

**Limitation worth knowing:** keyword targeting here is reasoning-based, not data-based — there was no search-volume tool in play. Validate the actual terms in Search Console once it has a few weeks of impression data, and adjust titles then.

### Copy
- H1: "Stop losing **eBay** auctions" — gains the keyword without narrowing to "card" (heaviest users are a collectibles business and a 555-item generalist).
- Hero sub + feature cards now mention the **Lock Screen countdown** and **price targets** — the actual wedge vs eBay's own app, previously absent from the site entirely.
- **Fabricated testimonials removed.** "Alex M." and "Ryan S." were invented; presenting them as endorsements is an FTC problem, not just weak copy. Replaced with the real 5.0/4-rating block and a "How it works" 3-step.
- New 9-question FAQ. Note Google restricted FAQ rich results to gov/health sites in Aug 2023 — this is for content depth and AI answer engines, not a SERP feature.
- `hero-note`: "Free · iOS 17+ · Android coming soon · eBay account required".

### Performance
- `screenshot.png` **2.7 MB → 69 KB** as `screenshot.webp` (600×1303, q80), PNG fallback also downsized to 684 KB. `<picture>` + `fetchpriority="high"` + `decoding="async"` + preload. Real dimensions set (the plan's 600×1300 was the wrong aspect; source is 1320×2868).
- `og-image.jpg` at 156 KB replaces the 529 KB PNG (the PNG was never referenced, so it was never actually costing page weight — the plan's "3.3 MB" figure was wrong; real weight was ~2.8 MB).
- **DM Sans self-hosted, one 36 KB file.** It's a variable font, so all weights come from a single file — the site had been loading 7 remote weights (DM Sans 300/400/500/600/700 + DM Mono 400/500) while using only 600, 700, and default 400. **DM Mono was never used at all.** Both Google preconnects and the render-blocking stylesheet are gone from every page.
- `aria-hidden` on decorative emoji; `prefers-reduced-motion` block added.

### Analytics
- **`?ref=win` now attributed** (was an open item in the iOS SESSION_LOG). Registered as `landing_ref` super property + `landing_viewed` event. It was already technically visible inside `$current_url`; this makes it usable in funnels.
- New events: `faq_opened{question,page}`, `support_link_clicked{location}`, `social_clicked{network,location}`, `policy_link_clicked{policy:'delete_account'}`. Existing `app_store_clicked{location}` preserved on every CTA, plus new `rating` and `support` locations.

---

## Manual steps owed (human required)

1. **Vercel:** import the repo → add `aucword.com` + `www` → update registrar DNS → verify.
2. **After DNS confirms:** disable GitHub Pages in repo settings, then delete `CNAME` (deleting it early breaks the Pages custom domain mid-propagation).
3. **Look at the rendered pages.** Nobody has yet.
4. **Search Console:** add domain property, verify by TXT, submit `https://aucword.com/sitemap.xml`, request indexing for `/`.
5. **Bing Webmaster Tools:** import from Search Console.
6. **Validate link previews:** Facebook debugger + iMessage self-test (`og-image.jpg` is new).
7. **PageSpeed:** re-run mobile after cutover.
8. **App Store Connect + Play Console:** set support URL to `https://aucword.com/support` and the Play account-deletion URL to `https://aucword.com/delete-account`.
9. **Read the privacy policy edits.** Legal text, unreviewed.
10. **Collect real testimonials** (plan §1.5: jeremy@sanfordcollectibles.com 124 items, jeff.vargas87@gmail.com 555 items, oezkantoprak@hotmail.com price-target user).

## Deliberately not done
- Email/waitlist capture form (skipped at the user's request — no Formspree dependency, no form on the site).
- Google Play badge + Android CTAs — a marked-off block in `index.html` nav is ready to uncomment when the Play listing goes live.
- Long-tail content pages (comparisons, sport-specific landers) — the plan's own phase 2.
- ASO pass on the App Store listing. Known values: title "Aucword: Auction Notifications", subtitle "Watchlist Tracker & Alerts", 5.0★/4, Free, iOS 17.0+, Shopping.
- CSP header — the inline PostHog snippet and inline `<style>` would force `unsafe-inline`, which buys nothing. Revisit if both get externalized.

## Gotchas for next session
- **Never edit files through the GitHub web UI.** That is what destroyed the original SEO tags.
- Vercel Hobby prohibits commercial use; with eBay Partner Network revenue this arguably needs Pro ($20/mo). Cloudflare Pages is free and permits commercial use — flagged, user chose Vercel.
- Local `python3 -m http.server` can't reproduce `cleanUrls`, so `/privacy` 404s locally while working fine on Vercel. Test extensionless routes on a preview deploy.
- The Claude browser tooling timed out repeatedly this session (3 × 300s). Don't burn time retrying it.
