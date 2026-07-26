# Aucword Site — Session Log

**Purpose:** Single source of truth for what's been done to aucword.com. Read this FIRST when starting a new session. Update at the END of every session.

- **Repo:** `github.com/scottierippen/aucword-site`
- **Domain:** `aucword.com`
- **Hosting:** Vercel (migrated off GitHub Pages 2026-07-26). DNS at Cloudflare, unproxied.
- **App Store attribution:** Provider ID `128248580`, campaign tokens on every store link
- **Analytics:** PostHog JS (project `phc_zjuDw38ZHVd2KKNT6b6Ln3Rvr8u3YzHUetTBiZ3AMgBP`)
- **Related repos:** `~/Desktop/aucword` (iOS), `~/Desktop/aucword-android` (Android)

---

## Current State

### (2026-07-26) SEO + compliance pass — **LIVE ON `aucword.com`, hosted on Vercel**

Implemented from `~/Downloads/aucword-seo-plan.md` (§2 items, email-capture form deliberately skipped) plus compliance work the plan did not cover. Branch `feat/seo-and-compliance` (3 commits) merged to `main` and pushed; Vercel redeployed in ~10s.

**Hosting: Vercel. DNS cut over 2026-07-26 and verified.** GitHub Pages disabled; `CNAME` deleted. Project is NOT in the `nuggget` Vercel team (that team still has only `nuggget`). DNS is managed at **Cloudflare** (`amanda`/`fattouche.ns.cloudflare.com`), records unproxied (grey cloud) — keep it that way, proxying Vercel breaks its cert provisioning.

**Verified live on `https://aucword.com`:** apex + `www` on Vercel with **zero GitHub Pages A or AAAA records left**; `http` → `https` 308; `cleanUrls` works (`/privacy`, `/support`, `/delete-account` 200; `/privacy.html` 308s to `/privacy`); custom 404 fires; `robots.txt`, `sitemap.xml`, `llms.txt`, `favicon.ico`, `site.webmanifest` all 200; all six assets 200 with correct content types; all five security headers present; font `immutable` 1yr, images 1d + `stale-while-revalidate`. Homepage 33,523 bytes (old site was 17,309); **cold load 185 KB vs ~2.8 MB before**. Fabricated testimonials, Google Fonts, and Pendo confirmed gone; `?ref=win` handler present.

**⚠️ `www.aucword.com` serves 200 rather than redirecting** — both hostnames serve the full site. The canonical tag on the www version correctly points at the apex so Google will consolidate, but a redirect is cleaner for crawl budget and analytics. Fix in Vercel → Domains → `www.aucword.com` → "Redirect to aucword.com".

**⚠️ A `cleanUrls` trap was caught live and is worth remembering.** Before the merge, the Vercel deploy served `main` (the old site) *without* `vercel.json`, and **`/privacy` returned 404 while `/privacy.html` returned 200**. GitHub Pages resolves extensionless URLs; Vercel does not unless `cleanUrls` is set. `https://aucword.com/privacy` is the URL both app stores and the in-app Settings row point at — cutting DNS in that state would have broken the privacy policy link in both apps. Merging fixed it because `vercel.json` came with the branch. **If `vercel.json` is ever removed or the project is re-imported, re-check `/privacy` before touching DNS.**

**✅ Visual rendering confirmed by the user** (full-page desktop screenshot, end of session). Renders correctly: self-hosted font, gradients, webp screenshot, card grids, FAQ accordions. The screenshot is what triggered the layout rework below.

**Note for future sessions: the assistant cannot see this site.** `aucword.com` and `aucword-site.vercel.app` are both blocked by browser policy in its environment, and the local preview tool timed out 3×300s. Every visual change needs a screenshot from the user to verify. Structure/headers/routes can be fully verified over HTTP.

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

### Desktop layout rework (from the user's screenshot)
The site read as a mobile layout stretched onto a desktop: card grids capped at 1000px and the hero at 800px left ~a third of a 1440px screen as empty margin, and the H1 broke into **four** lines because "Stop losing eBay auctions" doesn't fit 800px at 64px and the hard `<br/>` then split it again.

- **Hero is two columns above 900px** — copy left, phone right. Removes ~500px of vertical scroll and stops the phone sitting alone in a 300px column.
- Shell widened 1000 → **1200px** (card grids + footer).
- **Prose columns deliberately left narrow** (problem 660px, FAQ 800px). Line length past ~75 characters hurts readability, so the narrowness there was correct — widening it would make the page worse.
- H1: hard `<br/>` dropped, `text-wrap: balance` added, scale reduced to `clamp(34px, 4.6vw, 56px)`.
- The 5.0 rating moved out of its own near-empty band into the hero beside the CTA, where social proof affects the click.
- **New 701–899px breakpoint** drops feature cards to two-up; three in a ~720px window were cramped (true before this rework too). Phone frame steps 260 → 300 → 320px. Single column remains the mobile-first base state, so no override is needed for phones.

