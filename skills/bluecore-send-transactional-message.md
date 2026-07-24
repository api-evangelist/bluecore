---
name: Send a Bluecore transactional message
description: Authenticate and send a transactional email/SMS/MMS through a Bluecore campaign using an idempotency key, then poll its delivery status.
api: openapi/bluecore-openapi.yml
operations: [Authn_GetAccessToken, Transactional_Send, Transactional_Get]
---

# Send a Bluecore transactional message

Use this for welcome, order/shipment confirmation, or loyalty-status messages.

## Auth
1. `Authn_GetAccessToken` — obtain a Bearer token (see the consent skill). Requires the
   `transactional:api` scope. Send `authorization: Bearer <token>`.

## Steps
2. `Transactional_Send` — POST `/namespaces/{namespace}/transactional-messages/{id}` where
   `{id}` is a **client-generated unique idempotency key** (alphanumeric plus `: _ . -`,
   max 128 chars). Body: `{message: {identifiers:[{email|phone_number}], campaign_ids:[...],
   product_ids?, customer_attributes?, use_draft?}}`. Reusing an `id` returns 409 (conflict),
   so generate a fresh id per send and store it.
3. `Transactional_Get` — GET `/namespaces/{namespace}/transactional-messages/{id}` with the
   same id to read the `statuses[]` history (`INITIATED` → `SENT` → `DELIVERED`, or
   `HALTED`/`DELIVERY_FAILURE` with `details`).

## Rules
- SMS/MMS bodies support `{{attribute}}` templating from `customer_attributes`.
- Eligibility and per-email send caps apply; a `test` send bypasses eligibility checks.
- Rate limits burst 100/s, steady 1000/m; honor `RateLimit-*`; retry 5xx (google.rpc.Status body).
