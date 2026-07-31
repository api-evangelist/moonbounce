---
name: Run a batch content-moderation job
description: Bulk-evaluate a CSV of text or image content through a Moonbounce policy using the Batch API (presigned upload/download).
api: openapi/moonbounce-openapi-original.json
operations: [GatewayService_CreateBatchJob, GatewayService_GetBatchJobs, GatewayService_CancelBatchJob]
---

# Run a batch content-moderation job

Bulk moderation is a three-step, presigned-URL flow.

## Auth
- `Authorization: Bearer <API_KEY>` on every call (see `authentication/moonbounce-authentication.yml`).

## Steps
1. **Create the batch job** — `GatewayService_CreateBatchJob` (`POST /v1/batch-jobs`). The response returns a `batchJobId` and a **presigned upload URL** (valid ~4 hours).
2. **Upload the input CSV** — PUT your file to the presigned URL. The CSV columns are `ref_id, type, content` where `type` is `text` or `image_url` (image URLs must be publicly accessible). One file per job; a second upload overwrites the first. Processing starts automatically after upload.
3. **Poll for status** — `GatewayService_GetBatchJobs` (`GET /v1/batch-jobs`, filter by `batchJobIds` and/or `states`). When the state is `BATCH_JOB_STATE_COMPLETED`, the record includes a presigned `output_url` (valid ~4 hours; each poll of a completed job returns a fresh URL) — download it to get results.
4. **Cancel if needed** — `GatewayService_CancelBatchJob` (`POST /v1/batch-jobs/{batchJobId}/cancel`) stops further processing.

## Errors
- Domain batch errors surface as `v1BatchJobError.code`: `BATCH_JOB_ERROR_INVALID_INPUT_FILE`, `BATCH_JOB_ERROR_INVALID_POLICY`, `BATCH_JOB_ERROR_INVALID_POLICY_VERSION`, `BATCH_JOB_ERROR_INVALID_PRIORITY`.
- HTTP errors use the gRPC status envelope; `429` = rate/quota exceeded. See `errors/moonbounce-problem-types.yml`.
