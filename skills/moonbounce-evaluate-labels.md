---
name: Evaluate content against published labels
description: Score content against one or more published Moonbounce label versions (the label-first experience) in a single call.
api: openapi/moonbounce-openapi-original.json
operations: [LabelsService_EvaluateLabels]
---

# Evaluate content against published labels

Use this on a label-first account to classify content against published label versions.

## Auth
- `Authorization: Bearer <API_KEY>` (see `authentication/moonbounce-authentication.yml`). A label-first account is required for this endpoint.

## Steps
1. **Evaluate** — `LabelsService_EvaluateLabels` (`POST /v1/labels/evaluate`). Body: `contentData` (the item to evaluate), `labelVersionIds` (one or more published label versions to score against), optional `threshold`, optional `labelEvalOptions`, and optional `globalContext`.
2. **Read results** — the response returns **one result per requested label version, in request order** (`v1EvaluateLabelsResponseLabelResult`), each with its evaluation.

## Errors & conventions
- Only `200` and `default` are declared for this operation; `default` carries the gRPC status envelope `{ code, message, details }`.
- No idempotency key is supported. See `conventions/moonbounce-conventions.yml`.
