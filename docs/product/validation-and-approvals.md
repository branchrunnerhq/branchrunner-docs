---
title: Validation and approvals
description: How BranchRunner validates changes, reviews them independently, signs evidence, and opens a pull request after human approval.
group: Operating model
order: 100
---
## Evidence states

BranchRunner distinguishes agent-reported results from independently validated
checks and human-approved decisions. A completion statement is not a passing
test. Completion readiness names the exact evidence each requirement is still
missing.

## Validation

Validation runs in hardened containers on the worker, separate from the agent's
workspace, with no network access. Every repository runs:

- **Baseline checks.** A repository-hygiene check (JSON files parse and no
  merge-conflict markers remain) and a documentation check: every Markdown and
  reStructuredText file is UTF-8 text, is not empty, has no merge-conflict
  markers, and its relative links and images resolve.
- **Detected suite.** For each supported project the worker finds at the
  repository root, or at a root the repository's declaration names, it proposes
  the checks below. Each becomes a required check that supplies the matching
  evidence class.
- **Dependency security.** A repository with prepared dependencies also runs
  `dependency-audit`. It fails when the change adds a package, or moves one to
  another version, that has a known advisory of high severity or above.

| Ecosystem | Lockfile | Checks |
| --- | --- | --- |
| npm, including workspaces | `package-lock.json` version 2 or 3 | the `build`, `typecheck`, `lint` and `test` scripts |
| pnpm 9 and 10, including workspaces | `pnpm-lock.yaml` version 9.0 | the same scripts |
| Yarn classic, including workspaces | `yarn.lock` version 1 | the same scripts |
| Zero-install Yarn Berry (Yarn 3 and 4, Plug'n'Play) | `yarn.lock` with the committed `.yarn/cache` and `.yarn/releases` | the same scripts |
| Python, wheels only | hashed requirements files, `Pipfile.lock`, `uv.lock` or `poetry.lock` | `pytest` and `ruff` when the lockfile pins them; `mypy` when it is pinned and configured, otherwise a byte-compile check |
| Go modules | `go.mod` and `go.sum` | `go build`, `go vet` and `go test` |

- **Prepared dependencies.** The worker itself downloads every dependency,
  only from admitted hosts and only when it matches the lockfile's hash, and
  installs it in a container without network, with install scripts off. The
  checks then run offline in a writable copy of the source that is discarded
  afterwards. Node, pnpm, Yarn, CPython, uv and Go come from a toolchain catalog
  sealed into the release. An organization owner can admit a private registry
  or mirror under **Settings > Repositories > Validation preparation hosts**;
  registries that need credentials are not supported.
- **Declared checks and check services.** An owner can add up to 24 checks in
  the repository's **Validation suite** panel or in
  `.branchrunner/validation.yml`. A declared check can start up to four
  disposable service containers, such as PostgreSQL, pinned by image digest.
  A declaration applies only after an owner approves its exact contents with a
  recent authenticator sign-in, and a change never validates itself with its
  own edits. Services run on a private network with no internet access and are
  removed after each check.
- **Owner adjustment.** The **Validation suite** panel shows what was
  detected, the evidence each check supplies and what is still missing. An
  owner can turn a detected check off, with a recorded reason.

A project BranchRunner recognises but cannot prepare is listed with its reason
and is never skipped as passed: Cargo, Maven, Gradle, NuGet, Bundler, Composer,
Bun, Deno, `pdm.lock`, Go workspaces (`go.work`), pnpm 11 and later, Yarn Berry
without a committed cache, Python requirements without hashes, and npm
lockfile version 1. Python installs wheels only. A check that downloads
anything at run time fails, and Dockerfiles are not built. A repository with no
supported project runs only the baseline checks, so only a documentation-only
change can complete there, and completion names every missing evidence class.

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

## Secret scan

Before publication, completion readiness scans the run's change for secrets
with the organization's content policy. It reads every path in the final tree
and the content of every file the run added or changed; content the base
commit already held is not scanned again. A file fails the `secret_scan` gate
only when the policy matches it. Long lines, deeply nested or very large JSON and
minified bundles are scanned in overlapping windows instead of being refused.
Content the scanner cannot inspect, for example text in an encoding it refuses,
is reported as unavailable with a `completion_tree_content_<reason>` code and
never passes.

The captured change holds at most 8 MiB per added or changed file and 24 MiB of
such content in total. A larger change cannot be captured and is refused with
`capture_tree_export_blob_too_large` or `capture_tree_export_change_too_large`.

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

## Pull requests

Each attached repository has a pull request policy: whether pull requests are
allowed, whether they open as a draft or ready for review, and which base
branches they may target. Attaching a repository creates the default: pull
requests to the default branch only, opened as drafts, or ready for review for
a private repository owned by a personal GitHub account, where GitHub offers no
drafts. The repository page's **Pull requests** panel shows the policy and
anything that blocks publication, such as a missing GitHub App permission, and
an owner can change it there with a reason.

After approval, BranchRunner opens one pull request of the policy's type
through the GitHub App; the completion review names that type before you
approve. It has no merge or deploy capability. The installer turns trusted
publication on by default; `--no-publication` keeps it off.

## Cancelling a publication

If you cancel a publication, BranchRunner stops further repository-host effects.
The task branch and pull request stay unchanged. The completion review page
shows what was left there, with warnings if the branch moved or a draft pull
request left draft state, and the steps to remove them yourself.

Publication cancellation is supported but not yet live-verified; see
[release status](/docs/release-status).
