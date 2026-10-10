---
title: API
description: The status of the BranchRunner HTTP API in the 1.0.0 release.
group: Integrate
order: 110
---
## Availability

BranchRunner 1.0.0 has no stable public API. The web UI uses the installation's
HTTP API, and the product source includes its generated OpenAPI description
(`docs/api/openapi.json`). Those routes are deployment implementation details
and can change between releases without notice. Do not build production
automation against them.

## Contracts package

The `@branchrunner/contracts` package holds the statuses, schemas, projections
and commands that the API and web UI share. It is licensed under Apache-2.0, so
a client built against it does not inherit the Business Source License; see
[licensing](/docs/licensing). It is not published to npm in this release.
