---
name: Subscribe to Manheim auction event notifications
description: Register a subscriber, create filtered subscriptions, receive NOUN.VERB auction lifecycle events on a callback, and reconcile missed deliveries on the Cox Automotive Manheim Events API.
api: https://developer.manheim.com/apis/events/index.html
operations:
  - POST /subscribers
  - GET /subscribers/mine
  - POST /subscriptions
  - GET /subscriptions/mine
  - DELETE /subscriptions/id/{ID}
  - POST /expand
  - GET /events/subscription/{ID}
generated: '2026-09-13'
method: generated
source: https://developer.manheim.com/apis/events/index.html
---

# Subscribe to Manheim auction event notifications

Manheim publishes business events as a vehicle moves through the auction — the Inventory system emits
`UNIT.CREATED` and `CONSIGNMENT.CREATED`, and Inventory, Images and Valuations all notify over the same
bus. Subscribing replaces polling and replaces nightly batch reconciliation.

> **Read this first.** Both the Events and the Subscriptions reference pages carry a notice that this
> version **is no longer available for onboarding**. The replacement is **Eventer**, on the Cox
> Automotive API Storefront (<https://developer.coxautoinc.com>), whose reference is behind sign-in.
> Build against Eventer for anything new; this skill describes the documented surface an existing
> consumer still runs on. No retirement date has been published for the old surface.

Hosts: `https://api.manheim.com` (production), `https://uat.api.manheim.com` (pre-production),
`https://integration1.api.manheim.com` (QA).

## Authentication

OAuth 2.0 `client_credentials`, with the Mashery package key as `client_id` and its secret as the
client secret, base64-encoded together into a `Basic` authorization header.

**Use the legacy token endpoint for this API.** External customers calling the Subscriptions API with
the `client_credentials` grant must post to `https://api.manheim.com/oauth2/token` — *not*
`token.oauth2` — for every create, update, retrieve and delete method. Cache the token; refresh on
expiry, never on a timer.

## Steps

1. **Register a subscriber** for your company: `POST /subscribers`. Confirm with
   `GET /subscribers/mine`. You only do this once.
2. **Create one subscription per stream you care about:** `POST /subscriptions`, with
   `subscriber.href` pointing at the subscriber you just created (HTTPS only) and a `criteria[]` array.
   Filter by:
   - `resource` — a URL identifying a resource, such as an auction location. A company ID URL is
     required when you filter by type or by text.
   - `type` — an event type, e.g. `UNIT.CREATED`.
   - `text` — a VIN, to follow one vehicle through its whole lifecycle.
   - `richFilter` — an expression over the event body, e.g. `body.status=='SOLD'`.
   A `201 Created` returns a `Location` href carrying the generated subscription ID.
3. **Handle the delivery.** Event bodies are deliberately thin: `eventType`, `resource` (the URL of the
   changed thing), a small `body`, optional `relatedResources`, and a `created` timestamp that is *when
   the event happened*, not when it was delivered.
4. **Avoid the follow-up fetch.** `POST /expand` asks for delivery with the referenced API endpoint's
   response embedded in the event.
5. **Reconcile.** If your endpoint was down, replay from
   `GET /events/subscription/{ID}`, `GET /events/subscriber/{ID}`, `GET /events/resource/{resourceId}`
   or `GET /events/id/{ID}`. This is the only safety net — no retry policy is published.
6. **Tear down** with `DELETE /subscriptions/id/{ID}` and `DELETE /subscribers/id/{ID}`.

## Rules an agent must follow

- **No signature scheme is published.** Cox Automotive documents no HMAC, no shared secret and no mTLS
  on the callback, so a receiver cannot cryptographically verify a delivery. Treat the payload as a
  hint, then re-read the `resource` URL over an authenticated call before acting on anything that
  matters.
- **No retry policy is published either.** Assume at-least-once at best, and make your handler
  idempotent on `eventType` + `resource` + `created` — the API gives you no key to do it with.
- Event names are `NOUN.VERB` or `NOUN.NOUN.VERB`, uppercase, dot-separated. **There is no published
  event-type registry**; only `UNIT.CREATED` and `CONSIGNMENT.CREATED` are named as real business
  events. Discover the rest from live traffic and do not hard-code an exhaustive switch.
- Errors: `415` means your `Content-Type` is not `application/json`; `503` means the application server
  rejected the connection or hit a SQL exception — resubmit, then escalate. `596` from the gateway means
  the method or URL did not resolve to a registered service.
- Limits are 1,000 QPS and 2,000,000 calls/day per package key. Watch `X-Packagekey-Qps-Current` and
  `X-Packagekey-Quota-Current` on responses.
