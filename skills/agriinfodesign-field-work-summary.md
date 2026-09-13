---
name: agribus-field-work-summary
description: >-
  Summarise the work done on a farmer's fields in AgriBus - list the cultivated land parcels, count
  the passes recorded on each, and query the work records by date range, operation type or area.
api: AgriBus Datastore API
host: https://datastore.agribus-connect.net
operations:
  - listFieldItemsForPage
  - countTasksInField
  - listWithFilter
  - getTaskInfo
  - archiveTsv
generated: '2026-09-12'
method: generated
source: >-
  Grounded in openapi/agriinfodesign-datastore-openapi.yml, harvested from
  https://datastore.agribus-connect.net/v3/api-docs. Every operationId, path, parameter and schema
  field named below is quoted from that document. Authored by API Evangelist; Agri Info Design
  publishes no skills or documentation.
---

# Summarise field work in AgriBus

## What this does

Builds a per-field picture of what a tractor actually did: how many recorded passes a field has, and
the detail of each pass — worked area, distance, average speed, overlap, operation type and the
GNSS source that produced it.

## Before you start

`Authorization: Bearer <JWT>` on every call, issued by `auth.agribus-connect.net`.

**A retention caveat that will silently change your answer:** on the free plan, work data is kept
for **two days**. Indefinite retention is a paid (Plus) feature. Nothing in the API tells you which
plan the account is on, so an empty result may mean "no work" or "work that has already aged out".

## Steps

1. **List the fields.**

   `GET /v1/field_items/page` — operationId `listFieldItemsForPage`.

   Query parameters: `page` (default `0`), `pageSize` (default `20`), `sortKey`, `sortDirection`.

   Response envelope is `PagingResponseRemoteFieldItem`:
   - `contents[]` — the `RemoteFieldItem` rows (`id`, `name`, `caption`, `area`, `length`,
     `geoJson`, `centroid`, `bounds`, `geohash`, `deleted`)
   - `paginationInfo` — `number`, `numberOfElements`, `size`, `totalElements`, `totalPages`

   Note this endpoint includes fields belonging to users the account follows, not just its own.

   Use `listFieldsAround` (`GET /v1/agribus_navi/field_items/around`, parameters `lat`, `lon`,
   `radius` in metres defaulting to `3000`, `includeFollowee` defaulting to `true`) when you have a
   position rather than a field id.

2. **Count the passes per field.**

   `GET /v1/field_items/{id}/tasks/count` — operationId `countTasksInField`. Returns
   `CountTaskInFieldResponse`. Cheap; call it before pulling records.

3. **Query the work records.**

   `POST /v1/task_record_items` — operationId `listWithFilter`. **This is a POST that reads** — the
   filter is the request body, so it is not cacheable and not safe to retry blindly if you ever turn
   it into part of a write sequence.

   Query parameters: `page` (default `0`), `pageSize` (default `20`).

   Body is a `TaskFilter`, and both fields are required:
   - `dateTimeRange` — a `DateTimeRange`
   - `queries[]` — a discriminated array (`type` is the discriminator) of `Account` (`uuids[]`),
     `Field` (`fieldItemIds[]`), `Area`, `Distance`, or `OperationType` (`values[]`)

   The contract states the filter is applied under a fixed `deleted=false` condition.

   Response is `PagingResponseRemoteTaskRecordItem`. Each `RemoteTaskRecordItem` carries
   `startedAt`/`endedAt`, `gnssSource`, `workWidthOnStarted`/`workWidthOnEnded`, `workedArea`,
   `distance`, `avgSpeed`, `complete`, `fieldName`, and `operationType` from the enum
   `Tilling | Sowing | Planting | Fertilizing | CropProtection | Harvesting | Other`.

4. **Detail one group.** `GET /v1/task_record_items/info/{taskGroupId}` — operationId `getTaskInfo`.

5. **Export.** `GET /v1/task_record_items/archives/tsv/{filterName}` — operationId `archiveTsv` —
   returns TSV for a saved filter. Save a filter first with
   `POST /v1/task_record_items/filters` (`saveTaskFilter`).

## Failure handling

The contract declares only HTTP 200. Observed live: `403` with
`{"code":"IAM-…","message":"Permission is invalid"}` when unauthenticated, `404` with the Spring
default envelope on an unknown path. No `WWW-Authenticate`, no problem+json, no rate-limit headers.

## Watch out for

- **Duplicate list routes.** `/v1/field_items` and `/v1/field_items/` are both published, with
  different operationIds (`listAllFieldItems_1` and `listAllFieldItems`). Same for
  `/v1/ref_line_items` and `/v1/plantings`. Prefer the paged `/page` variant.
- **Unbounded lists.** Most list operations here take no paging parameters and return everything.
- **Timestamps.** Every date property in this contract declares
  `"format": "2016-01-01T00:00:00.000Z"` instead of `date-time`. The values are ISO 8601 strings;
  your generated client will type them as unformatted strings.
- **Never call `/v1/_errors/duplication/…` or `/v1/_connect/admin/…`.** They are published in the
  contract but they are internal data-repair and admin routes, including one that copies data
  between the platform's two datastores.
