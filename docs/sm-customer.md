# Salus Market — Mobile Ordering App

## Project Overview

**Salus Market** is an upcoming coffee, breakfast, and lunch restaurant located at **100 E. Leake St., Olde Towne Clinton, Mississippi**. It sits in the heart of downtown Clinton near Lions Club Park, surrounded by heavy foot and car traffic, families, and pedestrians.

The name “Salus” is a direct nod to Clinton’s original name — *Mount Salus*, meaning “mountain of health” — reflecting the owner’s deep roots in the community.

This app is a **branded mobile ordering PWA** built for Salus Market. It is not a generic SaaS product — it is purpose-built to feel like *Salus Market*, distinct from Square or any other off-the-shelf tool.

**Revision note:** This version resolves the closed-restaurant checkout behavior (see Cart screen) and notes two cross-doc dependencies that emerged from later planning: the `PREP_MINUTES` constant is now shared with the staff tool's urgency-cue logic (`sm-staff.md`), and a possible evening-hours business concept was raised during membership planning (`sm-membership.md`) that would require extending the hours logic below.

### The Core Problem It Solves

People walking through Olde Towne Clinton — families, kids, foot traffic from the park — should be able to pull out their phone, order ahead, know exactly when their food will be ready relative to how far away they are, and walk in right as it’s done. The entire UX is built around that single use case.

**Payment processing** is handled by Square (existing third-party POS). This app is the *experience layer* on top of Square — the branded journey from discovery to pickup.

-----

## Developer Context

This project is being built by **Hunter**, a 15-year professional software developer who is transitioning into independent/freelance development. This is one of his first independent projects. The prototype was built collaboratively with Claude before being ported to Claude Code for production development.

-----

## Tech Stack

|Layer         |Choice                                            |Rationale                                                 |
|--------------|--------------------------------------------------|----------------------------------------------------------|
|Framework     |**React 18 + Vite**                               |Component-based, fast HMR, great PWA support              |
|Styling       |**CSS Modules** or **Tailwind CSS**               |Scoped styles, design tokens map cleanly                  |
|State         |**React Context + useReducer**                    |Cart state, location state — no Redux needed at this scale|
|Payments      |**Square Web Payments SDK**                       |Owner already using Square for POS                        |
|Maps/GPS      |**Browser Geolocation API + Apple Maps deep link**|Native, no API key needed for the MVP                     |
|Deployment    |**Vite PWA plugin**                               |Installable, offline-capable, no app store required       |
|Source Control|**Git**                                           |Standard                                                  |

-----

## Design System

### Color Palette

```css
--charcoal:   #1e2019   /* Primary background — all screens */
--terracotta: #c1604a   /* Primary action color — buttons, headers, badges */
--cream:      #f5f0e8   /* Secondary accent — labels, prices, highlights */
--warm-white: #faf7f2   /* Light surface (used sparingly) */
--text-dark:  #1a1a18
--text-mid:   #4a4a42
--text-light: #8a8a80
```

**Color logic:**

- **Terracotta** = action, energy, calls to action (Order Now, Pay, headers, active states)
- **Cream** = data, prices, section labels, secondary text accents on dark backgrounds
- **Charcoal** = all screen backgrounds — chosen deliberately for outdoor readability in direct sunlight

### Typography

|Role             |Font                         |Usage                                                     |
|-----------------|-----------------------------|----------------------------------------------------------|
|Display / Headers|**Bebas Neue**               |Screen titles (YOUR ORDER, MENU, ORDER PLACED), brand name|
|Serif / Subtext  |**Cormorant Garamond** italic|Subtitles, taglines, order numbers                        |
|Body / UI        |**DM Sans**                  |All other UI text, buttons, labels                        |

All fonts loaded from Google Fonts.

### Outdoor Readability Principle

The app is designed to be used **outside in bright sunlight** while walking downtown. Every design decision should serve this:

