# Salus Market — Staff Internal Tool (Concept Doc)

## Status

This is a **planning document**, progressively moving toward a build spec. It's the third leg of the Salus Market software vault, alongside `sm-customer.md` (customer-facing ordering app) and `sm-membership.md` (membership program concept). This doc covers the internal tool staff and the owner use to fulfill orders and manage membership.

**Revision note:** This version resolves the order-fulfillment open questions from the original draft (queue layout, status advancement mechanic, urgency thresholds, role enforcement timing). One consequence worth flagging up front: **role enforcement is now a Phase 1 requirement, not deferred** — see "Two Halves, One Tool" below. This is a real scope change from the original plan, which assumed "everyone sees everything" was fine early on.

-----

## Core Premise

A single internal web app, used by whoever's working — front counter, kitchen, or the owner — to:

1. **See and fulfill incoming orders** placed through the customer app
2. **Manage the membership program** — see who's a member, honor perks, create/manage community events

**Payment is explicitly out of scope here.** Square handles all money — the customer pays through Square at the end of the ordering flow (per `sm-customer.md`), and that transaction is complete before this tool ever sees the order. This tool deals with *fulfillment and relationship management*, never charges or refunds.

-----

## Two Halves, One Tool

Rather than building "staff app" and "admin app" as separate products, this is **one tool with role-aware views.** Everyone logs into the same system; what they see and can do is gated by role.

**Decided — roles are enforced from day one**, not deferred. This is a change from the original plan (which left "everyone sees everything" as acceptable for v1). The split:

| Capability | Staff | Admin |
|---|---|---|
| Order queue — view and advance/back status | ✅ | ✅ |
| Member lookup — view status, founding flag, join date | ✅ (read-only) | ✅ |
| Create/edit events, set RSVP windows | ❌ | ✅ |
| Edit `Perk` / `MembershipTier` config (the standing perks, pricing) | ❌ | ✅ |
| Edit member records (e.g. manually pause/cancel a subscription) | ❌ | ✅ |
| Menu management (future) | ❌ | ✅ |

The reasoning: staff need enough membership visibility to do their actual job — honoring a Pillar 2 perk at the counter — but nothing that lets them change pricing, perks, or event structure. That's an admin-level action.

**Consequence:** because roles are enforced (not just modeled) from day one, **auth/login must also exist from day one.** There's no way to gate by role without first knowing who's logged in. This pulls authentication into Phase 1 scope — see Phasing below.

```
Role
  - staff       → order queue (full read/write on status), member lookup (read-only)
  - admin/owner → staff capabilities + member record edits,
                  event creation, perk/tier config, menu management (future)
```

-----

## Half 1 — Order Fulfillment

### The Problem It Solves

Right now, the customer app's "Confirmation" screen (per `sm-customer.md`) is **optimistic** — it assumes the order is being made, with no real feedback loop from the kitchen. This tool closes that loop: it's where an order placed in the customer app actually becomes visible to a human who needs to make it.

### How Orders Arrive

**Confirmed context:** there is no existing informal/paper process this tool replaces — the owner will use **Square's own native order management as a bridge** until this custom staff tool is ready. This is effectively "Phase 0," sitting in front of everything below.

**Confirmed sequencing:** the staff tool does **not** need to launch together with the custom customer app — it can come online first, while customers are still ordering through Square-native checkout.

