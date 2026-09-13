---
name: Run a Kelley Blue Book Batch VIN job
description: Authenticate, submit a batch VIN decode/valuation job, poll it to completion, retrieve the output file, and cancel a running job on the Cox Automotive Batch VIN API.
api: openapi/cox-automotive-kbb-batch-vin-openapi.yml
operations:
  - Token_PostCredentials
  - BatchJobs_PostBatchJob
  - BatchJobs_GetBatchJobs
  - BatchJobs_GetBatchJob
  - BatchJobs_GetBatchJobInputFile
  - BatchJobs_GetBatchJobOutputFile
  - BatchJobs_CancelBatchJob
generated: '2026-09-13'
method: generated
source: openapi/cox-automotive-kbb-batch-vin-openapi.yml
---

# Run a Kelley Blue Book Batch VIN job

Batch VIN is the volume counterpart to IDWS 4.0, which explicitly does not support batch calls. It is
the only Cox Automotive API in this repository that requires **two** credentials at once.

Base path `/idbv` on `https://sandbox.api.kbb.com` (sandbox). Client credentials are the username and
password used at `https://idbv.syndication.kbb.com`.

## Steps

1. **Mint a token.** `Token_PostCredentials` — `POST /Token`, sending `client_id` (your username) and
   `client_secret` (your password). Read `access_token` from the response.
2. **Send both credentials on every subsequent call.** `?api_key=…` on the query string **and**
   `Authorization: Bearer {access_token}` in the header. Missing either yields 401.
3. **Submit the job.** `BatchJobs_PostBatchJob` — `POST /BatchJobs`.
4. **Poll it.** `BatchJobs_GetBatchJob` — `GET /BatchJobs/{id}`. List your jobs with
   `BatchJobs_GetBatchJobs` — `GET /BatchJobs`. There is no webhook or callback for job completion on
   this API; polling is the only completion signal Cox Automotive publishes.
5. **Retrieve the results.** `BatchJobs_GetBatchJobOutputFile` — `GET /batchjobs/{id}/output`. Re-read
   what you sent with `BatchJobs_GetBatchJobInputFile` — `GET /batchjobs/{id}/input`.
6. **Cancel if you need to.** `BatchJobs_CancelBatchJob` — `PUT /batchjobs/{id}/cancel`. This is the one
   documented reversal path on the Kelley Blue Book surface; **no cancellation window is published**, so
   do not assume a job can still be cancelled once it has started producing output.

## Rules an agent must follow

- **Cache and reuse the access token.** Cox Automotive's own guidance for its OAuth surface is explicit:
  cache the token and request a new one on expiry rather than on a timer. Minting a token per request
  burns quota.
- Path casing is inconsistent in the published contract — `/BatchJobs` and `/BatchJobs/{id}` are
  capitalised, `/batchjobs/{id}/output`, `/batchjobs/{id}/input` and `/batchjobs/{id}/cancel` are not.
  Use the exact spelling from the spec; do not normalise it.
- No idempotency key exists. A retried `POST /BatchJobs` will create a second job. List jobs first.