- Dark backgrounds (charcoal) instead of light ones — light backgrounds wash out in sun
- High-contrast text — minimum 70% opacity for secondary text, 88%+ for anything important
- Font sizes are generous — 15px minimum for interactive/readable content, 13px for truly secondary text
- Touch targets are large — 44px minimum for anything tappable
- Terracotta headers create instant visual orientation even in glare

### Spacing & Layout

- Max width: **430px** (mobile-first, phone-width locked)
- Screen padding: **18px horizontal**
- Card margin: **8–10px horizontal**
- Border radius: **10–12px** for cards, **50%** for circular elements

-----

## Application Architecture

### Screen Flow

```
Home → Menu → Cart → Payment (Square) → Confirmation
         ↑_______↓ (back navigation)
```

There is **no persistent navigation bar**. The flow is intentionally linear — a funnel, not an app with tabs. Every screen has one job.

### Screens

#### 1. Home (`/`)

**Purpose:** Landing, brand impression, immediate CTA, directions.

**Key elements:**

- Hero photo (storefront, golden hour) with gradient fade into charcoal
- Salus Market icon (sun/circle mark) overlaid at bottom of hero
- “SALUS MARKET” in Bebas Neue
- “Neighborhood Café · Olde Towne Clinton” italic subtext
- **“Order Now”** CTA button (terracotta, full width)
- **Directions card** — shows real GPS distance and walk time, opens Apple Maps on tap (`maps://` deep link to `100 E Leake St, Clinton, MS`)
- Live open/closed status based on actual time of day

**Business hours logic:**

```
Mon–Fri: 7:00 AM – 3:00 PM
Saturday: 8:00 AM – 3:00 PM
Sunday: 9:00 AM – 2:00 PM
```

> ⚠️ **This is the daytime window only, and is itself still unconfirmed with the owner.** Evening beverage service is confirmed real (see "Evening Service" section below) and requires this logic to support a *second*, independent daily window — e.g. closed gap, then open again in the evening — not a single continuous block. Treat the schedule above as `day` period data only; don't build the hours check as a single open/closed boolean. See "Evening Service" section for the resolved approach.

**GPS behavior:** On tapping “Order Now”, the app requests `navigator.geolocation`. Distance is calculated via the Haversine formula. Walk time assumes ~80 meters/minute pace.

-----

#### 2. Menu (`/menu`)

**Purpose:** Browse and build order. Single scrollable list, no categories.

**Key elements:**

- Sticky terracotta header with back button, “MENU” title, cart pill (appears when items are added)
- Smart ETA bar — dynamically updates based on GPS distance:
  - No location: “Order now — ready in about 8 min”
  - Walk time ≤ prep time: “Order now — ready before you arrive”
  - Walk time > prep time: “Order now — ready in 8 min · leave in X min”
- Full menu as scrollable list, grouped by section labels
- **Tap anywhere on a cell** to add an item to the cart
- When an item is in the cart, a **count badge + minus button** appears on the cell
- No separate “add” button — the whole cell is the tap target

**Menu sections:** Coffee & Espresso, Cold Drinks, Breakfast, Lunch, Market Goods

**Prep time constant:** `PREP_MINUTES = 8` (hardcoded for MVP, should be configurable by owner)

-----

#### 3. Cart (`/cart`)

**Purpose:** Review order, select carry-out vs dine-in, proceed to payment.

**Key elements:**

- Terracotta header: “YOUR ORDER” + “Review before paying” (italic, left-aligned under title)
- Carry-Out / Dine-In toggle (gold border on active) — **both are confirmed in-scope** (owner decision)
- Line items with quantity +/− controls
- Subtotal, tax (8%), total
- **“Pay with Square”** button (terracotta)
- “Secured by Square” badge

**Open sub-question on Dine-In:** does selecting "Dine-In" need a table-number/identifier step so staff know where to deliver the order, or is it counter-pickup regardless of order type (customer just sits down with the food themselves)? Not yet decided — affects whether this screen needs an additional table-selection control. Build the toggle now; treat table selection as a possible addition once answered.

