---
name: Value a vehicle with Kelley Blue Book IDWS 4.0
description: Decode a VIN, apply or validate a configuration, and retrieve Kelley Blue Book values and cost-to-own for a Cox Automotive IDWS 4.0 vehicle.
api: openapi/cox-automotive-kbb-idws-vehicle-openapi.yml
operations:
  - IdwsVehicleVinIdByVinGet
  - IdwsVehicleValidateconfigurationPost
  - IdwsVehicleApplyconfigurationPost
  - IdwsVehicleVehicleoptionsGet
  - IdwsVehicleValuesPost
  - IdwsVehicleCosttoownPost
  - IdwsVehicleErrorcodesGet
generated: '2026-09-13'
method: generated
source: openapi/cox-automotive-kbb-idws-vehicle-openapi.yml
---

# Value a vehicle with Kelley Blue Book IDWS 4.0

IDWS 4.0 is Kelley Blue Book's RESTful vehicle service. It does **not** support batch calls — one
vehicle per request. Use the Batch VIN API for volume.

## Before you start

- You need a Mashery-issued `api_key`. It is not self-serve: request it at
  <https://b2b.kbb.com/contact/> or <https://developer.kbb.com/access>. Cox Automotive reviews the
  organisation and the application first, and issues a separate key per environment and per application.
- Send the key as a **query-string parameter**: `?api_key=…`. There is no header form.
- Declare your major version on every GET: `Accept: application/vnd.coxauto.v1+json`. Omitting it means
  your client breaks silently when the API is versioned.
- Server-side only. IDWS does **not** support CORS.
- Sandbox host: `https://idws-sandbox.datasolutions.coxautoinc.com`. Production:
  `https://idws.datasolutions.coxautoinc.com`. **Sandbox valuations are deliberately six months stale** —
  never validate pricing logic against them.

## Steps

1. **Decode the VIN.** `IdwsVehicleVinIdByVinGet` — `GET /idws/vehicle/vin/id/{vin}`. Returns the
   candidate vehicle(s) for that VIN. A VIN is 17 characters and never contains I, O or Q; validate that
   client-side before spending a call.
2. **Resolve the configuration if the VIN is ambiguous.** Walk the taxonomy with
   `IdwsVehicleYearsGet`, `IdwsVehicleMakesGet`, `IdwsVehicleModelsGet`, `IdwsVehicleTrimsGet` to pin
   year, make, model and trim.
3. **Check the configuration before you commit to it.** `IdwsVehicleValidateconfigurationPost` —
   `POST /idws/vehicle/validateconfiguration`. This is the dry run: it tells you whether a proposed
   combination of options is buildable without changing anything.
4. **Apply option changes.** `IdwsVehicleApplyconfigurationPost` —
   `POST /idws/vehicle/applyconfiguration`. Takes a `VehicleConfigurationModel` plus
   `VehicleConfigurationChangeModel` entries and returns the resolved configuration together with a
   `WarningModel` list. **Read the warnings** — a change can be accepted and still degrade the valuation.
   Retrieve the available options first with `IdwsVehicleVehicleoptionsGet`.
5. **Get the values.** `IdwsVehicleValuesPost` — `POST /idws/vehicle/values`. Blue Book values for the
   configured vehicle.
6. **Get cost to own, if you need it.** `IdwsVehicleCosttoownPost` — `POST /idws/vehicle/costtoown`.
   Returns `ValueRatingAttributes` and `CostPerMileAttributes`.

## Rules an agent must follow

- **There is no idempotency contract on this API.** No `Idempotency-Key` header or parameter exists
  anywhere in the Cox Automotive surface. `applyconfiguration` and `values` are effectively read-shaped
  and safe to retry; treat anything that mutates state as unsafe to blind-retry and reconcile with a
  read instead.
- **Errors.** IDWS returns `IDWS.Platform.ResponseMessage.ReturnResponse`, not RFC 9457 problem+json.
  Every operation declares 400 (bad request), 401 (invalid or revoked `api_key`) and 500. Fetch the
  registry once with `IdwsVehicleErrorcodesGet` — `GET /idws/vehicle/errorcodes` — and cache it.
- **Rate limits.** IDWS publishes none. The Manheim side of Cox Automotive publishes 1,000 QPS and
  2,000,000 calls/day per package key; assume a comparable gateway ceiling and back off on 403.
- Do not hard-code the sandbox host into production code paths — the two environments use different
  domains, not different keys on one domain.
