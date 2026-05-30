# Feno Offer + PDP — Handoff

Two static HTML pages that simulate the full Feno purchase flow:

- **Homepage** (`/`) — short choice page: Subscribe & Save vs One Time, with a pull-up sheet on mobile.
- **PDP** (`/pdp`) — full product detail page: gallery, buy box, feature spotlights, what's in the box, how it works, reviews, FAQ.

Both pages share the same downstream **slide cart → simulated Shopify checkout → post-purchase upsell** flow. **No backend, no real payments, no analytics.** Wireframe-grade prototype, but the interactions are real.

---

## Quick start (local)

The whole thing is two HTML files plus a `vercel.json`. To run it locally:

```bash
# from the repo root
cd feno-membership-preview

# pick any static server. all of these work:
python3 -m http.server 8000              # then visit http://localhost:8000
npx serve .                              # then visit http://localhost:3000
vercel dev                               # closest to production behavior (honors cleanUrls so /pdp works)
```

**Important about routing**: `vercel.json` sets `cleanUrls: true`, so on Vercel and `vercel dev` the PDP lives at `/pdp`. With plain `python -m http.server` you'll need to visit `/pdp.html` because no clean-URL rewrite happens. `npx serve` handles cleanUrls if you add `--single` or use a `serve.json` shim.

Just opening `index.html` from the filesystem (`file://...`) also works for the homepage but the `/pdp` link will fail — use a server.

---

## Where things live

| Thing | Location |
|---|---|
| Production URL | https://feno-membership-preview.vercel.app |
| Homepage source | `feno-membership-preview/index.html` |
| PDP source | `feno-membership-preview/pdp.html` |
| Vercel config | `feno-membership-preview/vercel.json` |
| Git branch | `claude/jolly-faraday-RSYnY` |
| PR | https://github.com/fenomktg/feno-marketing/pull/1 (draft) |
| Latest commit at handoff time | `d8d7cb9` |

Vercel project: `feno-membership-preview` under the `brandon-1935` account. Re-deploy with `cd feno-membership-preview && vercel --prod --yes`.

---

## File anatomy

Both files are self-contained: HTML + inline `<style>` + inline `<script>`. **No build step, no dependencies.**

```
feno-membership-preview/
├── index.html      ~1900 lines — homepage
├── pdp.html        ~2200 lines — PDP
├── vercel.json     static hosting config + cleanUrls + security headers
└── HANDOFF.md      this file
```

### Why two files instead of shared modules

When PDP was added, the cart drawer + checkout overlay + post-purchase overlay code was **duplicated** from index.html into pdp.html rather than extracted. Reasoning at the time: ship faster, keep it dead simple, refactor later when changes start hurting. If you find yourself editing the cart in two places, that's the signal to extract `cart.css` + `cart.js` and `<link>` / `<script src>` them.

---

## The two pages, at a glance

### `/` — Homepage

Short page, big choice. Mobile-first.

- Eyebrow + serif headline + subhead about biomarkers
- Visual hero placeholder (will become the Bloom-designed graphic)
- Trust line: `30-day money back · Free shipping · Ships in 2 days`
- **Plan cards** (mobile only — desktop shows the buy box inline):
  - Subscribe & Save card (featured, expanded by default) — `$35/mo`, descriptive headline, 3-bullet teaser, dark inline CTA
  - One Time card (collapsed by default) — `$299`, descriptive headline, expandable
- "Or" divider between cards
- "Learn more about Feno →" link beneath the cards, pointing to `/pdp`
- Floating `?` button to re-open the demo disclaimer modal

Tap a plan card → **pull-up sheet** slides up with the full buy box (toggle, billing/addon, 6-slot included grid, sticky CTA). Confirm in the sheet → flow.

### `/pdp` — Product Detail Page

Long-form page with the same buy machinery.

Section order (top to bottom):
1. Promo banner: `Holiday Bundle Deal · While stocks last · Back to homepage`
2. Top nav: Feno® logo, menu, Log In, Buy Now pill, Cart
3. **Hero grid**:
   - Gallery (6 thumbs + main image with `Special Monthly Pricing` tag)
   - Buy box lead (badges + title + price)
   - Buy box purchase (toggle + plan card + Buy Now CTA)
4. Value callout — `$530+ worth · all for $299 / $35/mo`
5. **Feature spotlights** (3 full-bleed banners, alternating layout, gradient backgrounds + ambient orbs):
   - Analytics — `Your mouth, mapped.` (blue)
   - TrueFit Mouthpiece — `Built for your bite.` (bronze, reverse layout)
   - Feno Foam — `Not toothpaste. Foam.` (mint)
