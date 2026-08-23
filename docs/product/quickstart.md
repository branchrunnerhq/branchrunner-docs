---
title: Quickstart
description: Prepare a self-hosted BranchRunner deployment and run the first controlled task.
group: Start here
order: 20
---
## Before you start
You need Docker with Compose, Git, and credentials for at least one supported agent provider. Repository access is configured during setup; no public clone command is shown until the official repository URL is configured.

## Start the deployment
Use the installation instructions distributed with your BranchRunner source package, then verify that the control plane and one runtime worker report healthy.

```text
control plane  healthy
runtime worker ready
```

## Run a task
1. Connect a repository with read and write access appropriate to the workflow.
2. Configure an agent provider and map models to roles.
3. Create a task with numbered requirements and an owner.
4. Start a run, review its proposed scope, and approve implementation.
5. Inspect evidence before approving delivery.

Next, read [workspace environments](/docs/workspace-environments) and [validation and approvals](/docs/validation-and-approvals).

