# Salus Market — Prototype Build Plan

## Purpose of This Document

This is an execution-ready build plan for a coding agent to produce two standalone HTML prototypes that will be demoed to the Salus Market owner **on an actual phone**. These prototypes replace the original `salus_market_prototype.html` (customer-only, no membership, no staff tool) and are meant to demonstrate the full current vision — including membership mechanics — across both the customer-facing app and the internal staff tool.

This doc assumes the agent has **not** read the four planning docs in full and may have limited context budget. Everything needed to build is restated here. The planning docs (`sm-customer.md`, `sm-membership.md`, `sm-staff.md`, `sm-owner-decisions.md`) are the deeper source of truth if a question comes up that this doc doesn't answer — read them if something is ambiguous, but this doc should cover the build itself end to end.

**This is a prototype, not production code.** Single HTML file per app, inline `<style>` and `<script>`, no build step, no real backend, no real Square integration. The goal is a convincing, interactive, phone-viewable demo — not the production architecture (which is separately specced in the planning docs as React + Vite + Supabase + Square SDK, for later).

-----

## Deliverables

Two files, each a complete single-file HTML document (style + script inline, matching the format of the original `salus_market_prototype.html`):

1. **`salus-customer-prototype.html`** — the customer ordering app
2. **`salus-staff-prototype.html`** — the internal staff/admin tool

Build them **one at a time, customer app first**, fully working and visually polished before starting the second. Do not attempt both simultaneously.

-----

## Shared Design System (Both Apps)

Reuse exactly — these are already finalized and should not be reinterpreted:

```css
--charcoal:   #1e2019   /* Primary background */
--terracotta: #c1604a   /* Primary action color — buttons, headers, badges */
--cream:      #f5f0e8   /* Secondary accent — labels, prices, highlights */
--warm-white: #faf7f2   /* Light surface, used sparingly */
--text-dark:  #1a1a18
--text-mid:   #4a4a42
--text-light: #8a8a80
```

- **Fonts (Google Fonts):** Bebas Neue (display/headers), Cormorant Garamond italic (subtext/taglines), DM Sans (body/UI)
- **Outdoor readability principle:** dark backgrounds, high contrast (70%+ opacity secondary text, 88%+ for important text), generous font sizes (15px+ interactive, 13px+ secondary), 44px minimum touch targets
- **Layout:** max-width 430px (phone-locked), 18px horizontal screen padding, 8–10px card margins, 10–12px border radius on cards, 50% on circular elements
- **Reference the existing prototype** (`/mnt/project/salus_market_prototype.html` if available, or its description below) for exact visual fidelity on the original 5 customer screens — same screen flow, same component look, same interaction patterns (tap-cell-to-add, badge counters, etc.). The original prototype's CSS/JS structure for Home/Menu/Cart/Payment/Confirm should be the literal starting point, extended rather than redesigned.

-----

## App 1: Customer Ordering Prototype

### Screen Flow

```
Home → Menu → Cart → Payment (mock) → Confirmation
         ↑_______↓
Plus: Events (new) — reachable from Home, not part of the linear ordering funnel
```

### Demo Controls (Important — Not in Production Spec)

