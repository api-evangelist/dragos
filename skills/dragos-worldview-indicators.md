---
name: Pull OT threat indicators from Dragos WorldView
description: Authenticate to the Dragos WorldView API and retrieve OT/ICS indicators of compromise and intelligence products, exporting as JSON, STIX 2.0, or CSV, with correct pagination and rate-limit handling.
api: openapi/dragos-worldview-openapi-original.json
operations:
- GET /api/v1/indicators
- GET /api/v1/indicators.stix2
- GET /api/v1/products
- GET /api/v1/products/{id}
- GET /api/v1/products/{id}/stix2
- GET /api/v1/tags
---

# Pull OT threat indicators from Dragos WorldView

Use this skill to retrieve threat intelligence (indicators of compromise and reports) from Dragos WorldView for OT/ICS defense.

## Authentication
- Base host: `https://portal.dragos.com` (mirror: `https://intel.dragos.com`).
- Generate API credentials on the User Profile page of `portal.dragos.com`.
- Send them on every request as headers: `API-Token: <token>` and `API-Secret: <secret>`.
- A `401` means missing/invalid credentials; a `403` means the account tier lacks entitlement (e.g. trial users cannot call the cached 12-month bundle).

## Steps
1. **List recent indicators.** Call `GET /api/v1/indicators` with `updated_after=YYYY-mm-dd` and `page_size` (default 500, max 1000). Filter with `type` (`domain|filename|hostname|ip|md5|sha1|sha256`), `value`, `serial[]`, or `tags[]`. Page with `page`.
2. **Export as STIX 2.0.** For a machine-ingestible bundle use `GET /api/v1/indicators.stix2` (same filters) or `GET /api/v1/indicators/stix2` for the cached last-12-months bundle (not available to trial users).
3. **Browse reports.** Call `GET /api/v1/products` (filter with `released_after`, `updated_after`, `serials[]`, `indicator`) to find intelligence products.
4. **Fetch a report's indicators.** For a product serial, call `GET /api/v1/products/{id}` for metadata, then `GET /api/v1/products/{id}/stix2` or `.../csv` for its indicators.
5. **Discover tags.** Call `GET /api/v1/tags` (optionally `tag_type`) to enumerate classification tags for filtering.

## Conventions and safety
- Read-only API (GET only) — no writes, no idempotency key needed.
- Respect rate limits: on `429`, back off. Dragos recommends running no more than once every 6 hours; stagger jobs on a random minute.
- Paginate fully via `page`/`page_size`; do not exceed `page_size=1000`.
- See `conventions/dragos-conventions.yml` and `errors/dragos-problem-types.yml` for the full request/response contract.
