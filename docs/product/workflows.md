---
title: Workflows
description: Configure the six workflow stages, publish an exact version, and bind it to repository work.
group: Operating model
order: 80
---
## Six stages

A BranchRunner workflow runs six stages in order:

1. **Plan** proposes how the requirements will be met. A person approves the
   plan before implementation starts.
2. **Implement** makes the change in the run's workspace.
3. **Validate** runs the repository's checks in a hardened validation container
   and returns failures to implementation for repair.
4. **Independent review** evaluates the change against the requirements without
   the implementing session's context. In 1.0.0 every run needs a reviewer from
   a different provider and model family than every Implement step, plus a
   human technical review; see
   [review requirements for every run](/docs/validation-and-approvals#review-requirements-for-every-run).
5. **Approval** is the human decision in the completion review, bound to the
   sealed evidence.
6. **Handoff** opens a draft pull request through the installation's GitHub App.
   BranchRunner never merges or deploys.

## Configure and publish

The workflow configuration page shows all six stages on one page. Each step
selects an exact published agent profile version and can override its model,
prompt and AGENTS.md instructions. Organization policy can mark settings as
enforced, and the page shows which values each step inherits. A new installation
offers a starter template with current models.

A saved draft is configuration, not dispatch authority. Publish the revision,
bind it to the repository, and make sure the worker, source import and provider
connections are ready before expecting stage progression. Earlier published
versions stay available to compare and restore.

The Validate checklist, the handoff selector and the branch prefix are not
configurable yet.

## Design sequence

1. Name the starting condition and expected artifact.
2. Define numbered requirements before implementation.
3. Give each agent the minimum tools it needs.
4. Let validation cover the requirements.
5. Keep approval boundaries for scope, sensitive actions and delivery.
6. Set time, token and cost limits with named owners.

[Release status](/docs/release-status) lists the configurations on which the
complete controlled delivery path was verified live. Browse examples in
[use cases](https://branchrunner.com/use-cases).
