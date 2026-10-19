# SmartCart Design Reference (DESIGN.md)

## 1. Purpose and sources

Design reference for a new SmartCart frontend, drawing on the quick-commerce patterns of Blinkit and Zepto. SmartCart takes the patterns (information density, one-tap add, sticky cart, bill breakdown). It does not take brand assets, logos, names, fonts or copy.

Research date: 2026-10-19.

**Detected SmartCart stack.** A single-file Flask app (`SmartCart.py`, Flask >= 3) that serves its UI with `render_template_string`. Styling is plain inline CSS using CSS variables in `:root`. Existing tokens are `--bg --card --bd --ink --mut --green #2E7D46 --lime #D7E66A --red --amb`. Fonts are Inter and Space Grotesk from Google Fonts. The only static asset is `static/products/grocery-hero.jpg`. There is no Tailwind, no bundler and no package.json. The UI is a fixed 1280x800 kiosk-style stage that scales with a transform. The backend also covers OCR (pytesseract/OpenCV) and serial (pyserial), so this looks like a smart-cart / self-checkout kiosk. This document gives tokens as CSS variables first, and a Tailwind snippet is included only as an optional path.

**Sources and verification level**
| Item | Method | Confidence |
|---|---|---|
| Zepto (zepto.com) | Live computed styles via in-app browser, no location needed (header, category list, product card, ADD button, price chip) | High for values in the Zepto column |
| Blinkit (blinkit.com) | Live computed styles for header, fonts, section headings, cart button. A location modal ("Detect my location" / search) blocked the product rails. I did not enter a location. | Medium. Product-card details for Blinkit are NOT verified (see section 2 notes). |
| Public design write-ups | A web search found only opinion pieces (e.g. a Design Bootcamp article on one-tap add) and no official Blinkit design-system or hex documentation | Low |

Blinkit values marked (v) were read from the live page. Values marked (u) are from general familiarity and are unverified.

## 2. Side-by-side comparison