**Closed-restaurant behavior (decided):** the menu remains fully browsable at any time, open or closed — someone can plan an order ahead even when Salus Market isn't currently taking them. The block happens specifically **at the Cart screen**: if the restaurant is closed (per the existing open/closed hours logic), the **“Pay with Square” button is disabled and replaced with a closed message** (e.g. hours + reopening time). The flow never proceeds to the Payment screen, and no order or charge is ever created while closed. This keeps the block entirely client-side and keeps the staff tool's order queue unaffected — a blocked checkout never becomes an `Order` record.

-----

#### 4. Payment (`/payment`)

**Purpose:** Square-branded checkout handoff.

In the **prototype**, this is a mock Square checkout screen for demo purposes. In production, this screen either:

- **Redirects** to Square’s hosted checkout URL, or
- **Embeds** the Square Web Payments SDK inline card form

The mock screen demonstrates the intended UX: Square branding is present but the overall color scheme stays consistent with Salus Market (terracotta header, dark body).

**Square SDK integration notes:**

- Use `@square/web-sdk` npm package
- Requires a Square Developer account and Application ID
- Sandbox available for testing
- On successful payment, Square returns an order/payment ID — use this as the order number on the confirmation screen

-----

#### 5. Confirmation (`/confirm`)

**Purpose:** Order placed — show timing, status, directions.

**Key elements:**

- “ORDER PLACED” header with check circle
- Order number (e.g., `#SM-1042`) and order type
- **Smart timing card** (three blocks):
  - **Min to Prepare** — fixed prep time (8 min)
  - **Min Walk Away** — from GPS, or `—` if unavailable
  - **Min Until Ready** — `max(0, walkMins - prepMins)`
- Dynamic advice text:
  - Walk ≤ prep: “Your order will be ready before you arrive. Head over now!”
  - Leave time ≤ 2 min: “Start walking now — your order will be ready right as you arrive.”
  - Leave time > 2 min: “Relax for X more minutes, then head over.”
- Order status steps (received → preparing → ready for pickup)
- Apple Maps directions card

-----

## Component Structure (Recommended)

```
src/
├── main.jsx
├── App.jsx                  # Router, screen state
├── context/
│   ├── CartContext.jsx       # Cart state + actions
│   └── LocationContext.jsx   # GPS, distance, walk time
├── screens/
│   ├── Home.jsx
│   ├── Menu.jsx
│   ├── Cart.jsx
│   ├── Payment.jsx
│   └── Confirm.jsx
├── components/
│   ├── MenuItem.jsx          # Single menu row with badge logic
│   ├── CartRow.jsx           # Single cart line item
│   ├── EtaBar.jsx            # Smart ETA message bar
│   ├── DirectionsCard.jsx    # GPS distance + Apple Maps link
│   ├── TimingCard.jsx        # Three-block timing display on confirm
│   ├── StatusSteps.jsx       # Order progress tracker
│   └── BackButton.jsx
├── utils/
│   ├── haversine.js          # Distance calculation
│   ├── hours.js              # Open/closed logic
│   └── squareClient.js       # Square SDK wrapper
├── assets/
│   ├── logo-icon.png         # Sun/circle mark (the standalone icon)
│   ├── logo-full.jpg         # Full SALUS MARKET wordmark logo
│   ├── hero-golden.jpg       # Storefront golden hour photo
│   └── hero-night.jpg        # Storefront night photo (used in payment)
└── styles/
    ├── tokens.css            # CSS custom properties (design tokens)
    └── global.css            # Reset, body, screen base styles
```

-----

## Key Constants

```js
// Location
const SALUS_LAT = 32.3414
const SALUS_LNG = -90.3216
const SALUS_ADDRESS = '100 E Leake St, Clinton, MS 39056'
const APPLE_MAPS_URL = 'maps://maps.apple.com/?daddr=100+E+Leake+St,+Clinton,+MS+39056&dirflg=w'

// Ordering
const PREP_MINUTES = 8         // Average order prep time
const WALK_SPEED_MPS = 80      // Meters per minute (average walking pace)
const TAX_RATE = 0.08          // 8% Mississippi sales tax
```

