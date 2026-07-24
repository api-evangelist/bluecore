---
name: Manage Bluecore customer and consent
description: Authenticate, upsert a Bluecore Customer Profile, then set and read the customer's marketing eligibility (consent) status.
api: openapi/bluecore-openapi.yml
operations: [Authn_GetAccessToken, Profile_CreateOrUpdate, Eligibility_Update, Eligibility_Get]
---

# Manage a Bluecore customer and their consent

Use this to create/update a shopper and record their marketing opt-in in Bluecore.

## Auth
1. `Authn_GetAccessToken` — POST `https://auth.bluecore.com/oauth/token` with
   `{client_id, client_secret, audience: "https://a.bluecore.com", grant_type: "client_credentials"}`.
   The response `access_token` is a Bearer token valid 24h. Send `authorization: Bearer <token>`
   on every subsequent call. You need scopes `customers:write` and `eligibility:read`/`eligibility:write`.

## Steps
2. `Profile_CreateOrUpdate` — POST `/namespaces/{namespace}/customers`. Provide an `id`
   (`{email}` or `{phone_number}`), optional `link_ids` (max 4), and `attributes`
   (snake_case keys). Changes reflect immediately.
3. `Eligibility_Update` — POST `/namespaces/{namespace}/eligibility_events` with the same
   `id` and a `state` of `{event: OPTIN|UNSUBSCRIBE|SIGNUP, message_type: MARKETING, consented_at}`.
   Use `SIGNUP` (not `OPTIN`) to trigger the SMS/MMS double opt-in flow.
4. `Eligibility_Get` — GET `/namespaces/{namespace}/eligibility?email=...` to confirm the
   resulting `status` (`SUBSCRIBED` / `UNSUBSCRIBED` / `PENDING_CONFIRMATION`).

## Rules
- Emails are lower-cased and trimmed; phone numbers must be E.164 (`+15556667777`).
- Rate limits: burst 100/s, steady 1000/m per account; honor `RateLimit-*` headers; back off on 429.
- Errors are `google.rpc.Status` (`code`/`message`/`details`); capture `x-bluecore-id` for support.
