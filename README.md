# Cox Automotive

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **Not from the company, and here with a question?** You are welcome here — we would rather be the
> front line and point you the right way than have a good report go nowhere. What this repository
> can answer is narrow, though, so it is worth knowing who you are actually looking for:
>
> - **A question about how the API works, an account, billing, or a bug in the service** — that is
>   the company's own support, not us. We profile this API; we do not operate it and cannot see
>   your account.
> - **A bug in an open-source project we only catalog** — file it on that project's own repository.
>   This has happened with a real and correct bug report that reached us instead of the people who
>   could fix it, which helped nobody.
> - **Anything about this listing itself** — the description, the tags, the rating, a missing or
>   wrong artifact — is ours. Open an issue here.
> - **Not sure, or something general about API Evangelist or APIs.io** — open an issue on the
>   [APIs.io Inbox](https://github.com/api-search/inbox) and we will route it.
>
> **This repository contains no software, and we will never ask you to download anything.** There is
> no build, release, installer, or binary here — only text and machine-readable API descriptions, so
> there is nothing here that can be "corrupt" or need "repairing". Any issue, comment, or email
> claiming otherwise and offering a download link is not from us and is hostile. Do not follow the
> link; it is a lure. Report it to GitHub and, if you like, tell us at
> [info@apievangelist.com](mailto:info@apievangelist.com) so we can take it down.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

Cox Automotive is one of the world's largest providers of products and services spanning the automotive
ecosystem, operating a portfolio that includes **Manheim** (wholesale vehicle auctions and remarketing),
**Kelley Blue Book** (vehicle valuations and editorial data), **Autotrader**, **Dealertrack**, **vAuto**,
**VinSolutions**, **Xtime**, **Dealer.com** and **HomeNet**.

## What this profile covers

Cox Automotive's public API surface is split across three developer properties, with three different
authentication models and three different error envelopes:

| Property | What it publishes | Machine-readable contract |
|---|---|---|
| [developer.kbb.com](https://developer.kbb.com/) | IDWS 4.0 Vehicle and Editorial, Advertising Data, Instant Cash Offer, Batch VIN | **Yes** — five Swagger 2.0 specs, 78 paths |
| [developer.manheim.com](https://developer.manheim.com/) | Hypermedia REST suite over the auction lifecycle, plus a publish/subscribe event service | No |
| [developer.coxautoinc.com](https://developer.coxautoinc.com/) | The company-wide API Storefront; 72 runtime components are named on the public status page | No — behind Okta / Entra ID sign-in |

All five Kelley Blue Book contracts are harvested verbatim into `openapi/_original/`. Everything else in
this repository is derived from those contracts or read from a public reference page, with the source URL
recorded on each artifact.

## Notable findings

- **No OpenAPI 3.x anywhere.** Every published contract is Swagger 2.0.
- **No idempotency contract**, across a write surface that includes unit creation, auction offering
  creation, lane assignment and instant-cash-offer creation.
- **A real event service with no AsyncAPI.** Manheim Events supports resource/type/VIN/expression
  filtering, replay and response expansion, and is documented only in prose. It is also mid-migration to
  *Eventer* on the API Storefront, which is closed to unauthenticated readers.
- **OAuth scopes are enforced but undocumented** — the token endpoint accepts a `scope` parameter and no
  scope vocabulary is published.
- **No first-party SDK, CLI, MCP server or A2A agent card.**
- **Two RFC 9116 `security.txt` documents** are served, both pointing at one responsible-disclosure
  policy. No bug bounty is offered.
- Access is never self-serve. Every environment on every property requires a human review before a key
  is issued.