### Favicon (was broken)
The site declared `logo.png` — **400×211, a 1.9:1 wordmark squashed into a 1:1 slot** — and used the same file as `apple-touch-icon`, where non-square gets letterboxed. **`/favicon.ico` 404'd**, so any client probing the default path got nothing.

Rebuilt from the iOS `AppIcon` (1024×1024, square, brand navy, designed to survive small sizes — so the browser tab now matches the App Store listing). Set: `favicon.ico` (PNG-in-ICO at 16/32/48, avoids an ImageMagick dependency), `favicon-16/32.png`, `apple-touch-icon.png` 180, `icon-192/512.png`, `site.webmanifest`. **The 16/32 variants are cut from a ~1.3× centre crop** — Apple's safe-area padding leaves the mark unreadable at 16px.

### App Store install attribution
PostHog saw `app_store_clicked` but **nothing survived the hop through the App Store**, so there was no way to know whether any website click became an install. Apple reports installs per campaign token in App Analytics, which closes it in aggregate. User-level web→install is not achievable without a third-party attribution SDK and isn't worth it at this volume.

- Every store link now carries `pt=128248580` plus a per-placement token: `website_nav`, `website_hero`, `website_rating`, `website_final`, `website_support`, `website_404`.
- Smart App Banner taps were **entirely untracked**; now carry `affiliate-data` with `website_banner` / `support_banner` / `privacy_banner`. This is the least-certain piece — treat as a bonus, not a load-bearing number.
- **`?ref=win` rewrites `ct` to `win_share` on load.** Previously it was impossible to tell whether the win-share loop drove a single install — the entire point of a feature shipped on both platforms. Static placement tokens stay in the HTML, so JS-off users still get attribution; the ref value is validated against `[a-z0-9_-]` before touching a URL.
- Links switched from `/us/app/...` to Apple's locale-less `/app/apple-store/...`. The old path forced every visitor onto the **US storefront**; international visitors now reach their own.
- **Verified** the campaign URL survives Apple's redirect with both `pt` and `ct` intact.
- JSON-LD `installUrl`/`downloadUrl` left clean — tracking params don't belong in structured data.
- **Unverified:** whether arbitrary `ct` values report without being pre-registered in App Store Connect. Only `website` was created by the user. Check Campaigns in a few days; if only `website` appears, generate the rest with App Store Connect's Campaign Link tool and match its format.


---

## Manual steps owed (human required)

1. ~~**Vercel:** import the repo, add domains, cut DNS over~~ — **DONE 2026-07-26, verified.**
2. ~~**Disable GitHub Pages, delete `CNAME`**~~ — **DONE.**
3. ~~**Search Console**~~ — **DONE 2026-07-26.** Confirmed no indexing blockers: no `x-robots-tag` anywhere, all 4 sitemap URLs 200 + self-canonical, Googlebot gets 200. Still worth requesting indexing for `/` and `/delete-account`.
4. ~~**Look at the rendered pages**~~ — **DONE** (user screenshot).
5. **App Store Connect + Play Console:** set support URL to `https://aucword.com/support` and the Play account-deletion URL to `https://aucword.com/delete-account`. **This is the one that unblocks the Play submission** — the 14-day closed-test clock is the Android critical path.
6. **Set `www` to redirect to the apex** in Vercel → Domains. Currently serves 200; canonical covers it, so low stakes.
7. **Confirm the `ct` tokens appear** in App Analytics → Acquisition → Campaigns (see attribution note above).
8. **Bing Webmaster Tools:** import from Search Console.
9. **Validate link previews:** Facebook debugger + iMessage self-test (`og-image.jpg` is new).
10. **PageSpeed:** re-run mobile now that DNS has cut over.
11. **Read the privacy policy edits.** Legal text, unreviewed.
12. **Collect real testimonials** (plan §1.5: jeremy@sanfordcollectibles.com 124 items, jeff.vargas87@gmail.com 555 items, oezkantoprak@hotmail.com price-target user).

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
- **The assistant cannot see this site** (browser policy blocks both hostnames; local preview tool hangs). Visual changes need a user screenshot to verify — plan for that round trip.
- Keep Cloudflare records **unproxied (grey cloud)**. Proxying Vercel breaks its cert provisioning.
- `aggregateRating` in `index.html` and the App Analytics `ct` tokens are both hand-maintained. Neither self-updates.