| Aspect | Blinkit | Zepto |
|---|---|---|
| Brand feel | Friendly, utilitarian, white page, green action color (cart green rgb(12,131,31) = #0C831F (v)). Logo is yellow/green wordmark. | Playful, high-contrast. Hot pink-red primary rgb(249,16,94) = #F9105E (v), violet/lavender tints (rgb(250,245,255) = #FAF5FF (v)), purple-ish ink rgb(43,30,53). |
| Body font | "Okra", Helvetica (v), proprietary geometric sans | "Zepto Norms" (proprietary), system-ui fallback (v) |
| Page background | #FFFFFF (v). Body text #666666 (v). Headings #1F1F1F (v). Placeholder #999999 (v). | White header and cards, lavender section washes. Text black, secondary rgb(88,98,116) = #586274 (v), name text #101418 (v). |
| Header | Left: logo, "Delivery in 11 minutes" (18px/800 (v)) and "Select Location" (13px (v)). Middle: wide search with rotating `Search "milk"` placeholders (14px #999 (v)). Right: Login text, green "My Cart" button (12px radius (v)). Header is about 60px tall inside a 128px block with the location row. | Left: logo, "Delivery in minutes*", "Select Location". Middle: search (white, 8px radius, placeholder `Search for "kurkure"` (v)). Right: login and cart icons with labels. 90px header, 1px #E5E7EB bottom border (v). |
| Navigation | Left category sidebar on listing pages (u). Home shows grouped category grid. | Horizontal tab strip under header (All, Cafe, Home, Toys, Fresh, Electronics, Mobiles, Beauty...), active tab in violet (v). |
| Category tiles | Grouped under bold section headings ("Grocery & Kitchen" 17px/800 #1F1F1F, tile label 13px/600 #1F1F1F (v)). Image tiles, 72px images (v). | "Shop by Category", 86px wide tiles, 16px/500 labels, 2-line centred wrap, about 10 per row on desktop (v). |
| Banners | Hero promos, pale pastel cards (u) | Two-up banner row: promo-fees card and partner banner, rounded, pastel gradient (v from screenshot) |
| Product card | (u) White card, image, small timer pill, name, weight, price, green-outlined ADD | 140px wide, 268px tall, radius 10px (v). Price chip is green (#329537 bg, #084121 border, 1.5px hard offset shadow, 6px radius, 16px/700 white) (v). Struck MRP 14px/450 #586274 (v). Name 14px/500 #101418 (v). Variant 12px/450 #586274 (v). |
| ADD button | (u) light/white fill, green border and text, 8px radius | White fill, #F9105E text and 0.8px border, "pressed-key" hard shadow 1px 1px 0 #F9105E, 56x32px, 14px/600, 8px radius (v) |
| Stepper | (u) Green filled pill with - qty + | Same footprint as ADD (u) |
| Cart | Cart button in header shows "N items / total" then "View Cart" (v: "0 items", "₹ 0", "View Cart"); opens a right-side drawer (u) | Header cart icon with label. Mobile uses a sticky bottom bar (u). |
| Radius language | 12px on buttons (v), small otherwise | 4, 6, 8, 10, 12, pill 9999 (v). 8px dominant (139 occurrences). |
| Shadows | Mostly flat (v: header and search have none) | Flat cards. Chunky offset "pressed" shadows on key buttons and price chips (v). |
| Trust and delivery claims | ETA is the loudest header element ("Delivery in 11 minutes") | "Delivery in minutes*" plus a fee-free promo banner ("Rs 0 fees" style) |

Notes
- Takeaway across both: white canvas, one saturated action color, tiny dense type, product images carry the colour.
- Items marked (u) are standard industry patterns that I could not confirm on the live Blinkit page because of the location gate.
- Elements I did not observe on either site: checkout, bill details, order tracking, skeletons and toasts. Those specs below are SmartCart's own, built on common quick-commerce practice.

## 3. SmartCart design direction

SmartCart is a Flask app with a kiosk heritage (1280x800 stage, green + lime + warm cream). Keep that identity so the new UI feels continuous, and borrow the quick-commerce mechanics.

Principles
1. **Speed first.** One tap adds. The stepper replaces ADD in place with no layout shift.
2. **One action color.** Forest green (existing `--green`) is the only filled CTA color. Lime is a highlight (ETA, savings), never a button.
3. **Dense but breathable.** 4px base grid, 12 to 16px card padding, 2-line name clamp.
4. **Honest pricing.** Price, struck MRP and savings are always visible and tabular-aligned.
5. **Kiosk and responsive.** Works on the 1280x800 stage and on a 375px phone (cart bar at the bottom, drawer becomes a sheet).
6. **Original identity.** No pink, no yellow-green wordmark and no proprietary fonts. Use Inter and Space Grotesk (already loaded). Copy is SmartCart's own.

## 4. Design tokens

### 4.1 CSS variables (drop into the existing `:root`)

```css
:root{
  /* Color: brand */
  --sc-green-700:#25683A;   /* pressed */
  --sc-green-600:#2E7D46;   /* primary (existing --green) */
  --sc-green-100:#E0EBD6;   /* tint, badges (existing) */
  --sc-green-50:#F1F7EC;    /* hover wash, ADD button fill */
  --sc-lime-400:#D7E66A;    /* highlight (existing --lime) */
  --sc-amber-400:#F2D45C;   /* offer / warning (existing --amb) */
  --sc-red-600:#C94D40;     /* destructive, errors (existing) */
  /* Color: neutrals */
  --sc-bg:#F7F8F5;          /* page */
  --sc-surface:#FFFFFF;     /* cards, header */
  --sc-surface-2:#F1F3EE;   /* skeleton base, chips */
  --sc-border:#E5EAE2;
  --sc-ink:#252925;         /* text primary */
  --sc-ink-2:#4A524C;       /* text secondary (AA on white) */
  --sc-mut:#6B7C73;         /* captions, MRP (AA on white for 14px+, see a11y) */
  /* Semantic */
  --sc-discount-bg:#E0EBD6; --sc-discount-fg:#1F5C32;
  --sc-eta-bg:#252925;      --sc-eta-fg:#D7E66A;
  --sc-focus:#1B5FD6;
  /* Type */
  --sc-font:'Inter',system-ui,sans-serif;
  --sc-font-display:'Space Grotesk','Inter',sans-serif;
  --sc-text-xs:.6875rem;   /* 11 ETA badge */
  --sc-text-sm:.75rem;     /* 12 variant, MRP */
  --sc-text-base:.875rem;  /* 14 name, body */
  --sc-text-md:1rem;       /* 16 price, labels */
  --sc-text-lg:1.125rem;   /* 18 section title */
  --sc-text-xl:1.5rem;     /* 24 page title */
  --sc-text-2xl:2rem;      /* 32 hero */
  --sc-lh-tight:1.2; --sc-lh-body:1.45;
  /* Spacing (4px grid) */
  --sc-1:4px; --sc-2:8px; --sc-3:12px; --sc-4:16px; --sc-5:20px; --sc-6:24px; --sc-8:32px; --sc-10:40px;
  /* Radius */
  --sc-r-sm:6px; --sc-r-md:10px; --sc-r-lg:14px; --sc-r-xl:20px; --sc-r-pill:999px;
  /* Shadow */
  --sc-sh-1:0 1px 2px rgba(37,41,37,.06);
  --sc-sh-2:0 4px 14px rgba(37,41,37,.10);   /* drawer, popovers */
  --sc-sh-key:1px 1px 0 var(--sc-green-700);  /* optional "pressed key" accent, ADD only */
  /* Motion */
  --sc-ease:cubic-bezier(.2,.8,.2,1);
  --sc-t-fast:120ms; --sc-t-base:200ms; --sc-t-slow:320ms;
  /* Layout */
  --sc-header-h:72px; --sc-container:1200px; --sc-tap:44px;
}
@media (prefers-reduced-motion:reduce){:root{--sc-t-fast:0ms;--sc-t-base:0ms;--sc-t-slow:0ms}}
```

### 4.2 Type scale

| Role | Size / weight | Font |
|---|---|---|
| Page title | 24 / 700 | Space Grotesk |
| Section title (rail) | 18 / 700 | Space Grotesk |
| Product name | 14 / 500, 2-line clamp | Inter |
| Variant / weight | 12 / 500, `--sc-mut` | Inter |
| Price | 16 / 700, tabular-nums | Space Grotesk |
| MRP | 12 to 14 / 400, line-through, `--sc-mut` | Inter |
| Button | 14 / 600 | Inter |
| ETA badge | 11 / 700, uppercase tracking .04em | Inter |
| Bill total | 20 / 700, tabular-nums | Space Grotesk |

### 4.3 Optional Tailwind theme extension (only if Tailwind is introduced via CDN or build)

```js
// tailwind.config
theme:{extend:{
  colors:{sc:{green:{50:'#F1F7EC',100:'#E0EBD6',600:'#2E7D46',700:'#25683A'},
    lime:'#D7E66A',amber:'#F2D45C',red:'#C94D40',
    bg:'#F7F8F5',surface:'#FFFFFF',border:'#E5EAE2',ink:'#252925',mut:'#6B7C73'}},
  fontFamily:{sans:['Inter','system-ui','sans-serif'],display:['Space Grotesk','Inter','sans-serif']},
  borderRadius:{sm:'6px',md:'10px',lg:'14px',xl:'20px'},
  boxShadow:{1:'0 1px 2px rgba(37,41,37,.06)',2:'0 4px 14px rgba(37,41,37,.10)'},
  transitionTimingFunction:{sc:'cubic-bezier(.2,.8,.2,1)'}
}}
```

## 5. Component specs

Common: every interactive element is at least 44x44px on touch (use padding if the visual is smaller), has a 2px `--sc-focus` outline with 2px offset on `:focus-visible`, and respects reduced motion.

### Header
- Anatomy (desktop, 72px, sticky, white, bottom border): logo area, LocationPicker trigger (ETA line bold plus address line truncated), SearchBar (flex-grow, max 640px), Account button, Cart button (green, shows item count and total).
- Mobile: two rows. Row 1 holds the location/ETA on the left and the account icon on the right. Row 2 holds a full-width search. Cart moves to the bottom CartBar.
- States: scrolled (adds `--sc-sh-1`), cart-empty (neutral outline button, label "Cart"), cart-filled (green fill, `N items | total`).

### LocationPicker
- Trigger shows ETA ("Delivery in ~N min") and the area. Opens a modal on desktop, a bottom sheet on mobile.
- Content: "Use my location" primary button, "or" divider, pincode/area search input, saved addresses list.
- States: unset (modal opens on first visit and can be dismissed, browsing stays allowed), loading, serviceable, unserviceable (inline error with alternative).

### SearchBar
- 44px high, radius `--sc-r-md`, white fill, 1px border, leading magnifier icon, placeholder that cycles through examples every 3s (`Search "milk"`) with a fade and slide, paused when focused or in reduced motion.
- Focus opens a panel: recent searches, trending chips, then live suggestions after 2 characters (debounce 200ms) with product thumbnails. Enter goes to the Search page. Esc closes it.
- Empty result: friendly message plus category shortcuts.

### CategoryTile
- Image area 72px (mobile) to 88px (desktop), soft `--sc-surface-2` rounded square (radius 14px), label below 12 to 13px / 600, centred, 2-line clamp.
- Grid: 4 columns on mobile, 6 on tablet, 10 on desktop. Hover lifts by 2px, active scales to .97.

### BannerCarousel
- 16:6 desktop and 16:9 mobile, radius `--sc-r-lg`, scroll-snap-x with partial next slide visible, dots below, autoplay 5s paused on hover or touch. Arrow buttons on desktop only.
- Content-safe zone: text on the left 55%, image on the right.

### ProductRail
- Title (18/700) plus "See all" text link, then a horizontal scroll-snap row of ProductCards with 12px gap. Desktop shows arrow buttons on hover. Mobile uses edge-bleed scrolling with no visible scrollbar.
- Card width 140px (mobile) to 160px (desktop).

### ProductCard
- Anatomy, top to bottom: image (square, `object-fit:contain`, on `--sc-surface-2`, radius 10), with ETA pill top-left and discount badge top-right (see below), then ADD / stepper pinned bottom-right overlapping the image edge, then price row, then name (14/500, 2-line clamp), then variant (12, muted).
- Price row: current price (16/700) next to struck MRP (12, muted). Optional "Save Rs X" lime-green chip. Never render the discount badge without a real MRP greater than price.
- Discount badge: `--sc-discount-bg` with `--sc-discount-fg`, 11/700, radius 6, text like "12% OFF".
- ETA pill: `--sc-eta-bg` / `--sc-eta-fg`, 11/700, with a small clock icon.
- States: default, hover (shadow-1, image zoom 1.03), out of stock (image 50% opacity, button replaced by "Notify me", no price emphasis), loading (skeleton), added (stepper).
- Whole card links to the Product detail page. The button stops propagation.

### AddToCartStepper
- ADD state: 56x32 (visual), 44px hit area, white/`--sc-green-50` fill, 1px green border, green text 14/600, radius 8, optional `--sc-sh-key`.
- Active state: replaces ADD in the same box, green fill, white text, `-  qty  +`, each control 32px wide with 44px hit area, qty is tabular-nums. Animates with a 120ms scale-in.
- Behaviour: `-` at qty 1 removes the item (becomes ADD again). `+` stops at the stock limit and shows a toast ("Max N per order"). Announce changes with `aria-live="polite"` ("Milk, quantity 2").
- Label for screen readers: "Add Milk 500 ml to cart", "Decrease quantity", "Increase quantity".

### CartBar / CartDrawer
- **CartBar (mobile, and kiosk compact):** fixed bottom, 56px, green fill, radius 14, 12px side margin, safe-area inset. Left: stacked thumbnails (up to 3) and "N items". Right: total and "View cart". Slides up 200ms when the first item is added, hides when the cart is empty. Content gets bottom padding so nothing is covered.
- **CartDrawer (desktop):** right panel 400px, white, `--sc-sh-2`, backdrop 40% ink. Header ("My cart", item count, close), a delivery-time strip ("Delivery in N min"), item list (thumbnail 48px, name, variant, stepper, line total), savings strip, "Bill details" (see below), and a sticky "Proceed" button. On mobile the same content opens as a full-height sheet.
- Empty: inline cart icon (consistent with the current app), short message, "Start shopping" button.
- Free-delivery progress: thin bar "Add Rs X more for free delivery".

### BillDetails
- Card with rows: Items total (struck MRP total plus discounted total), Delivery fee, Handling fee, Taxes if applicable, Discount (green text), and a divider before **Grand total** (20/700). Footer line: "You save Rs X on this order" in a lime-green strip.
- Each fee row has an info icon that opens a tooltip or popover explaining the charge. All numbers right-aligned with tabular-nums.

### CheckoutSteps
- Stepper with 3 steps: Address, Delivery and Payment, Review (the Review step can merge into Payment on mobile). On desktop it is a two-column page (form left, sticky summary right), on mobile single column with a sticky "Pay Rs X" bar.
- Address: saved address cards (radio), add-new form (inline validation, no autofill of invalid data). Delivery: ETA, optional tip chips. Payment: method list (UPI, card, cash), with the selected one expanded.
- Errors are shown inline plus an `aria-live` summary at the top. The primary button shows a spinner and is disabled while submitting.

### OrderTracking
- Vertical timeline: Confirmed, Packing, Out for delivery, Delivered. Active step has a pulsing green dot, completed steps are solid green with a check, upcoming are grey.
- Top: large ETA ("Arriving in N min") in Space Grotesk 32/700 on a lime-tinted card. Optional map placeholder (the existing app already has map UI) and an item summary accordion. Updates via polling or SSE (backend choice), with the text updating politely.

### Skeletons
- Same dimensions as the real component (no layout shift). `--sc-surface-2` blocks with a 1.4s shimmer gradient. Shown for category tiles, rails (6 cards), product grid, cart items and bill details.
- Show only after 150ms of loading to avoid flashes. With reduced motion, use a static block.

### Toast
- Bottom-centre (above CartBar on mobile), ink background, white text 14/500, radius 10, max 360px, 3s auto dismiss with a progress-free design, optional single action ("Undo", "View cart"). `role="status"`. Stack of at most 2.
- Types: success (green icon), info, error (red icon, persists until dismissed).

## 6. Page layouts

- **Home:** Header, category tab strip (optional), BannerCarousel, "Shop by category" grid, a series of ProductRails (Bestsellers, Dairy and bread, Fruits and veg, Snacks, Offers), footer with simple links. Mobile: CartBar fixed bottom.
- **Category / listing:** Desktop is a two-pane layout, left sidebar of sub-categories (180px, sticky, active item with green left border), right a responsive product grid (5 columns at 1280px, 3 at 768px, 2 at 375px). Top toolbar has a sort dropdown and filter chips. Mobile: sub-categories become a horizontal chip row. Infinite scroll with skeleton rows.
- **Product detail:** Two columns on desktop (gallery with thumbnails, then info). Info holds name, variant selector chips (size, pack), price block with MRP and savings, ADD/stepper (large 48px), delivery ETA, a "Why shop with us" short list, and accordions for description and nutrition. Related ProductRail below. Mobile: full-width gallery carousel, sticky bottom ADD bar.
- **Search:** Header search kept in focus. Results are a grid with a "Showing N results for ..." line, related category chips, and no-result state with trending suggestions.
- **Cart:** Desktop uses the drawer. Mobile uses a full page: item list, coupon row, BillDetails, delivery instructions, sticky "Proceed to checkout".
- **Checkout:** CheckoutSteps as described. The summary column mirrors BillDetails.
- **Order status:** OrderTracking, delivery partner card (optional), order summary, help link, "Reorder" button.

Responsive breakpoints: 375 (base), 640, 768, 1024, 1280. The kiosk stage (1280x800) uses the desktop layout, with 44px+ targets throughout since it is touch-driven.

## 7. Accessibility notes

- Contrast: `--sc-ink` #252925 on white is about 14:1. `--sc-green-600` #2E7D46 with white text is about 5.2:1 (AA for normal text). `--sc-mut` #6B7C73 on white is about 4.2:1, so use it only for 14px+ or non-essential text and prefer `--sc-ink-2` for anything essential. Lime on ink (ETA pill) is about 11:1. Never use lime as text on white.
- Tap targets: 44x44px minimum (hit area may exceed the visual size, e.g. the 32px stepper buttons).
- Focus: always visible, 2px outline `--sc-focus` with offset, never removed. Drawer and modal trap focus and return it on close. Esc closes.
- Semantics: header, nav, main landmarks. Product cards are links with accessible names that include price. The stepper exposes a labelled group and a live quantity. Toasts use `role="status"`, errors use `role="alert"`.
- Do not rely on color alone (discount, out of stock and errors also carry text or icons).
- Motion: honor `prefers-reduced-motion`. Carousels pause on hover and focus and offer controls.
- Text zoom to 200% without loss of function. Prices stay on one line.

## 8. Do and don't

Do
- Keep a single filled green CTA style, and keep ADD and the stepper the same size so nothing jumps.
- Show ETA, price, MRP and savings on every card.
- Reserve space for images and use skeletons to prevent layout shift.
- Explain every fee in the bill with an info affordance.
- Keep copy short, plain and original.
- Test at 375px and at the 1280x800 kiosk stage.

Don't
- Don't copy logos, wordmarks, names, banners, product imagery, proprietary fonts (Okra, Zepto Norms) or promotional copy from either site.
- Don't use pink-red or Blinkit's yellow as a brand colour. SmartCart stays green, lime and cream.
- Don't show a discount badge without a real MRP.
- Don't put more than one primary button in a view.
- Don't hide the cart or bill total behind extra taps.
- Don't autoplay carousels without pause controls, and don't use text below 11px.

## 9. Unverified items

- Blinkit product card, ADD button colors, stepper, cart drawer, listing page and checkout were not observed because the location modal gated content and no location was entered. Cells marked (u) are unverified.
- No checkout, order tracking, skeleton or toast behaviour was observed on either site (requires login or an order).
- Mobile (375px) views were not captured. Mobile behaviour here is based on common quick-commerce patterns plus the Zepto desktop layout.
- Hex values for Blinkit yellow (#F8CB46) were not confirmed, and are intentionally not used in SmartCart tokens.

## 10. Mapping to the SmartCart kiosk

SmartCart is an in-store cart, not a delivery app. Use the patterns below in the kiosk, and skip the delivery-only parts unless a delivery mode is ever added.

| Quick-commerce pattern | Kiosk equivalent | Backed by |
|---|---|---|
| ProductCard + ADD/stepper | Cart line items; the stepper drives qty | `POST /api/cart/<id>/inc` · `dec` · `delete` |
| "Item added" micro-feedback | Toast + stepper scale-in when YOLO adds an item | `POST /api/cart/recognized` (via `/api/state` poll) |
| CartBar / CartDrawer | Always-visible cart panel on the 1280x800 stage | `GET /api/state` |
| BillDetails + savings strip | Checkout ledger: subtotal, eco fee, tax, total | `totals()` |
| CategoryTile / search | "Shop by aisle" tiles that open the store map | client-side `CATS`, `ZN` |
| ETA pill | Replace with aisle/location pill (e.g. "Aisle 04") | client-side |
| LocationPicker, address step, OrderTracking | Not applicable to the kiosk | — |
| Checklist (new) | Shopping list with progress, ticked by recognition | `/api/list/*` |
| Alert states | Scale anomaly screen uses `--sc-red-600` and `--sc-amber-400` | `POST /api/anomaly` |
