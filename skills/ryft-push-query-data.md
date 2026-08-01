---
name: Push query data to Ryft
description: >-
  Send query-execution telemetry from a custom query engine to Ryft's Ingest
  API so Ryft can build usage-based Apache Iceberg optimizations. Covers auth,
  the payload shape, and bulk ingestion.
api: openapi/ryft-ingest-openapi.yml
operations: [pushQueries]
---

# Push query data to Ryft

Use this skill to report executed queries from any query engine to Ryft.

## Prerequisites
- A **Ryft ingest token** for your environment (provided by Ryft). Keep it secret.
- Outbound HTTPS (port 443) connectivity to `ingest.ryft.io`.

## Auth
Every request authenticates with the token in a header:

```
X-Ryft-Token: <RYFT_INGEST_TOKEN>
```

## Steps

1. **Build a query object.** Required fields: `context` (object; may be `{}`),
   `query` (SQL text), `queryId` (unique), `queryState`
   (`FINISHED` | `SUCCEEDED` | `FAILED`), `timestamp` (ISO 8601), and
   `duration_ms` (number). Optional: `context.catalog`, `context.schema`,
   `context.queryType`, `context.user`, `context.userAgent`, and `tags[]`.
2. **For failed queries** (`queryState: FAILED`) attach `failureInfo` with
   `errorCode.type` of `USER_ERROR` | `INTERNAL_ERROR` |
   `INSUFFICIENT_RESOURCES` | `EXTERNAL`.
3. **POST** the object (operation `pushQueries`) to
   `https://ingest.ryft.io/ingest/push/queries` with
   `Content-Type: application/json`.
4. **Bulk:** to send many at once, POST a **JSON array** of query objects to the
   same endpoint.
5. **Check the response.** A `200` means the message was accepted and will be
   processed.

## Conventions & error handling
- Content type is always `application/json`; transport is HTTPS only.
- There is **no documented idempotency contract** — the server does not promise
  de-duplication on `queryId`, so avoid re-sending the same batch on ambiguous
  failures unless duplicates are acceptable.
- Per-query problems surface inside the payload (`failureInfo`), not as HTTP 4xx.

## Example

```bash
curl -X POST https://ingest.ryft.io/ingest/push/queries \
  -H "X-Ryft-Token: <RYFT_INGEST_TOKEN>" \
  -H "Content-Type: application/json" \
  -d '{
    "context": {"catalog":"iceberg","schema":"gold","queryType":"SELECT"},
    "timestamp": "2026-03-10T12:00:00.000Z",
    "duration_ms": 100,
    "query": "SELECT * FROM gold.my_table",
    "queryId": "test-001",
    "queryState": "FINISHED",
    "tags": ["source:my-engine"]
  }'
```
