---
title: Introduction
description: Understand what BranchRunner controls, what runs on workers, and where people make decisions.
group: Start here
order: 10
---
## What BranchRunner is
BranchRunner is a development control plane for turning specified work into reviewed changes. It coordinates tasks, isolated workspaces, agent roles, validation, evidence, and explicit approvals.

It does not replace your Git host, CI system, model provider, or engineering judgment. It connects those systems through workflows whose state remains inspectable.

## Core model
- A **task** describes the requested outcome and requirements.
- A **run** is one attempt to deliver that task.
- A **workspace** isolates the checkout and services used by a run.
- A **workflow** assigns roles, tools, checks, budgets, and approval boundaries.
- **Evidence** connects tests, reviews, and decisions to requirements.

## Choose a path
Install a local deployment with the [quickstart](/docs/quickstart), understand [tasks and runs](/docs/tasks-and-runs), or read [contributing](/docs/contributing) before fixing an issue.

