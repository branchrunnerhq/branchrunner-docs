---
title: Validation and approvals
description: How BranchRunner validates changes, reviews them independently, signs evidence, and opens a draft pull request after human approval.
group: Operating model
order: 100
---
## Evidence states

BranchRunner distinguishes agent-reported results from independently validated
checks and human-approved decisions. A completion statement is not a passing
test. Completion readiness names the exact evidence each requirement is still
missing.

## Validation

Validation runs in a hardened container on the worker, separate from the agent's
workspace, with no network access.

- **Detected suite.** When a repository is imported, BranchRunner reads the
  `build`, `typecheck`, `lint` and `test` scripts in its `package.json`. npm's
  placeholder test script is ignored. Each detected script becomes a required
  check that supplies the matching evidence class.
- **Prepared dependencies.** The worker downloads only the tarballs pinned in
  `package-lock.json` from `registry.npmjs.org` and verifies each one against its
  lockfile integrity hash. It then runs `npm ci --offline --ignore-scripts` in a
  container without network and mounts the result read-only for the checks.
- **Baseline check.** Every repository also runs a repository-hygiene check:
  JSON files must parse and no merge-conflict markers may remain.
- **Owner adjustment.** The repository page's **Validation suite** panel shows
  what was detected, the evidence each check supplies and what is still missing.
  An owner can skip or restore a detected check, with a recorded reason.

Validation supports npm projects only in this release. A repository without a
`package-lock.json`, one that uses npm workspaces, or one in another language
gets only the hygiene check, and completion names every missing evidence class.

## Independent review

A separate reviewer evaluates the change and its evidence against the stated
requirements, without the implementing session's context. Each finding offers
**Accept and repair**, which returns the run to implementation, or **Reject with
reason**. See [agent providers](/docs/agent-providers) for the default review
policy.

## Signed evidence

The installation signs completion, evidence and review records with keys
created on your server. By default the installer approves and activates signing
itself after owner setup; the approval is recorded as made by the installer, not
by the owner. With `--signing-approval owner`, the owner approves each signing
proposal under **Runtime workers** instead.

## Approval records

An approval records who decided, what transition was authorized, what evidence
was visible and when it occurred. Publication is approved in the completion
review, which binds the decision to the sealed evidence. It needs a recent
sign-in with your authenticator. Delivery stays a separate decision from
implementation completion.

## Draft pull requests

Each attached repository has a draft pull request policy. By default drafts are
enabled and may target only the repository's default branch. The repository
page's **Draft pull requests** panel shows the policy and anything that blocks
publication, such as a missing GitHub App permission.

After approval, BranchRunner opens a draft pull request through the GitHub App.
It has no merge or deploy capability. The installer turns trusted publication on
by default; `--no-publication` keeps it off.

## Cancelling a publication

If you cancel a publication, BranchRunner stops further repository-host effects.
The task branch and draft pull request stay unchanged. The completion review
page shows what was left there, with warnings if the branch moved or the pull
request left draft state, and the steps to remove them yourself.

Release acceptance for the complete path to a draft pull request is still in
progress; see [release status](/docs/release-status).