-----

## Menu Data

The menu should be data-driven (JSON or a CMS in the future). For MVP, a static data file is fine.

```js
// Suggested structure
{
  id: 'cortado',
  name: 'Cortado',
  description: 'Double shot, steamed whole milk, La Marzocco',
  price: 4.50,
  category: 'coffee',        // coffee | drinks | breakfast | lunch | market
  icon: 'coffee',             // maps to SVG icon key
  available_during: 'day'    // 'day' | 'evening' | 'both' — see Evening Service section
}
```

**Current menu items (to be confirmed with owner):**

|Item              |Price |Category |
|------------------|------|---------|
|Cortado           |$4.50 |coffee   |
|Salus Latte       |$5.50 |coffee   |
|Cold Brew         |$5.00 |coffee   |
|Cold Press Juice  |$6.50 |drinks   |
|Sparkling Lemonade|$4.50 |drinks   |
|Avocado Toast     |$9.00 |breakfast|
|Granola Bowl      |$7.00 |breakfast|
|Breakfast Sandwich|$10.00|breakfast|
|Market Salad      |$11.00|lunch    |
|Turkey Club       |$12.00|lunch    |
|Granola Bag       |$8.00 |market   |
|Bottled Cold Press|$7.00 |market   |


> ⚠️ **These are placeholder items.** Actual menu must be confirmed with the owner before launch.

-----

## What’s Real vs. Mock in the Prototype

|Feature                             |Prototype Status            |Production Path                  |
|------------------------------------|----------------------------|---------------------------------|
|GPS distance/walk time              |✅ Real (browser geolocation)|Same — keep as-is                |
|Open/closed hours                   |✅ Real (time-based logic)   |Same — make hours configurable   |
|Menu browsing                       |✅ Real (static data)        |Move to JSON / CMS               |
|Cart management                     |✅ Real (JS state)           |React Context                    |
|Square payment                      |❌ Mock UI only              |Integrate Square Web Payments SDK|
|Order number                        |❌ Random client-side        |Must come from Square order ID   |
|Order status / queue                |❌ Static mock               |Requires backend (see below)     |
|Push notification (ready for pickup)|❌ Not implemented           |Requires backend + web push      |

-----

## Backend Architecture (Resolved)

The MVP can ship without the customer app talking to a backend directly — Square handles payment, and the confirmation screen is optimistic (it assumes the order is being made). But the backend itself is no longer optional/future at the project level — it's required from the start for the staff tool (`sm-staff.md`) and membership (`sm-membership.md`), both of which depend on it for order visibility, auth, and member data.

**Confirmed architecture (see `sm-staff.md` Backend & Data Architecture section for full detail):**

- **Platform: Supabase** (Postgres + realtime + auth) — confirmed over Firebase based on the relational shape of the data (members, orders, perks, events all reference each other).
- **Order ingestion: Square webhook only.** This customer app does **not** write orders directly to Supabase — Square notifies a Supabase Edge Function on payment success, which creates the `Order` row. This was a deliberate simplification: a direct-write path from this app was considered and rejected to avoid duplicate-order handling, and because the staff tool needs to function during a bridge period before this app is even live (Square-native checkout in the interim).
- This app's responsibility regarding the backend is narrow: send the customer through Square payment, and optionally **read** order status back (for the Confirmation screen to stop being purely optimistic) — never write order data directly.

Once this is live, the natural next additions are:

- **Web Push notifications** — in-app status only is confirmed (no SMS); push notification on "Ready for Pickup" is the future enhancement once order status is real
- **Menu management** — owner can update items/prices without a code deploy
- **Order history** — per-customer (requires auth)

-----

## Open Questions for Owner Meeting

