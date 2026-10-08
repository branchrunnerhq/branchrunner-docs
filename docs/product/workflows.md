---
title: Workflows
description: Define roles, steps, checks, budgets, and approvals as a repeatable delivery process.
group: Operating model
order: 80
---
## Workflow contract
A workflow states which roles participate, what tools they can use, which checks must pass, and where a person must decide. Keep repository-specific setup in the workspace definition.

A saved draft is configuration, not dispatch authority. Publish the accepted
revision, bind it to the repository, and verify worker, source and provider
readiness before expecting stage progression. The complete controlled delivery
path remains part of [release acceptance](/docs/release-status).

## Design sequence
1. Name the starting condition and expected artifact.
2. Define requirements before implementation.
3. Assign the minimum tools to each role.
4. Attach validation to requirements.
5. Add approval boundaries for scope, sensitive actions, and delivery.
6. Set time, token, and cost limits with named owners.

Browse complete examples in [use cases](/use-cases).
