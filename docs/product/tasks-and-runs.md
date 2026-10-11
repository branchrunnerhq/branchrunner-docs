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

## Runs

A run is one execution attempt through the six workflow stages. It records
workspace identity, stage attempts and the provider connection each stage used,
tool decisions, validation, review findings, cost and delivery status. The run
console shows live progress and links the draft pull request once it exists.
Its controls depend on the stage:

- **Redirect a Codex subscription turn.** While a Codex subscription turn runs,
  you can steer it with a **Steering instruction**. Other stages accept no
  input while they run, and no turn can be interrupted midway.
- **Output.** Codex subscription and API-key stages show live output. A Claude
  subscription stage's result appears when the stage settles.
- **Pause** takes effect at the next stage boundary, after the running turn
  settles. **Resume** continues the run.
- **Cancel run** stops the whole run. A single stage cannot be cancelled on its
  own.

## Retry without erasing history

Start a new run after a failed or rejected attempt. The prior run stays
available, so reviewers can compare attempts and see why the next one exists.
