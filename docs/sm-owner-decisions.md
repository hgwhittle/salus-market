# Salus Market — Owner Decisions Needed

## Purpose

This is a consolidated list of business-only questions feeding the three planning docs (`sm-customer.md`, `sm-membership.md`, `sm-staff.md`). Resolved items show the decision and any consequences it created; still-open items are what's left to bring back to the owner.

-----

## Ordering App (Customer-Facing)

- [ ] **Menu items, prices, and descriptions.** Still placeholder data in `sm-customer.md`. Blocks: real menu build, pricing display, Square catalog setup.
- [ ] **Hours of operation.** Still placeholder (Mon–Fri 7am–3pm, Sat 8am–3pm, Sun 9am–2pm) — not yet confirmed.
- [ ] **Realistic average prep time.** Still hardcoded at 8 minutes. Drives both customer-facing ETA messaging and the staff tool's urgency cue (shared constant) — worth getting close to real.
- [x] **Carry-out only, or dine-in too?** → **Both.** Sub-question still open: does dine-in need a table number/identifier for staff to deliver to, or is it counter-pickup either way regardless of order type? Affects whether the Cart screen's order-type toggle needs a table-selection step.
- [ ] **Square account type** — Square for Restaurants or standard Square POS? Still unconfirmed.
- [x] **Order-ready notification: SMS or in-app only?** → **In-app status only.** No SMS provider/cost to evaluate — web push (already planned post-MVP) is the only notification channel needed.
- [ ] **Domain / app name for deployment.** Needed before production deploy.

## Evening Beverage Service — Confirmed Real, Details Pending

- [x] **Is evening service a real, planned concept?** → **Yes, confirmed real and thought out by the owner.** Exact hours and scope not yet known to Hunter.
- [x] **Separate menu or same menu, just later?** → **Separate, lighter, drinks-focused menu** is the likely shape.
- [x] **Continuation of daytime hours, or a distinct second service period?** → **Distinct second period** — likely closed in the gap between day and evening service, possibly different vibe/staffing.

**Architectural consequence (already designed for, not yet built):** the customer app's hours logic needs to support **two independent daily service windows** rather than one continuous block — e.g., open day, closed gap, open evening, closed overnight — not just an extended single window. Menu data needs a service-period field (`available_during: 'day' | 'evening' | 'both'`) so the Menu screen shows the right item set for whichever window is active. The membership Perk model already accommodates this (`applies_to: 'evening_hours'` was designed in from the start). This is logged as **designed-for-but-deferred**: a coding agent should build the flexible two-window structure now, with placeholder evening hours/menu, rather than retrofitting later once real hours are known.

- [ ] **Still needed from owner:** actual evening hours, actual evening menu items/prices.

## Membership Program

- [ ] **Monthly subscription price.** Not yet set. Blocks: membership launch entirely, even the cheap Phase 1 manual version — no way to validate demand without a number to charge.
- [ ] **Community event cadence and format.** Not yet decided — no existing vision confirmed yet, may need to be invented or may come from the owner later.

## Staff Tool (Operational Context)

- [ ] **What device(s) will be at the counter/kitchen?** Not yet known. Determines whether the kanban board needs serious responsive work or can target one layout.
- [x] **Existing process for orders going from "placed" to "made"?** → **There isn't an informal/paper process to replace — the owner will use Square's own native order management as the bridge until the custom staff tool is ready.** This is effectively a "Phase 0."

**Sequencing consequence:** the staff tool and customer app do **not** need to launch together — confirmed the staff tool can come online first, while customers are still ordering through Square-native checkout. This is part of why the backend ingestion decision (see `sm-staff.md`, Backend & Data Architecture) landed on **Square webhook as the only, permanent ingestion path** — direct-write-from-the-customer-app was considered and rejected, partly because it can't be the sole path during this bridge period anyway.

-----

## Summary of What's Still Genuinely Open

1. Menu items, prices, descriptions (day + evening)
2. Hours of operation (day + evening)
3. Realistic prep time
4. Dine-in table/seating mechanic
5. Square account type
6. Domain / app name
7. Subscription price
8. Community event cadence/format
9. Counter/kitchen device(s)