- [ ] Confirm actual menu items, prices, and descriptions
- [ ] Confirm actual hours of operation
- [ ] What is the realistic average prep time? (Currently hardcoded at 8 min — note: this constant is now shared with the staff tool's urgency-cue logic in `sm-staff.md`, so changing it affects both apps)
- [x] ~~Does he want carry-out only, or dine-in too?~~ → **Both.** Open sub-question: does dine-in need a table number/identifier for staff delivery, or counter-pickup either way? Affects whether the Cart screen's order-type toggle needs a table-selection step.
- [ ] Square account details — is he on Square for Restaurants or Square POS?
- [x] ~~Does he want SMS notification when order is ready, or just in-app status?~~ → **In-app status only.** No SMS provider/integration needed — web push (already planned post-MVP) is the only notification channel.
- [x] ~~What should happen if the restaurant is closed and someone opens the app?~~ → Decided: menu browsable anytime; checkout blocked at Cart screen with a closed message, never reaches Payment. See Cart screen spec above.
- [ ] Domain / app name preferences for deployment
- [x] ~~Is evening beverage service a real, planned business concept?~~ → **Confirmed real**, exact hours/scope still pending. See new "Evening Service" section below.

-----

## Evening Service — Designed For, Not Yet Built

Confirmed by the owner: evening beverage service is a real, planned concept — separate from daytime café/lunch service. Two key shape decisions are already known even though exact hours/menu aren't:

- **Separate, lighter, drinks-focused menu** — not the same menu just running later.
- **Distinct second service period**, not a continuation of daytime hours — likely a closed gap between day service ending and evening service beginning (e.g., closed 3–6pm, then open again), possibly with a different staffing/vibe.

**This changes the hours/menu architecture, even before exact evening hours are known:**

- `hours.js` needs to support **two independent daily windows** rather than a single open/closed block. The function should return which service period (if any) is currently active — `'day' | 'evening' | 'closed'` — not just a boolean.
- Menu items need a `available_during: 'day' | 'evening' | 'both'` field added to the data structure (see Menu Data below), so the Menu screen renders the correct item set for whichever period is active.
- The membership Perk model (`sm-membership.md`) already anticipated this — the "free evening drink" perk has an `applies_to: 'evening_hours'` condition that's been inert until now. No rework needed there.

**Status:** build the flexible two-window structure now, with placeholder evening hours and a placeholder evening menu (clearly marked as such) — this is "designed-for-but-deferred," not "wait until hours are confirmed to start."

-----

## PWA Configuration

The app should be installable on iOS/Android via the browser “Add to Home Screen” prompt.

```js
// vite.config.js — add VitePWA plugin
import { VitePWA } from 'vite-plugin-pwa'

VitePWA({
  registerType: 'autoUpdate',
  manifest: {
    name: 'Salus Market',
    short_name: 'Salus',
    theme_color: '#c1604a',       // terracotta
    background_color: '#1e2019',  // charcoal
    display: 'standalone',
    orientation: 'portrait',
    icons: [/* 192px and 512px versions of the sun/circle icon */]
  }
})
```

-----

## Assets

All brand assets should be placed in `src/assets/`. The following are available:

- `logo-icon.png` — the standalone sun/circle gold mark on cream background (used as home screen icon, circular crop)
- `logo-full.jpg` — the full SALUS MARKET wordmark (dark background, sage lettering, gold icon)
- `hero-golden.jpg` — storefront exterior, warm golden hour tones (hero photo on home screen)
- `hero-night.jpg` — storefront exterior, night/dusk (used in Square payment merchant block)
- Team photo, patio photo — available for future About/Info screen

-----

## Prototype Reference

A complete single-file HTML prototype (`salus_market_prototype.html`) was built prior to this production version. It contains the full working UI across all 5 screens with all styling, logic, and embedded assets. Use it as the visual reference for component implementation — it is the source of truth for design intent.

The prototype can be opened directly in any browser and represents the target fidelity for the React build.