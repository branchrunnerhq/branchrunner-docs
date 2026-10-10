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
console shows live progress, lets you interrupt, redirect or cancel the agent,
and links the draft pull request once it exists.

## Retry without erasing history

Start a new run after a failed or rejected attempt. The prior run stays
available, so reviewers can compare attempts and see why the next one exists.
