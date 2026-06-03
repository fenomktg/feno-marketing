# Feno Offer + PDP — Build & Asset Checklist

Two audiences, one doc:

- **Part A** is for the developer building the real thing: what each page is supposed to *do*.
- **Part B** is for Bloom Design: every image and video the prototype needs, with exact sizes.

This prototype is an intentional **wireframe**. Every dashed grey box labeled "Main product view", "App dashboard", "Product demo video", etc. is a placeholder marking where a real asset goes. Nothing in those boxes is final art.

For architecture, state model, and the `data-target` sync system, see `HANDOFF.md` in this folder. This checklist sits on top of that.

Pages: `/` (offer page, `index.html`) and `/pdp` (product detail, `pdp.html`).

---

## Part A — Developer: what is supposed to happen

### Plans and pricing
- [ ] Three ways to buy the same Founders Edition bundle:
  - **One Time** — $299, pay once.
  - **Subscribe & Save, Monthly** — $35/mo.
  - **Subscribe & Save, Annual** — $360/year (saves $60 vs paying monthly for a year).
- [ ] Annual is positioned as best long-term value. Monthly is positioned as lowest cost to start.
- [ ] **Fulfillment differs by plan and must be reflected in copy:**
  - Annual ships the full year of supplies up front: 12 Feno Foam + 4 TrueFit Mouthpieces.
  - Monthly ships supplies quarterly: 3 Feno Foam + 1 TrueFit Mouthpiece every 90 days.
  - One Time includes the bundle's starting supplies only; refills are a cart upsell.

### Purchase flow
- [ ] **Subscribe & Save (monthly or annual) goes straight to checkout.** No cart step.
- [ ] **One Time goes to the cart first**, because the cart carries the refill upsell. Only One Time has a cart/upsell step.
- [ ] CTA copy must read as "go buy now," not "learn more." Offer-page sticky CTA switches between **Checkout** (subscribe) and **Add to cart** (one time). PDP primary CTA is **Buy Now** with the live price appended.
- [ ] After a user reaches checkout and presses browser Back, **do not** pop a pull-up sheet on the monthly plan. The pull-up cart/sheet belongs to the One Time path only.

### Plan carryover (offer page → PDP)
- [ ] On the offer page, "Learn more" links to the PDP carrying the selected plan: `/pdp?plan=<plan>`.
- [ ] The PDP reads `?plan=` on load and defaults its buy box to match:
  - `?plan=annual` → Subscribe & Save, Annual.
  - `?plan=upfront` (or `onetime`) → One Time.
  - `?plan=monthly` or no param → Subscribe & Save, Monthly (default).
- [ ] Verified working in the prototype for all three values.

### PDP buy box
- [ ] Top tabs select **One Time** vs **Subscribe & Save**. Inside Subscribe & Save, a sub-toggle selects **Monthly** vs **Annual** (Annual shows a "Save $60" chip).
- [ ] Price, plan name, included-items list, and the terms line all update live with the selection.
- [ ] A sticky Buy Now bar appears on scroll (mobile) with the current price.

### Reviews
- [ ] The 3 review cards on the page are sample/social-proof preview only.
- [ ] "See all 1,247 reviews" reveals the **full Okendo reviews widget** (review list, rating breakdown, photo reviews, filters). In the prototype this is a placeholder box. In production it is the embedded Okendo widget.
- [ ] Rating summary (4.8 / 5, review count) should be driven by Okendo data once wired.

### Cancellation terms (Subscribe & Save)
- [ ] Cancel anytime **after** the Smartbrush is paid off: 12 months of monthly billing, or immediately on the annual plan.
- [ ] If a user cancels before the device is paid off, the remaining device balance is due at cancellation.
- [ ] After payoff, refills and Feno Premium continue until the user cancels. No lock-in beyond device payoff.

### Product facts to keep accurate in copy
- [ ] Smartbrush: 18,000 bristles, 250 strokes per 20 seconds.
- [ ] TrueFit Mouthpiece: personalized to the user's mouth size via an in-app scan.
- [ ] Feno Foam: foaming, contains xylitol, nano-hydroxyapatite, fluoride-free.