Since this is a demo, not production, add a **small, unobtrusive demo control panel** (e.g. a collapsible drawer or a few toggle buttons tucked in a corner, doesn't need to be pretty, just functional) that lets the person showing the demo flip these states live:

- **Member status toggle:** Guest / Member / Founding Member (three states, not two)
- **Open/closed toggle:** force the app into "open" or "closed" state regardless of real time, to demo the closed-checkout block without waiting for actual off-hours
- **Service period toggle:** Day / Evening (to demo the evening-menu placeholder state — see below)

These toggles exist purely so the owner can be shown every state in one sitting without waiting for real clock time or real membership data. Make them visually distinct from the real app UI (e.g. a thin bar pinned to the very top or bottom, gray/neutral, clearly "control panel" not "app").

### Screen 1 — Home

Same as original prototype:
- Hero photo area (use a placeholder/gradient if no real photo asset is available — note where `hero-golden.jpg` would go)
- "SALUS MARKET" wordmark in Bebas Neue
- "Neighborhood Café · Olde Towne Clinton" italic subtext
- "Order Now" CTA (terracotta, full width)
- Directions card (can use a static/mock distance — real GPS not required for this prototype, but keep the visual treatment)
- Live open/closed status — driven by the demo toggle, not real time, for this prototype

**New for this build:**
- A small **"Events"** entry point (e.g. a secondary button or card below the main CTA) leading to the new Events screen
- A **member status indicator** if Member or Founding Member is toggled on — small badge or line of text, e.g. "Welcome back, Member" / "Founding Member No. 7" — tasteful, not loud (per the membership doc's "belonging, not status" principle — this should feel warm, not like a special-tier badge flex)

### Screen 2 — Menu

Same as original prototype: scrollable list, tap-cell-to-add, badge + minus button, section labels (Coffee & Espresso, Cold Drinks, Breakfast, Lunch, Market Goods).

**New for this build:**
- Respect the **service period toggle**. When toggled to "Evening," show a distinct, shorter menu (drinks-focused — invent 3-4 placeholder evening drink items, clearly labeled as placeholder, e.g. "Evening Menu — items TBD with owner") instead of the day menu. This demonstrates the two-window architecture even though real evening hours/menu don't exist yet.
- ETA bar logic unchanged from original (8 min prep time messaging).

### Screen 3 — Cart

Same as original prototype: Carry-Out/Dine-In toggle, line items with qty controls, subtotal/tax/total, "Pay with Square" button, "Secured by Square" badge.

**New for this build:**
- If **Member or Founding Member** is toggled on, automatically add a non-removable line item: **"Free Drip Coffee — Member Perk"** at $0.00, visually distinct (e.g. cream-colored text, small "MEMBER PERK" tag) so it's obviously the perk being demonstrated, not a regular item.
- If **Founding Member** specifically is toggled on, additionally show the membership price context isn't relevant here (that's subscription billing, not a food order) — skip that, the perk line item is the only founding-member-relevant thing on this screen.
- If the **open/closed toggle** is set to "closed," disable the "Pay with Square" button and replace it with a closed message (e.g. "Salus Market is closed right now — come back during business hours!" plus a faux hours blurb). Checkout must not proceed to the Payment screen in this state — this is the confirmed closed-restaurant behavior.

### Screen 4 — Payment (Mock)

Same as original prototype: mock Square-branded checkout screen, terracotta header, dark body, Square merchant block. No real payment processing — a "Complete Payment" button just advances to Confirmation.

### Screen 5 — Confirmation

Same as original prototype: "ORDER PLACED" header, order number, three-block timing card (Prepare / Walk Away / Until Ready), dynamic advice text, status steps (received → preparing → ready), directions card.

**New for this build:**
- If a member perk was applied in Cart, a small "Thanks for being a member 🌞" type line — warm, not flashy.

### Screen 6 — Events (New)

This is the net-new screen. Build it simply:

- Terracotta header: "EVENTS" or "COMMUNITY"
- List of 2-3 placeholder events (invent reasonable content — e.g. "Community Coffee Tasting," a date/time, short description)
- Each event card shows an **RSVP button** and a **window indicator**:
  - If Member/Founding Member is toggled: "Members can RSVP now" (button active)
  - If Guest is toggled and the event is still in its member-only window: "Opens to everyone in 2 days" (button disabled/grayed, shown as locked) — this demonstrates the "first crack" mechanic from the membership doc
  - If Guest is toggled and the public window has opened: normal active RSVP button
- No capacity/seats-remaining UI — events are uncapped by design decision, don't show a capacity number anywhere
- Tapping RSVP can just show a confirmation state (toast, checkmark, "You're in!") — no real backend needed

-----

## App 2: Staff Tool Prototype

### Screen Flow

This is not a linear funnel — it's a single-page tool with view-switching, not screen-to-screen navigation like the customer app.

```
[Role Toggle: Staff / Admin]
    ↓
Order Queue (kanban board) ←→ Member Lookup ←→ Event Management (admin only)
```

### Demo Controls

- **Role toggle:** Staff / Admin — prominent, top of screen, since this is the core thing being demonstrated (role-gated views). When switched, the available tabs/sections should visibly change (Event Management and editable Member records disappear in Staff view).
- A way to **seed/reset demo data** is helpful but not required — e.g. a "Reset Demo" button that reloads a few sample orders into the queue, since the owner will likely want to see this multiple times in one sitting.

### Section 1 — Order Queue (Kanban Board)

This is the centerpiece. Build it as the primary view.

- **Four columns, side by side** (horizontal scroll on phone width is fine, or stack if that reads better on a real phone — test both, pick whichever demos more clearly on an actual screen): **Received → Preparing → Ready for Pickup → Completed**
- Within each column, **orders sorted oldest-first** (top = longest waiting)
- **Seed 5-6 placeholder orders** distributed across the first three columns (not all in Received) so the board doesn't look empty — invent reasonable order numbers, items, carry-out/dine-in, and timestamps
- **At least one order should belong to a "member"** — show a member badge/tag on that card (e.g. small cream "MEMBER" or "FOUNDING MEMBER" tag) so staff-visibility-of-membership is demonstrable
- **Each card shows:** order number, items (brief), carry-out vs. dine-in, time placed, member status if applicable
- **Each card has explicit buttons:** **← Back** and **Advance →**
  - Tapping Advance → moves the card to the next column
  - Tapping ← Back moves the card to the previous column
  - **On a Received card: no ← Back button at all** (hidden, not grayed)
  - **On a Completed card: no Advance → button at all** (hidden, not grayed) — Completed is terminal, ← Back should still work to correct a mistake
- **Urgency cue:** any card sitting in Received or Preparing for longer than **8 minutes** (simulate this with a fast demo clock if needed — e.g. treat 8 *seconds* as the threshold for demo purposes so the owner can actually see it trigger live, and add a small code comment noting this is sped up for demo purposes only) should visually shift (e.g. card border or background shifts toward a warning color) to demonstrate the urgency mechanic.

### Section 2 — Member Lookup

- Simple searchable/scrollable list: name, subscription status (active/paused/canceled), founding member flag, join date
- Seed 5-6 placeholder members, **at least one marked Founding Member** with a number (e.g. "Founding Member No. 7")
- **Staff view:** read-only — no edit controls visible at all
- **Admin view:** same list, but with edit affordances visible (e.g. a pencil icon or "Edit" button per row) — doesn't need to actually persist edits for the prototype, just needs to visually demonstrate the role difference. Tapping edit can open a simple inline form or modal; saving can just close it without real persistence.

### Section 3 — Event Management (Admin Only)

- **Entirely hidden in Staff view** — this is the clearest demonstration of role-gating, so make sure it's actually absent (not grayed out) when Staff role is active.
- **Admin view:** a simple form to create an event (title, datetime, description) and a list of existing events with their member-RSVP-window and public-RSVP-window settings, plus a basic "who's RSVP'd" list per event. Mirror whatever placeholder events were used in the customer app's Events screen, so the demo feels connected (e.g. the same "Community Coffee Tasting" event appears in both prototypes).
- No capacity field anywhere — events are uncapped by design decision.

-----

## Explicitly Out of Scope for This Build

Do not attempt any of the following — they are real, deferred, and would distract from the core demo:

- Real backend/database (no Supabase, no persistence across page reloads — in-memory JS state is fine, resetting on refresh is acceptable)
- Real Square SDK integration (Payment screen stays mock, as in the original prototype)
- Real GPS/geolocation (use static/mock distance values, keep the visual treatment)
- Real authentication/login (the role toggle is a demo convenience, not real auth)
- Push notifications
- Real evening menu items, real evening hours, real founding-member discount math, real subscription pricing — all of these can use clearly-labeled placeholder values
- Multi-location anything

-----

## Source Documents (For Deeper Context If Needed)

If anything in this plan is ambiguous, these docs hold the full reasoning behind every decision referenced above:

- `sm-customer.md` — full customer app spec, screen-by-screen, including the Evening Service architecture and closed-checkout behavior
- `sm-membership.md` — full membership data model, Perk/Tier system, Founding Member mechanics, event/RSVP design
- `sm-staff.md` — full staff tool spec, including the resolved kanban/status/role decisions and Backend & Data Architecture section
- `sm-owner-decisions.md` — consolidated list of what's still genuinely unconfirmed by the owner (useful for knowing what to clearly mark as placeholder vs. what's actually finalized)

-----

## Build Order Recap

1. Build `salus-customer-prototype.html` completely (all 6 screens + demo controls), verify it works end-to-end including all three demo-toggle states, before starting the second file.
2. Build `salus-staff-prototype.html` completely (kanban + member lookup + event management + role toggle), verify both Staff and Admin views work and visibly differ.
3. Both files should be self-contained — no shared dependencies between them, no external assets that aren't either inlined or gracefully degraded (e.g. placeholder colored divs instead of real photos if assets aren't available).
4. Test both on an actual phone-width viewport before considering done — this is the explicit demo requirement from the owner ask.
