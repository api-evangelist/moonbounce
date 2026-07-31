---
name: Evaluate content against a policy
description: Submit text or image content to Moonbounce (Clavata Public API v1) for real-time evaluation against a moderation policy, then read the outcome and policy evaluation report.
api: openapi/moonbounce-openapi-original.json
operations: [GatewayService_CreateJob, GatewayService_GetJob, GatewayService_Evaluate]
---

# Evaluate content against a policy

Use this to run content through a Moonbounce policy and get a moderation decision.

## Auth
- Every request needs `Authorization: Bearer <API_KEY>`. Create/manage keys in the app under **Account Settings → API Keys** (Admin role only). Keys can expire (30/90/180/365 days or never) and be disabled/revoked.
- Base path is `/v1` on the API gateway.

## Steps
1. **Create the job** — `GatewayService_CreateJob` (`POST /v1/jobs`). Body: `contentData` (the text/image items), `policyId` (the policy to evaluate against), optional `threshold`, optional `expedited`, and an optional `webhook` (`url` + `extraHeaders`) to be called on completion. Set the create option to wait for completion if you want the result inline.
2. **Get the result** — if you did not wait inline, either supply a `webhook` and receive the result on completion, or poll `GatewayService_GetJob` (`GET /v1/jobs/{jobUuid}`) until `status` is `JOB_STATUS_COMPLETED` (or `JOB_STATUS_FAILED` / `JOB_STATUS_CANCELED`).
3. **Read the decision** — each result carries an `outcome` (`OUTCOME_TRUE` = flagged by a policy section, `OUTCOME_FALSE` = not flagged, `OUTCOME_FAILED`) plus a `PolicyEvaluationReport` with per-section/rule detail and token usage.

## Real-time alternative
- For streaming results as content is processed, use `GatewayService_Evaluate` (`POST /v1/jobs/stream`) instead of create + poll.

## Errors & conventions
- Errors use the gRPC status envelope `{ code, message, details }` as `application/json` (not RFC 9457). See `errors/moonbounce-problem-types.yml`.
- `429` = rate/quota exceeded (free plan caps at 1,000 evaluations/day).
- `499` = precheck failure (e.g. `PRECHECK_FAILURE_TYPE_NCMEC` CSAM match, unsupported/invalid image); inspect the precheck failure type and `matchConfidence`.
- No idempotency key is supported — do not assume safe retries create the same job.