6. What's in the box — 6-item grid (Smartbrush, mouthpiece, foam, scraper, charger, app)
7. How it works — 3-step flow (brush + scan, app maps biomarkers, sync with dentist)
8. Reviews — 4.8★ summary + 3 placeholder cards (verified dentist + 2 buyers)
9. FAQ — 7 accordions
10. Specifications + Shipping accordions
11. Sticky mobile Buy Now bar — fixed-bottom, appears after the buy box scrolls off-screen

---

## Shared patterns (read this before editing)

### `data-target` sync

Buy-box content (price, included list, terms, incentive pill, CTA text, etc.) **exists twice** on the homepage: once in the desktop card, once in the mobile sheet. Both elements carry matching `data-target="..."` attributes.

`render()` writes to **every element with the matching attribute** via:

```js
function setText(name, text)   // updates every [data-target="name"]
function setHTML(name, html)
```

So **don't query by ID** when adding new dynamic fields — use `data-target` and they'll stay in sync between contexts automatically.

The PDP also uses `data-target` for the buy-box fields and the sticky Buy Now bar's price/label.

### Internal `membership` vs user-facing "Subscribe & Save"

Throughout the JS and HTML, the internal state name is `membership`:

- State variable: `primary === 'membership' | 'onetime'`
- HTML attributes: `data-tab="membership"`, `data-plan="membership"`, `id="tab-membership"`
- The Vercel slug: `feno-membership-preview.vercel.app`

But all **user-facing copy** says "Subscribe & Save". Renaming the internals was deliberately skipped — it cascades through CSS, JS, and the deploy URL with zero user benefit.

### The `purchasedAs` flag

When `openCheckout()` is called, it captures `purchasedAs = primary`. This freezes the purchase type so that if the user flips the tab on the underlying page while the checkout overlay is open, the checkout summary + post-purchase view stay consistent.

- `purchasedAs === 'membership'` → checkout shows the single plan line ($35 or $360), post-purchase says "Thanks for your order" with no upsells.
- `purchasedAs === 'onetime'` → checkout shows the bundle + any addons + discount rows, post-purchase shows the upsells for items not yet on the order.

---

## The pricing matrix (One Time path)

Same totals whether items are added via the upfront refills checkbox, in the slide cart, or accepted post-purchase:

| On the order | Charged today |
|---|---|
| Bundle only | $299 |
| Bundle + Quarterly Refills | $269 (−$30 first-order bonus) |
| Bundle + Feno Premium app | $309 (+$10 first month) |
| Bundle + Refills + Feno Premium | $264 (−$30 −$15 bundle savings) |

Subscribe & Save path is simpler: $35/month or $360/year. Refills + Feno Premium are included.

The math lives in `todayTotal()` and `subtotalToday()` near the top of the `<script>`.

---

## Key state variables

At the top of each page's `<script>`:

| Variable | Meaning |
|---|---|
| `primary` | `'membership'` (Subscribe & Save) or `'onetime'` — current tab |
| `billing` | `'monthly'` or `'annual'` (Subscribe & Save only) |
| `qrAdded` / `appAdded` | Whether Quarterly Refills / Feno Premium is on the order |
| `qrDiscountEarned` | The −$30 first-order bonus is active |
| `bundleSavingsEarned` | The −$15 bundle discount is active (both subs on order) |
| `qrCheckedUpfront` | Refills was committed via the product-page checkbox |
| `qrFromPostPurchase` / `appFromPostPurchase` | Item was added at the post-purchase step (post-purchase doesn't grant the $30 retroactively but the UI tracks it) |
| `purchasedAs` | `primary` captured at `openCheckout()` time |
| `expandedPlan` | (homepage only) which mobile plan card is expanded in the accordion |

---

## CRO measurements at the time of this handoff

Verified via Playwright across iPhone SE / 14 / Pixel 6 / Desktop 13" / Desktop 15":

- **PDP Buy Now CTA above the fold on every measured viewport** (iPhone SE: 649px CTA top on 667px fold, 18px clearance; iPhone 14: 829px on 844px fold, 15px clearance)
- **Zero awkward line wraps** (no orphan 1-4 char trailing lines) on the main page across all viewports — only remaining orphan is inside the slide-cart drawer's upsell sub copy, which is a secondary overlay
- **No horizontal overflow** anywhere

If you change copy, make sure the lede or bullet doesn't end with a 1-3 char orphan word. The audit script is at `/tmp/cro-audit.mjs` on the original dev environment; you can reconstruct it by walking the DOM with `range.getClientRects()` to count line boxes per element.

---

## Testing

There's no test suite. Flows have been verified end-to-end with Playwright (Chromium) at multiple viewport widths. Quick boilerplate to drive a flow from a fresh checkout:

```js
import { chromium } from 'playwright';
const b = await chromium.launch();
const ctx = await b.newContext({ viewport: { width: 390, height: 844 } });
const p = await ctx.newPage();
await p.goto('https://feno-membership-preview.vercel.app/pdp');
await p.evaluate(() => localStorage.setItem('feno-demo-seen', '1')); // skip the disclaimer
await p.reload();
// drive: click '[data-target="cta"]' on PDP, '.plan-card[data-plan="membership"]' on homepage, etc.
```

Useful selectors:
- Homepage: `#tab-onetime`, `#tab-membership`, `.plan-card[data-plan="membership"]`, `.plans-sheet .cta`, `#cart-checkout`, `#co-place`
- PDP: same plus `[data-target="cta"]` for the inline Buy Now, `#sticky-cta` for the sticky bar
- Cart drawer: `#cu-qr-btn`, `#cu-app-btn`, `#cart-checkout`, `#cart-close`
- Checkout: `#co-total`, `#co-place`, `#co-back`, `#co-item-bundle`, `#co-item-membership`
- Post-purchase: `#pp-title`, `#pp-upsell-qr`, `#pp-upsell-app`, `#pp-qr-accept`, `#pp-app-accept`

Manual QA checklist worth running after any change:
- Subscribe & Save monthly → checkout shows `$35.00` → "Thanks for your order", no upsells
- Subscribe & Save annual → checkout `$360.00` → same, with "Next charge $360 in 1 year"
- One Time + refills checkbox **unchecked** → slide cart → both upsells visible → Checkout $299 → post-purchase shows both upsells
- One Time + refills checkbox **checked** → cart skipped → checkout $269 directly
- Accept Refills post-purchase → today total drops to $269 with `−$30 first-order bonus` row
- Accept both post-purchase → drops further to $264 with `−$15 bundle savings`
- iPhone 14 (390×844) and iPhone SE (375×667) — Buy Now visible above the fold on the PDP
- Sticky bar appears after scrolling past the PDP buy box; live-updates its price as the tab changes
- "Learn more about Feno →" on the homepage navigates to `/pdp`

---

## Intentional things — do not "fix"

- **Dev annotations are deliberate**: dashed-border media placeholder on homepage, italic `Bloom-designed product photography goes here` note inside the gallery, `Special Monthly Pricing` purple tag, italic `Placeholder reviews shown — real reviews populate from Yotpo once live.` under the reviews block.
- **Demo disclaimer modal** ("Two flows to try" / "Test it out") on homepage is intentional framing for reviewers — not a real consent gate. Floating `?` re-opens it.
- **Pre-filled Shopify checkout** (`jane@feno.co`, `Jane Smith`, `123 Mission St`, card `•••• 4242`) is intentional so a reviewer can hit Place Order without typing.
- **Internal `membership` vs user-facing `Subscribe & Save`** — see Shared patterns above.
- **No frameworks / no third-party scripts** — keep it two static HTML files until shared modules are extracted on purpose.

---

## Open items

- The PDP duplicates the cart/checkout/post-purchase flow code from the homepage. First refactor candidate: extract `flow.css` + `flow.js` and `<link>` / `<script src>` them into both pages.
- Real Bloom photography → replaces the placeholder shapes in the homepage hero, PDP gallery, PDP spotlights, and What's-in-the-box grid. All are CSS-driven placeholders today; swap `<img>` tags in.
- Reviews block uses placeholder cards; real reviews from Yotpo (or whichever review system) plug in via the same `.review-card` markup.
- Desktop: on the homepage, the One Time refills checkbox (~52px) and the Subscribe & Save Monthly/Annual toggle (~80px) aren't quite the same height, so the CTA nudges ~28px when toggling tabs. Could height-match them to fully lock.
- Annual plan currently saves $60/yr (~14% off monthly). If you want the marketing claim back to ~25%, drop the annual price to ~$315/yr.
- Sticky mobile bar on PDP is different from the homepage's pattern (homepage has no sticky; PDP has Amazon-style sticky-bottom). Deliberate — different page types. Documented above.

---

## What to edit, for common changes

| You want to change... | File · approximate location |
|---|---|
| Subscribe & Save monthly price ($35) | both files · grep `\$35` |
| Annual price ($360) | both files · grep `\$360` |
| One Time price ($299) | both files · grep `\$299` |
| Refills first-order bonus amount ($30) | both files · grep `30` carefully — there are 30-day refs too |
| Bundle savings amount ($15) | both files · grep `15` carefully |
| What's in the box items | `pdp.html` — search `<div class="box-item">` |
| Spotlight copy | `pdp.html` — search `class="spotlight"` |
| FAQ questions | `pdp.html` — search `<summary>` near the FAQ section |
| Placeholder reviews | `pdp.html` — search `<article class="review-card">` |
| Demo disclaimer body (homepage) | `index.html` — search `disclaimer-flow__desc` |

---

*Two-file static prototype. Edit `index.html` or `pdp.html`, commit, `vercel --prod --yes` from `feno-membership-preview/`. PR #1 reflects the working branch.*
