# Feno Offer Page — Handoff

UX prototype of the Feno offer page: a two-plan buy box (**Subscribe & Save** vs
**One Time**) with a mobile pull-up sheet, slide cart, upsells, a simulated
Shopify checkout, and a post-purchase upsell screen.

**It is a wireframe demo, not production code.** No backend, no real payments,
no analytics. Everything is one static HTML file driven by client-side JS.

---

## Where things live

| Thing | Location |
|---|---|
| Production URL | https://feno-membership-preview.vercel.app |
| Source | `feno-membership-preview/index.html` (single file, ~1820 lines) |
| Vercel config | `feno-membership-preview/vercel.json` (static hosting, no build) |
| Git branch | `claude/jolly-faraday-RSYnY` |
| PR | https://github.com/fenomktg/feno-marketing/pull/1 (draft) |
| Latest commit | `f41b766` |

The Vercel project is `feno-membership-preview` under the `brandon-1935`
account (team slug `brandon-theplaybooks-projects`).

---

## Deploying

The whole thing is one static file. No build step.

```bash
cd feno-membership-preview
vercel --prod --yes
```

That re-aliases `feno-membership-preview.vercel.app` to the new deployment.
The CLI is authenticated interactively (device flow) — if the token has
expired, run `vercel login` first.

Commit + push to the branch as normal; the PR updates automatically. There is
**no** CI on the repo, so deploys are manual via the command above.

---

## How the page is structured

Everything is in `index.html`:

- **`<style>`** — all CSS. Desktop-first; mobile overrides live in
  `@media (max-width: 768px)` with extra tightening at `max-width: 360px`.
- **`.section`** — the desktop offer card (two-column: media placeholder +
  buy box).
- **Mobile** swaps to a different layout (see below): visual hero + two plan
  cards + a pull-up sheet that holds the buy box.
- **Overlays** (all fixed-position, layered above the page):
  - `#cart-drawer` — slide cart (One Time only)
  - `#checkout-overlay` — Shopify-style checkout simulation
  - `#post-purchase-overlay` — order confirmation + upsells
  - `#plans-sheet` — mobile pull-up sheet
  - `#disclaimer-modal` — first-load demo explainer
- **`<script>`** — all logic at the bottom. No framework, no dependencies.

### The `data-target` pattern

The buy-box content (price, included list, CTA, terms, incentive pill) exists
**twice**: once in the desktop `.card` and once in the mobile `.plans-sheet`.
Both copies carry matching `data-target="..."` attributes. `render()` writes to
**all** matching elements at once via `setText()` / `setHTML()`, so the two
stay in sync. Same idea for the tab toggle (`[data-tab]`), billing toggle
(`[data-bill]`), and addon checkbox (`[data-target="addon-checkbox"]`, synced
on change).

When you add a field, give it a `data-target` and update it in `render()` —
don't query a single ID.

---

## The two flows

### Flow 1 — Subscribe & Save (`primary === 'membership'`)

> Internal code still calls this `membership` (state var, element IDs,
> `data-plan`/`data-tab` values). Only the **user-facing copy** says
> "Subscribe & Save". Renaming the internals was deliberately skipped — it
> cascades through CSS/JS for zero user benefit.

