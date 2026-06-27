# Salus Market — Membership Program (Concept Doc)

## Status

This is a **planning document**, progressively moving toward a build spec. It exists to capture the shape of the idea so that when it's time to build, the team (Hunter + owner) already knows what they're executing. This doc is a companion to `sm-customer.md` (the mobile ordering app prototype) — the membership program is not a separate product, it's a layer on top of that same app.

**Revision note:** This version resolves the membership-mechanics open questions from the original draft (per-visit perk, Founding Member cutoff/perk, event scarcity). Two threads remain genuinely open and are called out explicitly: evening hours as a business concept, and the expectation that perks themselves may still change before launch. The data model below is deliberately built so that kind of change is a data edit, not a rebuild.

-----

## Core Premise

A recurring monthly subscription that makes someone a standing supporter of Salus Market — **"support local," not "VIP."** The goal is not to set members apart from other customers in a way that feels exclusionary; the goal is to give people a meaningful way to say "I'm part of this place" and have that membership feel worth it, ongoing, month after month.

That said — this is explicitly **a balance**, not a pure equality play. The program has to be genuinely *desirable*. People won't subscribe to something that feels like a tip jar. There has to be a real, felt sense of "I get something by being in this" alongside the communal framing. The two pillars below are the attempt to hold both of those at once.

This is also the direct answer to the loyalty-program fragmentation problem: most local loyalty programs fail to generate enthusiasm because they're generic points systems with weak emotional pull, and customers don't want to manage 20 different point balances across 20 different businesses. This program avoids that by not being a points system at all — no balance to track, nothing to manage. It's a standing subscription with perks that apply automatically or via simple notification, not a ledger.

-----

## The Two Pillars

### Pillar 1 — Access to Community Life
*The differentiator. This is what makes the program feel aspirational rather than transactional, and it's the direct software expression of the owner's vision for Salus Market as a relationship-building, communal space.*

- First notification of events hosted at Salus Market (community dinners, tastings, etc.)
- Early or guaranteed RSVP access before the general public
- Possibly a recurring members-oriented gathering (cadence TBD — e.g. monthly)

This pillar costs the business very little per-member (no COGS impact) but delivers high emotional value — it's literally "you're in the loop on the life of this place before everyone else." It's exclusivity of *belonging*, not exclusivity of *status* — nothing here puts a member's name or face in front of other customers. It's a private relationship between the member and the restaurant's calendar.

**Decided:** events are **uncapped** — no artificial capacity limit. The "first crack" mechanic is purely a timing edge: members see and can RSVP to events before the general public does, via two staggered windows (`member_rsvp_window_start` before `public_rsvp_window_start`). There is no waitlist or "event full" state to design for.

### Pillar 2 — Tangible Per-Visit Value
*The everyday, tangible reminder that the subscription is worth it — independent of whether an event is happening that month.*

**Decided (current best guess — see Perk Model below for why this is built to flex):**

- **Free drip coffee** — standing add-on attached to every order, regardless of time of day.
- **Free evening drink** — a second standing perk, *not competing with* the drip coffee perk. This is additive. It is contingent on Salus Market having evening hours, which do not currently exist (see **Open Thread: Evening Hours** below). The perk is modeled in the system now so it's ready the moment evening hours are real, but it has no effect until then.

Both perks are intentionally simple from a software standpoint — a flag/lookup checked at order time, not a balance to manage or redeem. No tracking, no math, no friction.

**Important caveat carried forward from this round of decisions:** neither perk should be treated as final. The owner may change what the standing perk is. The system is built so that's a data edit (see Perk Model), not a code change.

-----

## What's Explicitly Out (For Now)

- **Points/credit balances** — directly counter to the "not another fragmented loyalty program" goal
- **Public-facing recognition** (e.g. name on a community board) — reads as "look at me," which cuts against the communal, non-exclusionary tone the owner wants
- **Priority queuing / skip-the-line** — reconsidered. Cutting in line, even reframed, risks feeling like members go first at the expense of others, which conflicts with communal equality. If revisited, it should be reframed as something like staff visibility ("we know to say hi") rather than literal queue position.
- **Event capacity caps / waitlists** — decided against. See Pillar 1 above.

-----

## Open Thread: Evening Hours

Salus Market's stated hours (`sm-customer.md`) run morning through early afternoon (latest close 3:00 PM Mon–Sat, 2:00 PM Sun). The owner has separately raised the idea of evening beverage service, which doesn't exist as a confirmed business concept yet — no hours, no menu, no operational decision.

This matters for membership because the **"free evening drink" perk depends on it.** It's tracked here as:

- Not a replacement for or competitor to the daytime drip-coffee perk — both are intended to be active simultaneously, per owner direction.
- Blocked on a real business decision (does Salus Market operate evenings at all, and if so what does that service look like) that lives outside this doc's scope — likely belongs as a new open question in `sm-customer.md`'s hours/business-logic section.
- Represented in the data model today (see below) so no rework is needed once it's confirmed — the perk and its `applies_to: evening_hours` condition already exist, they're just inert until evening hours exist.

-----

## Founding Member — A Related but Separate Mechanism

**Important distinction:** Founding Member is a *launch-window cohort*, not the ongoing program. It has a shelf life and is meant to generate early excitement and reward the people who believed in Salus Market from day one. The ongoing subscription program above is meant to endure indefinitely.

**Mechanism (unchanged from original recommendation):** Founding Member is not a separate system. It's a permanent badge/flag layered onto whoever joins the core ongoing subscription during the launch window — one boolean (`is_founding_member`) plus a sequential number, not a parallel membership engine.

**Decided — cutoff:** count-based. **First 100 members** are Founding Members. (Not date-based — a pure date cutoff was considered but doesn't create the same urgency, and undercuts the appeal of sequential numbering, e.g. "Member No. 7.")

**Decided — perk:** a **fixed percentage discount off the ongoing membership's current list price, forever.** Example: a Founding Member locked at 20% off always pays 20% less than whatever the current list price is — if the owner raises the base price later, the Founding Member's price rises too, but always at the same relative discount. This is a deliberate choice over a flat locked dollar amount: it requires no price-migration step when list price changes, and it scales naturally.

-----

## Perk Model — Why Perks Are Data, Not Code

Because the actual perks (drip coffee, evening drink, possibly others later) are explicitly not final, perks are modeled as rows in a table the owner/admin can edit via the staff tool — not hardcoded logic. Adding, retiring, or changing a perk should never require a code deploy.

A `MembershipTier` exists in the schema from day one, but in practice there is only one row ("Standard") for the foreseeable future — this resolves the original "single tier vs. tiered structure" question without overbuilding: the door is open for a second tier later, but nothing about today's build assumes one will ever exist.

-----

## Data Model (Resolved)

```
Perk
  - id
  - name                    // "Free Drip Coffee", "Free Evening Drink"
  - description
  - applies_to              // "every_order" | "evening_hours" | future values
  - active (boolean)

MembershipTier
  - id
  - name                    // "Standard" — only one row exists today
  - list_price
  - perks[]                 // join table: MembershipTierPerk(tier_id, perk_id)

Member
  - id
  - name / contact (phone or email — needed for auth + notifications)
  - tier_id                 // FK to MembershipTier, currently always "Standard"
  - subscription_status     // active, paused, canceled
  - is_founding_member (boolean)
  - founding_member_number  // nullable int, assigned sequentially, capped at 100
  - founding_discount_pct   // e.g. 0.20 — locked at signup, applied to current list_price at billing time
  - join_date

Subscription
  - member_id
  - billing_provider        // Stripe likely, separate from Square POS — still an open question, see below
  - current_charge          // = tier.list_price * (1 - founding_discount_pct if founding else 0)
  - billing_cycle_status

Event
  - id
  - title, datetime, description
  - capacity                // always null — uncapped, kept nullable for schema flexibility only
  - member_rsvp_window_start
  - public_rsvp_window_start

RSVP
  - event_id
  - member_id
  - timestamp

Order (existing entity from sm-customer.md)
  - + member_id (nullable foreign key)
  - + member_perk_applied (boolean / type)
```

**Still open — narrowed to a specific technical spike:** Square handles payment for food orders, but the *subscription billing* (monthly membership fee) is a separate recurring charge unrelated to a food order, with no physical good being delivered. Two real candidates, each with a specific friction point:

- **Square Subscriptions API** — genuinely capable (recurring billing, webhooks, card-on-file charging all real and current). Two open concerns, not yet confirmed either way:
  1. Square's docs describe subscription catalog items as needing to be "shipped to the customer" and state subscription items aren't yet available for in-person pickup — unclear whether this is a hard block for a no-physical-good membership, or just unused/irrelevant fields for this use case. **Needs a direct sandbox test**, not a guess.
  2. The Founding Member mechanic (fixed % discount off whatever the *current* list price is, forever) doesn't map cleanly onto Square's plan → plan variation → subscription pricing model, which sets price per variation rather than as a relative discount. Likely workable via a separate plan variation per discount tier or manual per-subscription price overrides, but more friction than a first-class discount object.
- **Stripe** — first-class support for non-physical recurring billing and percentage-based coupons that automatically track list-price changes, which maps directly onto the Founding Member mechanic as designed. Costs a second payment provider/vendor relationship alongside Square.

**Decided approach:** run a bounded Square sandbox spike before committing either direction. Concretely:
1. Create a test subscription plan in the Square sandbox with no shipping/fulfillment data attached — confirm whether it can be created and charged without a physical delivery component.
2. Attempt to model the founding-member discount as a Square subscription plan variation — confirm whether per-subscription relative-percentage pricing is achievable without an unreasonable number of plan variations.
3. Time-box this to a single short spike (this is a go/no-go test, not an integration build) before deciding between Square Subscriptions and Stripe.

This spike is a prerequisite for finalizing the `Subscription.billing_provider` field below — treat it as blocking that one field, not the rest of the membership data model, which is provider-agnostic as designed.

-----

## How This Integrates With the Existing Ordering App

Confirmed from owner conversation: **Square handles payment only, as the final step.** The owner wants the full cart-building and ordering *experience* to be custom-branded and outside Square's generic interface — this matches the existing `sm-customer.md` architecture exactly (Square Web Payments SDK invoked only at the Payment screen). Membership doesn't change this; it adds metadata and logic that travel through the existing flow.

Mapped against the existing screen flow (`Home → Menu → Cart → Payment (Square) → Confirmation`):

| Screen | Membership touchpoint |
|---|---|
| **Home** | Member status indicator / entry point to join. Possibly a lightweight "Members" or "Community" section. |
| **Menu** | No major change — Pillar 2 perks don't need to be "selected," they're automatic. |
| **Cart** | Active perks (drip coffee always; evening drink once evening hours exist) applied automatically if the user is a recognized member (auto-added line item or flag). |
| **Payment** | Member-adjusted total passed to Square as part of the order — perk is reflected in what Square actually charges, not handled out-of-band. |
| **Confirmation** | No major change initially. Could eventually reflect "thanks for being a member" messaging. |
| **New: Events/Community screen** | Net-new surface — not in the original prototype. Needed for Pillar 1 (event list, RSVP, notification preference). This is the first real expansion of the app beyond the linear ordering funnel. |

**Notable architectural consequence:** Pillar 1 requires push notifications and a simple event/RSVP data structure — both already flagged as "post-MVP, requires backend" in `sm-customer.md`. Membership is effectively what justifies building that backend sooner rather than later, since recurring billing + member identity also requires persistent state and auth that the pure ordering MVP could previously avoid.

-----

## Phasing (Proposed)

**Phase 1 — Manual/low-tech validation**
Test the concept with minimal software: a simple recurring payment link (Square invoicing or Stripe), staff manually aware of who's a member (e.g. a shared list), perks honored manually at the register. Validates demand before any custom engineering.

**Phase 2 — Integrated into the app**
Member auth/login added to the ordering app. Pillar 2 perks auto-apply at checkout. Member status is real, not staff-memory-based. The `Perk` table becomes editable by admin role in the staff tool at this phase — the owner can change the standing perk without a code deploy.

**Phase 3 — Full community layer**
Events/RSVP screen built out. Push notifications live. This is where Pillar 1 becomes a real software feature rather than a promise.

-----

## Open Questions for Owner

- [x] ~~What's the actual per-visit add-on?~~ → Free drip coffee (every order) + free evening drink (pending evening hours). May still change — perk is editable data, not fixed.
- [ ] Price point for the monthly subscription?
- [ ] Cadence and format of the members' community event — monthly? Is there a vision for this already, or does it need to be invented?
- [x] ~~Should event capacity be genuinely limited?~~ → No, uncapped. Early notice is the only edge.
- [x] ~~Founding Member cutoff and perk?~~ → First 100 members (count-based); fixed % discount off current list price, locked forever.
- [ ] Is Square's own subscription/invoicing capability sufficient for recurring membership billing, or does this need Stripe? → **Narrowed to a specific sandbox spike** — see "Still open" note above. Not a guess between the two anymore; it's a bounded technical test.
- [x] ~~Single tier or tiered structure?~~ → Single tier (`MembershipTier` table exists for future flexibility, but only one row today).
- [ ] **New:** Is evening beverage service a real, planned business concept? If so, what hours/scope? (Blocks the evening-drink perk from ever activating.)

-----

## Relationship to `sm-customer.md`

This doc assumes and builds on every architectural decision already made there: React/Vite PWA, Square as payment-only rail, charcoal/terracotta/cream design system, linear funnel UX. Membership should look and feel like a native part of that same app, not a bolted-on feature with its own visual language. When this moves to implementation, the Events/Community screen should be designed in the same component structure and design token system already established.

The evening-hours open thread, if resolved, should also prompt an update to `sm-customer.md`'s business-hours logic (currently a single Mon–Sun schedule block) to support a possible second daily window.
