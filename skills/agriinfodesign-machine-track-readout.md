---
name: agribus-machine-track-readout
description: >-
  Read an agricultural machine's work-position history out of AgriBus over the Japanese
  農機オープンAPI (Agricultural Machinery Open API v2.0.0) device and location surface, and return it
  as GeoJSON.
api: AgriBus Datastore API
host: https://datastore.agribus-connect.net
operations:
  - getDevices
  - getLocationsByDeviceId
generated: '2026-09-12'
method: generated
source: >-
  Grounded in openapi/agriinfodesign-datastore-openapi.yml (harvested from
  https://datastore.agribus-connect.net/v3/api-docs) and the live scope catalogue at
  https://auth.agribus-connect.net/v1/agricultural-api/oauth/scopes. Every operationId, path,
  parameter and scope named below is quoted from those documents. Agri Info Design publishes no
  documentation and no skills of its own; this skill is authored by API Evangelist.
---

# Read a machine's work track from AgriBus

## What this does

Lists the agricultural machines linked to an AgriBus account and pulls one machine's position
history for a time window. The position history comes back as a GeoJSON `FeatureCollection`
(RFC 7946), so it can be mapped or intersected with field boundaries without conversion.

## Before you start

- **Credential.** Every operation needs `Authorization: Bearer <JWT>`. The token is RS256 and is
  issued by `auth.agribus-connect.net`; verify it against
  `https://auth.agribus-connect.net/.well-known/jwks.json`.
- **Scopes.** This surface is governed by `noki.devices.read` and `noki.locations.read` from the
  provider's published scope catalogue. There is **no documented client-registration process**, so
  obtaining an OAuth client for a third-party integration requires contacting the company directly.
- **No documentation exists.** Agri Info Design publishes no developer portal and no API reference.
  The contract is the documentation.

## Steps

1. **List the machines.**

   `GET /v1/agricultural-machinery/devices` — operationId `getDevices`.

   Returns a `DeviceListResponse` of `DeviceInfo`: `device_id` (UUID), `device_name`,
   `manufacturer`, `model`, `serial_number`, `firmware_version`. Note the snake_case — this surface
   follows the Japanese cross-vendor standard, while the rest of the platform is camelCase.

   Takes no parameters. It returns everything linked to the authenticated user, unpaged.

2. **Pull the track for one machine.**

   `GET /v1/agricultural-machinery/devices/{device_id}/locations` — operationId
   `getLocationsByDeviceId`.

   | Parameter | In | Required | Notes |
   |---|---|---|---|
   | `device_id` | path | yes | UUID from step 1 |
   | `since` | query | no | ISO 8601 start |
   | `until` | query | no | ISO 8601 end |
   | `limit` | query | no | max rows |
   | `sort` | query | no | `asc` or `desc` |

   Returns a GeoJSON `FeatureCollection`. **Served as `application/json`, not
   `application/geo+json`** — do not content-negotiate for the GeoJSON media type, you will not get
   it.

3. **Window, do not page.** There is no cursor and no next-page token. To walk a long history,
   advance `since`/`until` yourself using the last feature's timestamp. Combine with `limit` and
   `sort: asc` so you know which end of the window you consumed.

## Failure handling

The contract declares **only HTTP 200** for these operations — no error responses at all. What the
live service actually returns:

| Condition | Status | Body |
|---|---|---|
| No/invalid credential | `403` | `{"code":"IAM-2076068935","message":"Permission is invalid"}` |
| Unknown path | `404` | `{"timestamp":"…","status":404,"error":"Not Found","path":"…"}` |

There is **no `WWW-Authenticate` header**, and a missing credential is `403`, not `401`. Do not
treat `403` as "this account lacks permission" — check for a token first.

## What you must not assume

- **No rate-limit signal.** No `RateLimit-*`, `X-RateLimit-*` or `Retry-After` header is emitted and
  no limit is published. Back off on your own schedule; you will get no warning before a failure.
- **No idempotency.** Irrelevant here (both operations are reads) but relevant the moment you touch
  any write on this platform — there is no `Idempotency-Key` anywhere.
- **Do not use `/poc/devices/{device_id}/locations`.** It is the deprecated proof-of-concept of this
  same endpoint (`getLocationByDeviceId`, tag `PocNokiOpenApi`). It is still routable and returns
  `406` to a default `Accept` header, with no `Deprecation` or `Sunset` header to warn you.
