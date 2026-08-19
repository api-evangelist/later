---
generated: '2026-08-13'
method: generated
name: Review and approve creator content
description: Work the Later Influence content approval queue — read submitted concepts and drafts, leave feedback, set status, then approve or reject the resulting activation.
api: openapi/later-influence-api-openapi.json
operations: [getConceptContentForListMember, updateConceptContentFeedback, updateConceptContentStatus, getDraftContentForListMember, updateDraftContentFeedback, updateDraftContentStatus, getActionGroupActivations, approveApprovalStatus, rejectApprovalStatus, unrejectApprovalStatus]
source: >-
  Grounded in the Later Influence contract published at https://api.mavrck.co/api-docs;
  every operationId above was verified verbatim in
  openapi/later-influence-api-openapi.json on 2026-08-13.
---

# Review and approve creator content

The approval chain in Later Influence runs concept → draft → activation. Each stage has its own content object, its own feedback endpoint and its own status endpoint.

## Base URL
- `https://api.mavrck.co/v1`.

## Auth
- `api-key` request header. See `authentication/later-authentication.yml`.

## Steps
1. **Read the concept** — `getConceptContentForListMember` (`GET /v1/list-member/{listMemberId}/concept-content`) for what the creator proposed.
2. **Leave concept feedback** — `updateConceptContentFeedback` (`PUT /v1/concept-content/{conceptContentId}/feedback`); read it back with the matching `GET`, and comment inline with `createConceptContentComment` (`POST /v1/concept-content/{conceptContentId}/comments`).
3. **Set the concept status** — `updateConceptContentStatus` (`PUT /v1/concept-content/{conceptContentId}/status`).
4. **Read the draft** — `getDraftContentForListMember` (`GET /v1/list-member/{listMemberId}/draft-content`), or `getDraftContent` (`GET /v1/influencer-draft-content/{actionGroupId}`) for everything on one campaign.
5. **Leave draft feedback and set status** — `updateDraftContentFeedback` (`PUT /v1/draft-content/{draftContentId}/feedback`) then `updateDraftContentStatus` (`PUT /v1/draft-content/{draftContentId}/status`).
6. **Find the resulting activation** — `getActionGroupActivations` (`GET /action-groups/{actionGroupId}/activations`), or `getActivations` (`GET /activations`) across campaigns.
7. **Approve or reject** — `approveApprovalStatus` (`PUT /v1/activations/{activationId}/approveApprovalStatus`) or `rejectApprovalStatus` (`PUT /v1/activations/{activationId}/rejectApprovalStatus`). A rejection is reversible with `unrejectApprovalStatus` (`PUT /v1/activations/{activationId}/unrejectApprovalStatus`).

## Errors
- `412` `Invalid custom status id.` — the status id you sent does not belong to this action group's custom status set. Read the valid ones from the `ActionGroupCustomStatuses` operations first.
- `409 Conflict` — the object is already in a state that forbids the transition.
- See `errors/later-problem-types.yml`.

## Notes
- Approval is the gate in front of payment. Once an activation is approved, the reward-win flow in `later-fulfill-creator-rewards.md` can pay against it — so treat approve/reject as a money-moving action.
- There is no idempotency key. Approving twice is a second write with no protection; check `getActivationHistory` (`GET /v1/activation/history`) if a call's outcome is uncertain.
- No webhook tells you a creator submitted content. Poll the activation and draft lists.
