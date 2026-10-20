---
title: Tasks and runs
description: Separate the requested work from each execution attempt and its evidence.
group: Operating model
order: 90
---
## Tasks

A task owns the requested outcome, numbered requirements, repository, workflow
and accountable person. Create a task from the organization's Work list or a
repository, or import a GitHub issue. A task keeps one durable conversation;
clarify assumptions there before a run writes code.

## Run preflight

Before a run starts, the task's **Workflow and run preflight** panel shows the
readiness checks and the review requirement: every 1.0.0 run needs a reviewer
agent from a different provider and model family than every Implement step,
and a human technical review before publication. See
[review requirements for every run](/docs/validation-and-approvals#review-requirements-for-every-run).

Each check reads **Ready**, **Blocked** with its reason, **Checking** while it
runs, or **Starting**:

- **Starting** appears after a deploy, restart or upgrade while the protected
  Codex or Claude route attests the host, publishes its signed boot or binds to
  the restarted worker. The row names that state, for example "Starting — the
  Claude native service is publishing its signed boot". The page reads the
  preflight again every 5 seconds for up to 10 minutes, and the row turns
  **Ready** by itself. A route that is broken reads **Blocked** with its cause
  instead. A run started while a row reads **Starting** is refused with the
  named cause. See [run preflight](/docs/readiness-reasons#run-preflight) for
  each state.
- The managed Codex login row stays **Ready** while the login's latest
  verification on this host is less than an hour old and still current,
  whoever is using the account. Refreshing the preflight does not verify it
  again. When that verification no longer stands, the preflight starts one
  verification and shows "Verifying the managed Codex login…", or names what
  the verification waits for, such as a running turn. Starting the run checks
  the login again with its own, stricter rule.

## Runs

A run is one execution attempt through the six workflow stages. It records
workspace identity, stage attempts and the provider connection each stage used,
tool decisions, validation, review findings, cost and delivery status. The run
console shows live progress and links the pull request once it exists.

The top of the run page shows the run's own workflow stages in their published
order: Plan, each Implement step, Validate, Independent review, Approval and
Handoff. A workflow with several Implement steps names each one
`Implement · <node ID>`. Each stage shows the state its durable stage records
establish, such as **Passed**, **Awaiting Plan decision**, **Awaiting
admission**, **Running · attempt 2**, **Awaiting completion decision**,
**Failed** with its reason, **Outcome unknown**, **Stopped · run cancelled** or
**Not reached**. The current stage is highlighted. Select a stage to see its
node ID, attempt and source, worker, measured turn duration, retained reason and
exact state. A cancelled run keeps the outcomes it reached before the
cancellation.

Every other place that names a run's stage reads the same records: the
organization and repository run lists, active runs on the home page, the
repository overview and assistant, run comparison, the task workbench's
attempts and **Stage**, and the work item page. They name the stage or stages
the run is at,
otherwise the last stage it reached, or "Not started". A run whose stage
records cannot be read exactly reads "Stage unavailable" there; the lists still
load. **Elapsed** is the wall-clock time from the run's start to now while it
is open, or to its end once it finished, failed or was cancelled.

The run's workspace page names the stages that ran in it, for example
"Provisioned and idle. Its run’s Plan, Implement and Validate stages ran in
it.", or "In use by its run’s Validate stage." while a stage runs there.

The run console's controls depend on the stage:

- **Claude session approval.** A stage that runs on a Claude subscription
  waits until its session is requested and approved on the run page. See
  [Claude session approval](/docs/agent-providers#claude-session-approval).
- **Redirect a Codex subscription turn.** While a Codex subscription turn runs,
  you can steer it with a **Steering instruction**. Other stages accept no
  input while they run, and no turn can be interrupted midway.
- **Output.** Codex subscription and API-key stages show live output. A Claude
  subscription stage's result appears when the stage settles.
- **Pause** takes effect at the next stage boundary, after the running turn
  settles. **Resume** continues the run.
- **Cancel run** stops the whole run in any stage, including while Validate
  runs. A single stage cannot be cancelled on its own.

## Retry without erasing history

Unless [automatic failover](/docs/agent-providers#automatic-failover) moves it
to another connection, a stage that failed or stopped waits for a person.
Retry it in place under
**Retry stopped stages** on the run page, with a reason and a recent
authenticator sign-in; the retry keeps the published workflow, the exact source
and the original attempt. For example, when the last Implement step on a Codex
subscription changes no files, the stage reads "Failed · the step changed no
files" instead of passing an empty change to Validate. A Claude stage whose
session start was refused before any session ran fails with a reason that
starts with `workflow_claude_start_failed_`, and can be retried the same way.

Or start a new run after a failed or rejected attempt. The prior run stays
available, so reviewers can compare attempts and see why the next one exists.
