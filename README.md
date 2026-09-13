# Agri Info Design

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **Not from the company, and here with a question?** You are welcome here — we would rather be the
> front line and point you the right way than have a good report go nowhere. What this repository
> can answer is narrow, though, so it is worth knowing who you are actually looking for:
>
> - **A question about how the API works, an account, billing, or a bug in the service** — that is
>   the company's own support, not us. We profile this API; we do not operate it and cannot see
>   your account.
> - **A bug in an open-source project we only catalog** — file it on that project's own repository.
>   This has happened with a real and correct bug report that reached us instead of the people who
>   could fix it, which helped nobody.
> - **Anything about this listing itself** — the description, the tags, the rating, a missing or
>   wrong artifact — is ours. Open an issue here.
> - **Not sure, or something general about API Evangelist or APIs.io** — open an issue on the
>   [APIs.io Inbox](https://github.com/api-search/inbox) and we will route it.
>
> **This repository contains no software, and we will never ask you to download anything.** There is
> no build, release, installer, or binary here — only text and machine-readable API descriptions, so
> there is nothing here that can be "corrupt" or need "repairing". Any issue, comment, or email
> claiming otherwise and offering a download link is not from us and is hostile. Do not follow the
> link; it is a lure. Report it to GitHub and, if you like, tell us at
> [info@apievangelist.com](mailto:info@apievangelist.com) so we can take it down.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

Agri Info Design, Ltd. (株式会社農業情報設計社) is a Japanese agricultural-technology company founded
21 April 2014 in Obihiro, Hokkaido, with a Tokyo office in Nihonbashi. It builds the **AgriBus**
precision-farming line: **AgriBus-NAVI**, an Android GPS/GNSS guidance app for tractors with more
than 100,000 downloads worldwide; **AgriBus-GMiniR** and **AgriBus-G2** RTK-GNSS receivers; the
**AgriBus-AutoSteer** automatic steering package; and **AgriBus-Web**, a browser console for field
boundaries, guidance lines, work-record history, elevation maps and RTK base-station management. The
company also sells ISOBUS / ISO 11783 / AG-PORT consulting.

## What this profile found

The AgriBus cloud platform is a set of Spring Boot microservices on `agribus-connect.net`, and each
one **publishes a machine-readable contract anonymously** at the framework's default path. Four
contracts, 186 operations, harvested verbatim into `openapi/`:

| Service | Contract | Size |
|---|---|---|
| [datastore](https://datastore.agribus-connect.net/v3/api-docs) | OpenAPI 3.0.1 | 50 paths / 61 operations |
| [auth](https://auth.agribus-connect.net/v3/api-docs) | OpenAPI 3.0.1 | 20 paths / 22 operations |
| [manager](https://manager.agribus-connect.net/swagger-resources) | Swagger 2.0 | 71 paths |
| [pay](https://pay.agribus-connect.net/v3/api-docs) | OpenAPI 3.0.1 | 21 paths / 22 operations |

The datastore contract **declares a domain standard in its own tag metadata** — `農機オープンAPI
v2.0.0`, the Japanese cross-vendor Agricultural Machinery Open API coordinated by NARO under MAFF's
data-infrastructure programme — and implements its device and location surface, returning machine
work-position history as RFC 7946 GeoJSON. The auth service backs it with an OAuth 2.0 surface and a
**live, anonymously readable catalogue of ten `noki.*` scopes**.

## And what it did not find

There is no developer program around any of it. No developer portal, no API reference, no
getting-started guide, no authentication documentation, no SDK in any language, no CLI, no sandbox,
no Postman collection, no status page, no changelog, no published rate limits, no error catalogue,
no MCP server, no agent card, and no documented way for a third party to register an OAuth client.
Not one of the 186 published operations declares a single non-200 response.

Everything in this repository was therefore established by reading the contracts the platform serves
and by calling the live hosts without credentials. Notable findings are recorded in place:

- `conformance/` — the 農機オープンAPI domain-standard signature, and an OIDC discovery document
  served at `/.well-known/openid_configuration` (underscore) that **404s at the standard hyphenated
  path** and advertises a *development* issuer from the production host.
- `errors/` — the two vendor error envelopes, probed live, including a billing endpoint that answers
  a missing credential with **HTTP 500**.
- `lifecycle/` — 13 operations flagged deprecated with no sunset date, no replacement and no RFC 8594
  headers.
- `conventions/` — `idempotency.coverage: none` across a surface that includes money-moving writes,
  and a `reversibility` grade of `documented` with no stated window anywhere.
- `data-model/` — the entity graph, plus spec-hygiene findings: every timestamp declares
  `"format": "2016-01-01T00:00:00.000Z"`, and the auth contract publishes 13 Spring framework
  internals as schemas.
- `skills/` — three Agent Skills grounded operation-by-operation in the provider's own contracts.

Full machine-readable index: [`apis.yml`](apis.yml).