**Resolved: Square webhook only, permanently.** Square notifies a Supabase Edge Function the moment payment completes; that function writes the `Order` row. A direct-write path (the customer app writing orders to Supabase itself) was considered and explicitly rejected — see the Backend & Data Architecture section below for the full reasoning (it requires deduplication logic that isn't worth the complexity, and webhook-only cleanly covers the bridge period with no special-casing regardless of whether the payment came from Square-native checkout or the future custom app).

This confirms the need for the shared backend already flagged as "post-MVP" in `sm-customer.md` — this tool is effectively what makes that backend necessary, not optional.

### Order Queue View — Decided

**Layout: kanban board, one column per status**, not a flat sorted list. This was a deliberate choice over a single newest-first or oldest-first list: it mirrors how a real kitchen line actually works (nothing in an earlier status gets buried under newer orders), and each column doubles as an at-a-glance count of how many orders are sitting in that state right now.

- Columns: **Received → Preparing → Ready for Pickup → Completed**
- Within each column, orders are sorted **oldest-first** (top of column = longest-waiting in that status)
- Completed likely belongs as either a 4th visible column or a filtered/archived view rather than cluttering the live board — worth a quick implementation call, but functionally it's the terminal state either way

Each order card shows: order number, items, carry-out vs. dine-in, time placed, **member status** (so staff know to honor a Pillar 2 perk if it's not yet automated).

### Status Advancement — Decided

Each card has explicit, always-visible controls:

- **← Back** and **Advance →** buttons, both present whenever applicable
- Tapping **Advance →** moves the card one status forward (e.g. Received → Preparing)
- Tapping **← Back** moves the card one status backward (corrects a mis-tap or premature status change)
- **Boundary behavior:** buttons are **hidden, not disabled,** at the ends of the lifecycle — no ← Back button on a Received card (nothing precedes it), no Advance → button on a Completed card (nothing follows it). Completed is a terminal state; there is currently no path to un-complete an order.

This status change is what the customer's Confirmation screen should eventually reflect (closing the loop in the other direction) — see Notifications below.

### Urgency Cue — Decided

**Single global threshold**, not tuned per-status (deliberately deferred — tune later once real shift data exists rather than guessing now). The threshold is **tied to the existing `PREP_MINUTES` constant from `sm-customer.md` (currently 8 minutes)** rather than an independent number.

Rationale: this is one shared constant rather than two numbers that can drift out of sync. If the owner changes average prep time, both the customer-facing ETA messaging *and* the staff urgency cue move together automatically.

**Behavior:** any order sitting in a non-terminal status (Received or Preparing) for longer than `PREP_MINUTES` triggers a visual urgency cue (e.g. color shift on the card). Exact visual treatment is an implementation detail for the build pass.

### Notifications (Future)

Once order status is real (not optimistic), the natural next step is notifying the *customer* when their order moves to "Ready for Pickup" — web push, as already noted in `sm-customer.md`. Not in scope for this tool's first version, but the status field this tool manages is the trigger for it.

-----

## Half 2 — Membership Administration

This is the operational side of everything sketched in `sm-membership.md`.

### Member Lookup

- Search/list of current members — name, subscription status, founding member flag, join date
- **Visible to both roles** (staff: read-only; admin: editable) — this is what lets staff honor the Pillar 2 perk at the point of an order, especially in early phases before it's automatically applied at checkout
- In Phase 1, this might literally just be this list — membership starts here, before it's wired into the ordering flow at all

### Event Management

- **Admin-only.** Create an event (title, datetime, description). Capacity is not a feature — events are uncapped by decision (see `sm-membership.md`), so there's no capacity field to manage in the UI.
- Set the member RSVP window vs. public RSVP window (the "first crack" mechanic from `sm-membership.md`)
- See who's RSVP'd
- This is the admin-side tool that makes Pillar 1 ("first notification," "first crack at events") actually operable — without this, Pillar 1 is just a promise with no way to deliver on it

### Perk & Tier Configuration — New

- **Admin-only.** Per the resolved Perk Model in `sm-membership.md`, the standing perks (currently: free drip coffee, free evening drink pending evening hours) are editable data, not hardcoded logic.
- Admin can edit the active `Perk` rows and what's attached to the (currently single) `MembershipTier`.
- This is what makes "perks aren't fully decided yet" survivable in production — the owner can change the standing perk without Hunter touching code, starting in Phase 2.

### What's Deliberately Not Here

- **Billing management** — subscription billing (Stripe or Square invoicing, still an open question — see Backend & Data Architecture) lives with whatever payment provider is chosen, not rebuilt inside this tool. This tool reads membership *status*, it doesn't process membership *payments*.
- **Menu management** — mentioned as a future need in `sm-customer.md` but not core to this doc's first pass. Worth a future expansion once the menu is confirmed and stable.

-----

## Multi-Location Scaffolding

Salus Market is single-location today but has multi-location ambitions. The right move now is **silent scaffolding, not visible complexity**:

- `Order` and `Member` records carry a `location_id` field from day one, hardcoded to the single Clinton location
- No location picker, no location-aware UI, no staff-facing concept of "which location am I in" — none of that exists yet
- This means: if/when a second location opens, the data model already supports filtering by location. It becomes a UI and query problem (add a picker, scope the queue), not a migration problem (re-architecting every table to add a field that should've been there from the start)

This is the only multi-location consideration that belongs in this doc. Everything else (separate menus per location, location-specific events, cross-location membership) is genuinely a future problem and shouldn't shape decisions now.

-----

## Backend & Data Architecture (Resolved)

Decisions made across this and the companion docs, consolidated here since this tool is what makes the shared backend necessary in the first place.

**Platform: Supabase, confirmed.** Considered against Firebase — Supabase won on data-model fit. Everything sketched across `sm-customer.md`, `sm-membership.md`, and this doc (`Member`, `Subscription`, `Order`, `Event`, `RSVP`, `Perk`, `MembershipTier`, `StaffUser`) is inherently relational — members have orders, orders reference perks, events have RSVPs. Postgres (what Supabase is) models this natively with foreign keys; Firestore's NoSQL model would force denormalizing data that's naturally relational. Supabase also bundles auth and realtime subscriptions, both of which this tool needs (role-gated login, live-updating kanban board).

**Order ingestion: Square webhook only, permanently.** A direct-write path (the customer app itself writing orders to Supabase on payment success) was considered and explicitly rejected — not because it doesn't work, but because running both paths simultaneously requires deduplication logic (a unique constraint on Square's payment/order ID, since both paths could otherwise create a row for the same payment) that isn't worth the added complexity for the speed gained. Single path, single source of truth: Square notifies a Supabase Edge Function on payment completion, which writes the `Order` row. This also cleanly handles the confirmed bridge period (staff tool live before the custom customer app is) without any special-casing — webhook works identically whether the payment came from Square-native checkout or the future custom app's embedded Square payment screen.

**Subscription billing provider: pending a bounded sandbox spike.** See `sm-membership.md` for the specific test plan (confirm Square Subscriptions can model a non-physical membership without shipping/fulfillment fields, and confirm the founding-member percentage-discount mechanic is achievable). This is the one piece of the backend architecture still genuinely open — everything else here is decided.

-----

## Data Model (Resolved)

Builds directly on the `Order` and `Member`/`Subscription`/`Event`/`RSVP`/`Perk`/`MembershipTier` entities sketched in `sm-membership.md`. New/extended for this tool:

```
Order
  - id
  - location_id (hardcoded single value for now)
  - items[]
  - order_type (carry-out / dine-in)
  - status (received | preparing | ready | completed)
  - status_history[] // optional: for back-button audit trail
  - member_id (nullable)
  - member_perk_applied (boolean)
  - placed_at, status_updated_at

StaffUser
  - id
  - name
  - role (staff | admin) — enforced from day one
  - location_id (hardcoded single value for now)
  - auth credentials (login required from day one — see Phasing)

Event (from sm-membership.md, unchanged — capacity field present but always null)
  - + created_by (StaffUser.id)
```

-----

## Phasing (Proposed — Revised)

**Phase 1 — Order queue + auth, manual membership**
The kanban order queue (Received/Preparing/Ready/Completed), with Back/Advance controls and urgency cues, **plus working staff login and role enforcement** — this is pulled forward from the original plan because roles are no longer deferred. Membership stays manual/staff-memory or a simple shared list, per `sm-membership.md` Phase 1. Validates the fulfillment loop and access control together, without waiting on full membership infrastructure.

**Phase 2 — Membership administration added**
Member lookup (read-only for staff, editable for admin) and event management come online. Perk/Tier configuration becomes editable by admin. This is also when Supabase (confirmed platform — see Backend & Data Architecture) becomes the real shared source of truth for both the customer app and this tool, rather than something either app talks to independently.

**Phase 3 — Notifications live, menu management**
Order status changes trigger customer-facing push notifications. Menu management may join here too.

-----

## Open Questions for Owner

- [x] ~~Does he want role separation at all?~~ → Yes, decided: staff vs. admin, enforced from day one.
- [ ] What device(s) will actually be at the counter/kitchen during a shift — worth confirming before investing time in responsive polish vs. a single target layout
- [x] ~~Square webhook vs. direct app write for order ingestion?~~ → **Webhook only, permanently.** See Backend & Data Architecture section above.
- [x] ~~Oldest-unfulfilled-first or newest-first default sort?~~ → Neither — decided: kanban board, status-grouped, oldest-first within each column.
- [x] ~~Any existing process today (even informal/paper) for how orders currently get from "placed" to "made"?~~ → No existing process — Square's native order management is the bridge until this tool exists. See "How Orders Arrive" above for the sequencing consequence this creates.
- [ ] **New:** Should "Completed" be a 4th visible kanban column, or a filtered/archived view separate from the live board? (Functional behavior is decided; this is a layout detail.)

-----

## Relationship to Other Docs

This tool is the operational backend for both `sm-customer.md` and `sm-membership.md` — it doesn't introduce new concepts so much as it gives staff a way to *act* on the data those two systems already define (orders, members, events, perks). Same design system and component language should carry over where it makes sense, though information density and layout will differ for a counter/kitchen context vs. a customer's phone.

The `PREP_MINUTES` constant is now explicitly shared between this doc's urgency cue logic and `sm-customer.md`'s ETA messaging — it should be defined once (e.g. in shared backend config) and referenced by both apps, not duplicated as two separate hardcoded values.
