---
name: Run a Quantemplate pipeline and download its output
description: Execute a preconfigured Quantemplate data pipeline for an organisation, then download the resulting dataset as CSV.
api: openapi/quantemplate-fabric-openapi.yml
operations:
  - execute-pipeline
  - download-dataset
generated: '2026-07-20'
method: generated
---

# Straight-Through Processing with the Quantemplate FabricAPI

Automate the Quantemplate data flow: kick off a preconfigured pipeline, wait for
it to succeed, then pull the output dataset. Every step below is grounded in a
real operation in `openapi/quantemplate-fabric-openapi.yml`.

## Authentication
- Send `Authorization: Bearer <JWT>` on every request (scheme `bearerAuth`, `http`/`bearer`, JWT).
- Base URL: `https://api.prod.quantemplate.com/v1`.
- Optionally send `X-QT-Trace-Id: <your-id>` to correlate a request across the platform.

## Steps

1. **Execute the pipeline** — `execute-pipeline`
   `POST /organisations/{organisationId}/pipelines/{pipelineId}/executions`
   - Path params: `organisationId`, `pipelineId`.
   - On `200` you receive a `PipelineExecutionView` — capture `id` (begins `e-...`),
     `runNumber`, and `status` (`Started|Succeeded|Failed|Cancelled`).

2. **Poll until finished**
   - Re-read the execution status until it is `Succeeded` (or handle `Failed`/`Cancelled`).
   - When `Succeeded`, `outputs[]` lists each `PipelineExecutionOutputView` (`id` begins `o-...`, plus `name`).

3. **Download the output dataset** — `download-dataset`
   `GET /organisations/{organisationId}/datasets/{datasetId}`
   - Path params: `organisationId`, `datasetId`.
   - Optional `version` query param selects a specific dataset version.
   - `200` streams `text/csv` (UTF-8). A `204` means no content yet.

## Error handling
- `401` / `403`: text/plain body — refresh/replace the Bearer JWT or check the principal's access to the organisation/pipeline/dataset.
- `404`: JSON `PipelineExecutionOutputNotFound` (`executionId`, `outputId`) — verify the ids and dataset `version`.
- There is no idempotency-key contract; do not blindly retry `execute-pipeline` on a timeout — re-read execution state first.
