---
generated: '2026-08-13'
method: generated
name: Fulfill creator rewards and payments
description: Take Later Influence reward wins from earned to paid — read the queue, fulfil singly or in bulk, and handle reversals.
api: openapi/later-influence-api-openapi.json
operations: [getSummary, getAllRewardWins, getAllInfluencerRewardWins, fulfillRewardWin, bulkFulfillRewardWinsObjects, unfulfillRewardWin, reverseRewardWin, voidTipaltiPayment, getWinners, disqualifyUser, requalifyUser]
source: >-
  Grounded in the Later Influence contract published at https://api.mavrck.co/api-docs;
  every operationId above was verified verbatim in
  openapi/later-influence-api-openapi.json on 2026-08-13.
---

# Fulfill creator rewards and payments

Reward wins are how Later Influence pays creators. This flow moves real money.

## Base URL
- `https://api.mavrck.co/v1`.

## Auth
- `api-key` request header. See `authentication/later-authentication.yml`.

## READ THIS FIRST — no idempotency
The Later Influence API publishes **no idempotency key on any operation**. `fulfillRewardWin` and `bulkFulfillRewardWinsObjects` are payment instructions with no replay protection. A timed-out request that you retry may pay twice. Before any retry, re-read the reward win and confirm its state. See `conventions/later-conventions.yml`.

## Steps
1. **Size the queue** — `getSummary` (`GET /v1/reward-wins/dashboard/summary`) for the dashboard totals.
2. **List what is owed** — `getAllRewardWins` (`GET /v1/reward-wins`), or `getAllInfluencerRewardWins` (`GET /influencers/{influencerId}/reward-wins`) for one creator.
3. **Confirm the payee is onboarded** — the creator must have a payment recipient. On the influencer-facing surface those are created through the Stripe Connect onboarding links (`/v1/later-influencers/{influencerIdOrCreatorId}/payment-providers/stripe/onboarding-link`).
4. **Fulfil** — `fulfillRewardWin` (`PUT /v1/reward-wins/{id}/fulfillment`) for one, or `bulkFulfillRewardWinsObjects` (`PUT /v1/reward-wins/fulfill-objects/bulk`) / `bulkFulfillRewardWinsObjectsWithParams` for many. Bulk fulfilment is the highest-blast-radius call in this API.
5. **Correct a mistake** — `unfulfillRewardWin` (`PUT /v1/reward-wins/{id}/unfulfillment`), `reverseRewardWin` (`PUT /v1/reward-wins/{id}/reversal`), or `voidTipaltiPayment` (`POST /v1/reward-wins/{id}/void-payment`) when the payout went out through Tipalti.
6. **Incentive winners** — `getWinners` (`GET /incentives/{incentiveId}/winners`), `selectWinnersRandom` (`PUT` on the same path), and `disqualifyUser` / `requalifyUser` on `/incentives/{incentiveId}/winners/{id}/…` when a winner is ineligible.

## Payment rails in this API
Stripe (`StripePayments`, `PaymentRecipient`), Tipalti payouts, cash payments (`CashPayments`), gift cards and Tango Cards, and Shopify discount codes. Which rail a reward win uses is a property of the reward, not a separate endpoint.

## Errors
- `403` — missing/invalid `api-key`; the most common failure in this API.
- `409 Conflict` — the reward win is already fulfilled or reversed.
- `500 Server Error` — declared on nearly every operation. Do **not** blind-retry a fulfilment on a 500; read the reward win back first.
- See `errors/later-problem-types.yml`.

## Notes
- No webhook fires on payment state change. Poll `getAllRewardWins` and reconcile.
- Payout status vocabulary appears in the contract's enums: `PENDING_PAYMENT`, `PAYMENT_APPROVED`, `PAYMENT_CANCELED`, `PAYOUT_PENDING`, `PAYOUT_FAILED`, `TRANSFER_SUCCESS`, `BLOCKED_BY_TIPALTI`, `FAILED_PAYMENT`.
