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
reason**.

The reviewer can be a signed-in Codex or Claude subscription or an API-key
connection. The workflow's Review stage runs a published profile with the
reviewer role, and that profile decides who reviews. For a subscription
reviewer, the review evidence records the connection, the model and both
reviewer sessions. See
[review independence](/docs/agent-providers#review-independence) for how the
review signing policy is derived from the reviewer profiles and for the
independence rules.

## Review requirements for every run

Every run in BranchRunner 1.0.0 needs two reviews before its change can be
published:

- **A reviewer agent from a second provider.** Every change is reviewed as
  Critical risk until it has been semantically classified, and 1.0.0 does not
  classify changes, so every change is Critical. A Critical review needs a
  reviewer from a different provider and model family than every Implement step:
  if the implementer runs on Codex (OpenAI), the reviewer runs on Claude
  (Anthropic), or the other way round. Task readiness and run start refuse a
  workflow whose reviewer does not meet this, and an installation with
  connections from only one provider can never pass Review.
- **A human technical review.** A qualified person who did not contribute to
  the change reviews it in the completion review before publication. An owner or
  administrator qualifies reviewers.

The workflow editor's Review stage and the task's run preflight state this
requirement before you publish a workflow or start a run.

## Signed evidence

The installation signs completion, evidence and review records with keys
created on your server. By default the installer approves and activates signing
itself after owner setup; the approval is recorded as made by the installer, not
by the owner. Review signing follows once a published reviewer profile can
review, and after a rebuild changes a signed fact, such as the reviewer image,
the installation prepares and approves the replacement. With
`--signing-approval owner`, the installation owner approves evidence signing
under **Runtime workers**, and the review signing proposal and any replacement
on **Home**, instead.

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

Publication cancellation is supported but not yet live-verified; see
[release status](/docs/release-status).
