---
name: agribus-guidance-lines
description: >-
  Work with AgriBus guidance (A-B reference) lines - list them, pull one as GeoJSON, find the ones
  near a position, and understand what the fields on a reference line mean before importing or
  deleting any.
api: AgriBus Datastore API
host: https://datastore.agribus-connect.net
operations:
  - listRefLines
  - getGeoJson
  - listRefLinesAround
  - importRefLines
  - updateRefLineItemMetaData
  - deleteRefLineItem
generated: '2026-09-12'
method: generated
source: >-
  Grounded in openapi/agriinfodesign-datastore-openapi.yml, harvested from
  https://datastore.agribus-connect.net/v3/api-docs. Authored by API Evangelist.
---

# Guidance lines in AgriBus

## What a reference line is

The A-B line a tractor steers along. `RemoteRefLineItem` carries `fieldItemId` (the field it belongs
to), `refLineType`, `guidanceInterval` (the implement-width spacing between passes), `refDirection`,
`length`, `name`, `caption` and the usual `id` / `documentId` / `geoDocumentId` / `deleted` set.

Saving and reusing guidance lines is a **paid (Plus) feature** of the product. An account on the free
plan may have none, and the API will not tell you why.

## Reading

1. `GET /v1/ref_line_items/` — operationId `listRefLines` (the trailing-slash variant is
   `listRefLines_1`). Query parameters `page` (default `0`) and `pageSize` (default `20`); response
   envelope `PagingResponseRemoteRefLineItem` with `contents[]` and `paginationInfo`. Includes lines
   belonging to followed users.
2. `GET /v1/ref_line_items/{id}/geo.json` — operationId `getGeoJson` — one line as GeoJSON.
3. `GET /v1/agribus_navi/refline_items/around` — operationId `listRefLinesAround` — proximity lookup,
   the same shape AgriBus-NAVI uses in the cab.
4. `GET /v1/ref_line_items/archives/tsv` — operationId `archiveTsv_1` — TSV export.

## Writing — read this before you write anything

1. `POST /v1/ref_line_items` — operationId `importRefLines` — takes an `ImportRequest`.
2. `PUT /v1/ref_line_items/{id}/meta` — operationId `updateRefLineItemMetaData` — takes an
   `UpdateMetaDataRequest`.
3. `POST /v1/ref_line_items/sync/metadata` — operationId `waitSyncAsUpdateRefLineItem` — **call this
   after a write.** The platform persists geometry to Firestore asynchronously and exposes an
   explicit client-side wait for the write to settle. If you read back immediately without it, you
   may read stale data.
4. `DELETE /v1/ref_line_items/{id}` — operationId `deleteRefLineItem`.

### The three things that make writes here risky

- **No idempotency.** No `Idempotency-Key` header exists anywhere on this platform. If
  `importRefLines` times out, you cannot safely retry it — you may create duplicates.
- **No dry run.** No preview or validation mode on any write operation.
- **Delete is the only undo, and there is no undo for an edit.** There is no restore, trash or
  version history operation in the contract. `deleteRefLineItem` reverses a create; nothing reverses
  a `updateRefLineItemMetaData`. No retention or recovery window is stated by the provider anywhere,
  so treat every edit as permanent.

## Failure handling

The contract declares only HTTP 200 for every one of these operations. Observed live: `403` with
`{"code":"IAM-…","message":"Permission is invalid"}` on a missing credential; `404` with the Spring
default envelope on an unknown path. Plan for undeclared failures on every call.
