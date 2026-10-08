---
title: Contributing
description: Prepare a focused, evidence-backed change to BranchRunner documentation or source.
group: Contribute
order: 160
---
## Repository access
The official public repository URL is deployment-configured. When it is available, repository, issue, and edit links appear throughout this site. Until then, do not use clone commands copied from older pages or assume an organization name.

## Before you contribute code
BranchRunner is source-available, not open source. The product is licensed under the Business Source License 1.1, which converts to GPL-3.0-or-later on each version's Change Date; the contracts and schema package is Apache-2.0 so that building a client never inherits the BSL.

The contribution policy requires the Contributor Licence Agreement. The product's
`CLA.md` version 1.0 states it is in force from 2026-09-02 and records pending
counsel review; release review has not resolved that status against the requirement
for counsel-approved intake. Confirm the applicable version and intake status
with the maintainers before submitting a contribution. Existing acceptance
records keep the terms accepted at the time. You do not need to publish a personal email address
to contribute — a GitHub `noreply` identity is fine.

See [licensing](/docs/licensing) for package exceptions and commercial enquiries.

Contributions written with a coding agent are expressly permitted and need no disclosure. What you take on is review of the result.

Security findings do not belong in an issue or a pull request. Email `security@branchrunner.com`, the contact in [security.txt](https://branchrunner.com/.well-known/security.txt). `SECURITY.md` in the product repository explains scope and disclosure.

## Fix an issue
1. Read the issue and confirm its acceptance criteria.
2. Find the closest existing implementation and tests.
3. Make one focused change on a branch.
4. Run the relevant checks and record exact commands.
5. Explain behavior changes, risks, and evidence in the pull request.

## Edit documentation
Public docs live as Markdown under `docs/product/`. Each page displays its exact source path. Once repository access is configured, “Edit this page on GitHub” targets that file and branch directly.