### Intentional — do not "fix"
- [ ] Internal value `membership` maps to the user-facing label **Subscribe & Save**. Do not rename in the UI.
- [ ] Wireframe placeholders are deliberate. They get replaced by Part B assets, not removed.
- [ ] Em dashes appear in some legacy prototype body copy. Final copy should avoid them (house style), but that is a copy pass, not a bug.

---

## Part B — Bloom Design: image and video assets

Every asset below is currently a dashed placeholder. Rendered sizes are measured from the prototype at desktop (1280px) and mobile (390px). "Deliver at" is roughly 2x rendered for retina. All product imagery should share one consistent look (background, lighting, angle language).

### 1. Gallery (PDP hero) — highest priority
Six views. Each main image is a **square (1:1)** and doubles as its own thumbnail (square crop).

| View | Rendered (desktop main) | Rendered (mobile main) | Thumb | Deliver at |
|---|---|---|---|---|
| Main product view | 524 x 524 | full width, 1:1 (~360) | 64 / 46 sq | 1200 x 1200 |
| Foam tubes | same | same | same | 1200 x 1200 |
| Smartbrush | same | same | same | 1200 x 1200 |
| Mouthpiece (TrueFit) | same | same | same | 1200 x 1200 |
| App view | same | same | same | 1200 x 1200 |
| In use (lifestyle) | same | same | same | 1200 x 1200 |

- Desktop layout: a 64px vertical thumbnail strip on the left, large square main on the right.
- Mobile layout: full-width square main, horizontal thumbnail row below.
- A "Special Monthly Pricing" tag overlays the top-left of the main image. That is a UI element, not part of the photo. Leave headroom there.

### 2. Feature spotlights — 3 banners
Each spotlight is text on one side, a visual on the other (alternating sides on desktop, stacked on mobile).

| Spotlight | Subject | Shape | Rendered visual | Deliver at |
|---|---|---|---|---|
| App dashboard | Feno Premium app UI / biomarker dashboard | Portrait phone (~9:17) | ~210 wide | 600 x 1100 |
| TrueFit mouthpiece | The custom mouthpiece, hero product shot | Square or 4:3 | ~340 wide | 800 x 800 |
| Foam refills | The 3 foam tubes | Square or 4:3 | ~340 wide | 800 x 800 |

- Desktop visual column is ~593px wide; the asset sits centered within it.
- The app dashboard should look like a real app screen (scores, trends, dentist sync).

### 3. Product demo video — "See it in action"
- [ ] One demo video, **16:9**. Suggested content: the scan, the brush, and what the app shows after (about 90 seconds).
- Rendered up to 760px wide (desktop) / full width (mobile).
- Deliver 1920 x 1080, plus a **poster frame** (still image) for before-play. Hosting can be a file or a YouTube/Vimeo embed — confirm with dev.

### 4. What's in the box — 6 product shots
**4:3 aspect**, consistent style. Desktop shows them 3 across (~244px wide each), mobile 1-2 across.

| Item | Deliver at |
|---|---|
| Feno Smartbrush | 640 x 480 |
| TrueFit Mouthpiece | 640 x 480 |
| 3 Feno Foam tubes | 640 x 480 |
| Tongue scraper | 640 x 480 |
| Wireless charger | 640 x 480 |
| Feno app | 640 x 480 |

### 5. Cart / checkout / upsell thumbnails — small
Small square product thumbnails used in the One Time cart, checkout summary, and post-purchase upsell. Rendered 44 to 56px.

| Where | Items | Deliver at |
|---|---|---|
| Cart cross-sell | Refill, App | 128 x 128 |
| Checkout summary | Bundle, Plan, Subscription, App | 128 x 128 |
| Post-purchase upsell | Refill, App | 128 x 128 |

### 6. Social / OG image
- [ ] One Open Graph image for link previews, **1200 x 630**. (The page references `og:image`; supply the real one.)

### General image guidance
- [ ] Deliver web-optimized (WebP preferred, JPEG/PNG fallback). Keep each under ~200KB where possible.
- [ ] Provide retina (2x) for everything; sizes above already account for it.
- [ ] Provide descriptive **alt text** for every image (accessibility + SEO).
- [ ] Square crops (gallery, thumbnails) should keep the product centered with even margin so a center-crop never clips it.
