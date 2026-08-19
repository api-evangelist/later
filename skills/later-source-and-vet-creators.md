---
generated: '2026-08-13'
method: generated
name: Source and vet creators
description: Search the Later Influence creator index, read audience demographics, and run a brand-suitability check before adding a creator to a list.
api: openapi/later-influence-api-openapi.json
operations: [getGlobalUsers, getQueryTotal, get, getAudienceDemographics, getGlobalUserAudienceCountries, generateBrandSuitabilityReport, getBrandSuitabilityReport, addNewInfluencer, createGlobalUserList, createGlobalUserListMember]
source: >-
  Grounded in the Later Influence contract published at https://api.mavrck.co/api-docs;
  every operationId above was verified verbatim in
  openapi/later-influence-api-openapi.json on 2026-08-13.
---

# Source and vet creators

Find creators in Later Influence, check who their audience actually is, screen them for brand safety, and shortlist the ones that survive.

## Base URL
- `https://api.mavrck.co/v1`. Paths shown with a leading `/v1` resolve at the host root — see `conventions/later-conventions.yml`.

## Vocabulary
- A **global user** is the person record that spans communities. An **influencer** is that person scoped to one community, and a **membership** is the join between them.

## Auth
- `api-key` request header. See `authentication/later-authentication.yml`.

## Steps
1. **Search the index** — `getGlobalUsers` (`GET /v1/global-users`) with `q` for free text, `limit`/`offset` for paging, and `userFilter` for structured filtering. **`userFilter` is jsurl-encoded, not ordinary query-string encoded** — the contract says so in its own parameter description. Pair with `getQueryTotal` (`GET /v1/global-users-total`) to get a count, because the list operation does not return one.
2. **Read the profile** — `get` (`GET /v1/global-users/{id}`) for the full creator record and its connected social presences.
3. **Check the audience, not the follower count** — `getAudienceDemographics` (`GET /v1/global-users/{globalUserId}/audience-demographics`) and `getGlobalUserAudienceCountries` (`GET /v1/global-users/{globalUserId}/audience-countries`). If the data is stale, `refreshAudienceDemographics` (`POST /v1/global-users/{globalUserId}/audience-demographics/latest`) queues a refresh — it is asynchronous, so poll the GET afterwards.
4. **Screen for brand safety** — `generateBrandSuitabilityReport` (`POST /v1/global-users/{globalUserId}/brand-suitability`) to run the check, then `getBrandSuitabilityReport` (`GET` on the same path) to read it. `makeBrandSafetyRequest` (`POST /v1/global-users/{globalUserId}/brand-safety-request`) is the older entry point for the same screen.
5. **Add a creator you do not already have** — `addNewInfluencer` (`PUT /v1/global-users`). Dry-run it first with `previewAddCreator` (`PUT /v1/global-users/preview`) so you see what would be created.
6. **Shortlist** — `createGlobalUserList` (`POST /global-user-lists`) then `createGlobalUserListMember` (`POST /global-user-lists/{globalUserListId}/members`) to move the survivors onto a curated list you can attach to a campaign.

## Errors
- `404 Resource Not Found` — ids are community-scoped and do not resolve across accounts.
- `400 Missing Parameter` — the legacy envelope returns the offending names in `params[]`.
- See `errors/later-problem-types.yml`.

## Notes
- **This flow reads personal data about real creators.** Audience demographics and brand-suitability reports are profiling outputs. `markAsNonContactable` (`POST /v1/global-users/{globalUserId}/mark-as-non-contactable`) and `deletePublicSummaryRecord` exist for a reason — honour them.
- Brand suitability has its own 33-operation surface (`BrandSuitability` and `BrandSuitabilityHarness` tags) if you need more than the summary report.
