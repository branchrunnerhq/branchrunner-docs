---
title: Introduction
description: Understand what BranchRunner controls, what runs on workers, and where people make decisions.
group: Start here
order: 10
---
## Release scope

This documentation describes BranchRunner Community Edition 1.0.0, the first
self-hosted release. Version 1.0.0 is a release candidate: it has not passed
release acceptance and is not published yet. Read
[release status](/docs/release-status) for what the release includes, its
known limitations and what is still being proven.

## What BranchRunner is

BranchRunner is a development control plane for turning specified work into
reviewed changes. It coordinates tasks, isolated workspaces, coding agents,
validation, independent review, signed evidence and explicit human approvals.
Its only delivery is a draft pull request. It never merges or deploys.

It does not replace your Git host, CI system, model provider or engineering
judgment. It connects them through workflows whose state stays inspectable.

## Core model

- A **task** describes the requested outcome and its numbered requirements.
- A **run** is one attempt to deliver that task.
- A **workspace** isolates the checkout and services one run uses.
- An **agent profile** names a coding agent, its prompt, instructions and limits,
  and an ordered list of provider connections.
- A **workflow** runs six stages: Plan, Implement, Validate, Independent review,
  Approval and Handoff (a draft pull request).
- **Evidence** connects changes, validation and reviews to requirements. It is
  signed by the installation.

## Where things run

The control plane (web UI, API and PostgreSQL) runs on your server. Runtime
workers run repository checkouts, coding agents and validation in hardened
containers, either on the same server or on other hosts you enroll. In 1.0.0,
a worker on another host runs Codex subscription and API-key stages, while
Claude subscription stages run only on the worker on the server. GitHub access
goes through a GitHub App that the installation creates on your own account.

## Choose a path

Install BranchRunner with the [quickstart](/docs/quickstart), understand
[tasks and runs](/docs/tasks-and-runs), or read [security](/docs/security) to
report a vulnerability.
