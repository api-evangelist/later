---
generated: '2026-08-13'
method: generated
name: Launch an influencer campaign
description: Create a campaign in Later Influence, load candidate creators onto it, bulk opt them in and send the brief.
api: openapi/later-influence-api-openapi.json
operations: [getCommunities, createCampaign, createActionGroup, getAllActionGroupCandidates, bulkOptIn, confirmActionGroupParticipation, sendBulkMessageForActionGroup, getWorkflowStatusBuckets]
source: >-
  Grounded in the Later Influence contract published at https://api.mavrck.co/api-docs;
  every operationId above was verified verbatim in
  openapi/later-influence-api-openapi.json on 2026-08-13.
---

# Launch an influencer campaign

Stand up a campaign in Later Influence (formerly Mavrck), get creators onto it, and brief them.

## Base URL
- `https://api.mavrck.co/v1` — verified live 2026-08-13.
- **Trap:** 40 paths in the contract carry their own literal `/v1` prefix (`/v1/campaigns`, `/v1/global-users`). Those resolve at the host ROOT, not under `basePath`. Do not concatenate `basePath` + path for them. See `conventions/later-conventions.yml`.

## Vocabulary
- An **action group** IS a campaign. `/action-groups` is the older brand-facing name; `/v1/campaigns` is the newer one. Both exist and both are live.

## Auth
- Send the `api-key` request header. See `authentication/later-authentication.yml`.
- A missing or wrong key returns **403**, not 401.

## Idempotency
- **There is none.** The API publishes no idempotency key on any of its 687 operations. Every POST in this flow — especially `bulkOptIn` and `sendBulkMessageForActionGroup` — is unsafe to retry blindly, because a retry sends the message again. Record the ids you created and check before re-running.

## Steps
1. **Resolve the community** — `getCommunities` (`GET /communities`) to find the `communityId` the campaign belongs to. Almost every other call is scoped to it.
2. **Create the campaign** — `createCampaign` (`POST /v1/campaigns`), or `createActionGroup` (`POST /action-groups`) if you are working against the action-group surface. Capture the id.
3. **Pull the candidate pool** — `getAllActionGroupCandidates` (`GET /action-groups/{id}/candidates`) with `limit`/`offset` and optional `orderBy` + `orderDirection` (`ASC`/`DESC`). Most list operations return a bare array with no total count — page until short.
4. **Opt the creators in** — `bulkOptIn` (`POST /action-groups/{actionGroupId}/bulk-opt-in`), then `confirmActionGroupParticipation` (`POST /action-groups/{actionGroupId}/confirm-participation`) for creators who accept.
5. **Send the brief** — `sendBulkMessageForActionGroup` (`POST /action-groups/{actionGroupId}/bulk-message`) to message everyone on the campaign at once, or `sendMessageForActionGroup` (`POST /action-groups/{actionGroupId}/list-members/{globalUserId}`) for one creator.
6. **Watch the funnel** — `getWorkflowStatusBuckets` (`GET /action-groups/{id}/workflow-status-summary`) for the count of creators in each workflow status.

## Errors
- `403` with `{"type":"RESOURCE_FORBIDDEN_ERROR",...}` — missing or invalid `api-key`.
- `400` `Validation failed` — the request body failed the operation schema; read `details`.
- Two different error envelopes are in play (`{type,error}` and `{message,details}`). Handle both. See `errors/later-problem-types.yml`.

## Notes
- There are no webhooks you can subscribe to. Campaign, opt-in and status changes are poll-only. See `asyncapi/later-influence-webhooks.yml`.
- No rate-limit headers are published. Back off on 500s, which are declared on 685 of 687 operations.
