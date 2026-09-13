---
name: Produce a Kelley Blue Book Instant Cash Offer
description: Configure a vehicle, create a prospect with history, condition and contact answers, and generate a Kelley Blue Book Instant Cash Offer through the Cox Automotive ICO API.
api: openapi/cox-automotive-kbb-instant-cash-offer-openapi.yml
operations:
  - GET /vehicles/vin/{vin}
  - GET /vehicles/eligibility
  - GET /plate2vin/lookup
  - POST /prospects
  - PATCH /prospects/{prospectId}/history
  - PATCH /prospects/{prospectId}/conditions
  - PATCH /prospects/{prospectId}/options
  - PATCH /prospects/{prospectId}/contactInfo
  - POST /offers
  - GET /offers/{offerId}
generated: '2026-09-13'
method: generated
source: openapi/cox-automotive-kbb-instant-cash-offer-openapi.yml, https://developer.kbb.com/ico/1-Default
---

# Produce a Kelley Blue Book Instant Cash Offer

Three concepts, in order: a **Vehicle** configuration creates a **Prospect**; a submitted Prospect
creates an **Offer**. Creating the Offer **ends the Prospect's lifecycle and invalidates the
`prospectId`** — this is a one-way door.

Base URI: `https://api.kbb.com/ico/v1` (production), `https://sandbox.api.kbb.com/ico/v1` (sandbox).
Separate key entitlements are required for each. Auth is `?api_key=…` on the query string.

## Steps

1. **Identify the vehicle.** `GET /vehicles/vin/{vin}`, or `GET /plate2vin/lookup` when you only have a
   licence plate and state (`GET /plate2vin/states` lists the supported states). Fall back to the
   taxonomy endpoints — `/vehicles/years`, `/makes`, `/models`, `/trims`, `/transmissions`, `/engines`,
   `/drivetrains`, `/colors`, `/options` — when the VIN is ambiguous.
2. **Check eligibility before spending anything.** `GET /vehicles/eligibility`. This is your dry run.
   Not every vehicle can receive an offer; insufficient market data, a recent auction, or a
   deliberately disqualified vehicle all produce an `INELIGIBLE` result later if you skip this.
3. **Create the prospect.** `POST /prospects` with year, make, model, trim, transmission, engine,
   drivetrain, colour, mileage and a dealer ID. All are required.
4. **Answer the questionnaires.** Required, not optional:
   - `PATCH /prospects/{prospectId}/history` — ownership, title, history report, insurance claim, odour,
     service records, key sets, auction and rental questions.
   - `PATCH /prospects/{prospectId}/conditions` — condition and aftermarket modifications.
   - `PATCH /prospects/{prospectId}/options` — deviations from standard options.
   - `PATCH /prospects/{prospectId}/contactInfo` — name, email, confirm-email, phone, legal acceptance.
   Read each back with the matching `GET` before submitting. Result codes `400091` and `400092` mean a
   questionnaire is incomplete.
5. **Create the offer.** `POST /offers`. Returns `offerId`, `amount`, and `priceAdvisor` (a Kelley Blue
   Book Price Advisor URL that **may legitimately be empty** — if it is, do not render the Price Advisor
   link at all).
6. **Track it.** `GET /offers/{offerId}`. Status moves through `CLEARED`, `FR` (pending adjuster
   review), `INSPECTION`, `INELIGIBLE`, `EXPIRED`, and then the dealer-side states `GROUNDED`,
   `CONFIRMED`, `PURCHASED`, `AUCTION`, `RECEIVED`, `SOLD`.

## Rules an agent must follow

- **This flow is not idempotent and not reversible.** There is no `Idempotency-Key`, and no cancel, void
  or re-issue operation for an Offer. A retried `POST /offers` returns result code `400093`
  "Offer already processed" rather than replaying the original result. **Never blind-retry `POST /offers`
  on a timeout — read the prospect back first.**
- **Read `meta.codes[]`, not just the HTTP status.** Every response is a `{meta, data}` envelope.
  `meta.codes[]` carries six-digit Cox Automotive result codes; the full registry of 44 is in
  `errors/cox-automotive-error-codes.yml`. `400021` is a malformed VIN, `400020` an unserviceable ZIP,
  `400030` an unavailable dealer, `403000` invalid authorization.
- `meta.links[]` carries `{rel, href, method}` — follow it rather than rebuilding URLs.
- Offers expire, and **the expiry window is not published**. Do not promise a customer a duration the
  documentation does not state.
