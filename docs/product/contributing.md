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

**External contributions are not accepted yet.** The licensor, Savelkouls
Ventures, LLC, does not accept pull requests or patches from outside contributors
until counsel has reviewed the Contributor Licence Agreement. External pull
requests are not reviewed or merged in the meantime. The opening of intake will
be announced in the product's `CONTRIBUTING.md`.

Once intake opens, contributions require the product's `CLA.md`. Version 1.1 names
Savelkouls Ventures, LLC as the party that receives contribution licences. Its
relicensing, moral-rights and governing-law sections are still with counsel, and
a new version will be published before intake opens if the review changes the
terms. You will not need to publish a personal email address to contribute — a
GitHub `noreply` identity is fine.

See [licensing](/docs/licensing) for package exceptions and commercial enquiries.

Contributions written with a coding agent are expressly permitted and need no disclosure. What you take on is review of the result.

Security findings do not belong in an issue or a pull request. Email `security@branchrunner.com`, the contact in [security.txt](https://branchrunner.com/.well-known/security.txt). `SECURITY.md` in the product repository explains scope and disclosure.

## Fix an issue
Once contribution intake opens:

1. Read the issue and confirm its acceptance criteria.
2. Find the closest existing implementation and tests.
3. Make one focused change on a branch.
4. Run the relevant checks and record exact commands.
5. Explain behavior changes, risks, and evidence in the pull request.

## Edit documentation
Public docs live as Markdown under `docs/product/`. Each page displays its exact source path. Once repository access is configured, “Edit this page on GitHub” targets that file and branch directly.