- Monthly $40/mo or Annual $360/yr (toggle in the buy box).
- Refills + Feno Premium are **included** in the plan.
- CTA → Shopify checkout directly (no cart).
- Checkout summary shows a single `Subscribe & Save · Monthly/Annual` line.
- Post-purchase: **"Thanks for your order", no upsells** (there's nothing left
  to upsell — it's all in the plan). Shows plan total + next-charge date.

### Flow 2 — One Time (`primary === 'onetime'`)

- $299 one-time Smartbrush bundle.
- Buy box has an **Add Quarterly Refills** checkbox (the $30 first-order
  carrot).
- CTA routing:
  - **checkbox checked** → straight to Shopify checkout, total $269.
  - **checkbox unchecked** → slide cart opens first.
- **Slide cart** shows the bundle + a savings progress bar + two upsell cards:
  - **Quarterly Refills** — $65/qtr, carries the −$30 first-order bonus.
  - **Feno Premium** — $10/mo, no discount on its own.
  - Adding **both** unlocks an extra −$15 "bundle savings".
- Checkout → Place Order → **post-purchase upsells** for whatever the user did
  *not* already add. One-click add, no second checkout.

### Pricing matrix (One Time)

| On the order | Charged today |
|---|---|
| Bundle only | $299 |
| Bundle + Refills | $269 (−$30) |
| Bundle + Feno Premium | $309 (+$10 app) |
| Bundle + Refills + Feno Premium | $264 (−$30 −$15) |

Same totals whether the items are added via the upfront checkbox, in the slide
cart, or accepted post-purchase. The math lives in `todayTotal()` /
`subtotalToday()`.

---

## Key state variables (top of `<script>`)

| Var | Meaning |
|---|---|
| `primary` | `'membership'` or `'onetime'` — current tab |
| `billing` | `'monthly'` or `'annual'` (Subscribe & Save only) |
| `qrAdded` / `appAdded` | Quarterly Refills / Feno Premium on the order |
| `qrDiscountEarned` | the −$30 first-order bonus is active |
| `bundleSavingsEarned` | the −$15 bundle discount is active (both subs on order) |
| `qrCheckedUpfront` | refills committed via the product-page checkbox |
| `qrFromPostPurchase` / `appFromPostPurchase` | added at the post-purchase step |
| `purchasedAs` | `primary` captured at `openCheckout()` — drives checkout + post-purchase rendering so it stays correct even if the underlying tab changes while overlays are open |
| `expandedPlan` | which mobile plan card is expanded (accordion) |

---

## Mobile specifics

- **Plan cards are an accordion**: the featured (Subscribe & Save) card is
  expanded, the other collapses to a one-row summary (`SUBSCRIBE & SAVE · $40/mo`
  + descriptive headline on a second grid row). Tap a collapsed card to expand;
  tap an expanded card to open the sheet. `onPlanCardClick()` handles this.
- **Pull-up sheet** (`#plans-sheet`) holds the full buy box. Opens via
  `openSheetWith(plan)`. Its sticky-footer CTA runs `handleSheetCtaClick()`,
  which closes the sheet (180ms) then runs the standard flow.
- The desktop media placeholder is hidden on mobile; the visual hero shows a
  compact phone+tube shape.
- The **demo disclaimer** modal shows on first load (`localStorage` key
  `feno-demo-seen`). The floating **`?`** button (bottom-right) re-opens it.

---

## Intentional things — do not "fix"

- **Dev annotations are deliberate** (the reviewer wants them):
  - Dashed-border media box with "Bloom-designed graphic goes here".
- **Demo disclaimer modal** ("Two flows to try" / "Test it out") is intentional
  framing for reviewers, not a real consent gate.
- **Pre-filled Shopify checkout** (jane@feno.co, etc.) is intentional so a
  reviewer can hit "Place Order" without typing.
- **Internal `membership` naming** vs user-facing "Subscribe & Save" — see Flow
  1 note above.
- **No frameworks / no third-party scripts** — keep it one static HTML file.

---

## Testing / screenshots

There's no test suite. Flows were verified with Playwright (Chromium) during
development — handy snippet for re-checking after a change:

```js
// node script using /opt/node22/lib/node_modules/playwright (env-specific)
import { chromium } from 'playwright';
const b = await chromium.launch();
const ctx = await b.newContext({ viewport: { width: 390, height: 844 } });
const p = await ctx.newPage();
await p.goto('https://feno-membership-preview.vercel.app');
await p.evaluate(() => localStorage.setItem('feno-demo-seen', '1')); // skip modal
await p.reload();
// ...drive the flow, assert on #co-total / #pp-title / etc.
```

Useful selectors: `#tab-onetime`, `.plan-card[data-plan="membership"]`,
`.plans-sheet .cta`, `#cart-checkout`, `#co-place`, `#co-total`, `#pp-title`,
`#pp-upsell-qr`, `#pp-upsell-app`.

Manual QA checklist worth running after any change:
- Both tabs on desktop fit in one viewport; CTA doesn't jump when toggling.
- Subscribe & Save monthly → checkout $40 → "Thanks for your order", no upsells.
- Subscribe & Save annual → checkout $360 → same, next charge in 1 year.
- One Time unchecked → cart → both upsells → checkout $299 → post-purchase shows
  both upsells.
- One Time checked → skips cart → checkout $269.
- Accept each post-purchase upsell → today's total updates correctly.
- No horizontal scroll 320px → 1920px.

---

## Open items / ideas (not done)

- Desktop: the One Time refills checkbox (~52px) vs the Subscribe & Save
  Monthly/Annual toggle (~80px) aren't the exact same height, so the CTA nudges
  ~28px when switching tabs. Could height-match them to fully lock it.
- The media placeholder is still a wireframe box — drop in the real Bloom
  asset when it's ready (replace `.media` contents; it already collapses on
  mobile).
- Copy is final-ish but unreviewed by legal (the cancel/refund terms strings
  are placeholders).

---

*Single-file static prototype. Edit `index.html`, commit to
`claude/jolly-faraday-RSYnY`, run `vercel --prod --yes` from
`feno-membership-preview/`.*
