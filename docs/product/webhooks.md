---
title: Webhooks
description: What webhook support BranchRunner 1.0.0 has, and what is not yet a public contract.
group: Integrate
order: 120
---
## Availability

BranchRunner 1.0.0 has no public outbound webhook contract. Internal event
names and payloads are not a compatibility promise. Do not build integrations
that depend on them.

## GitHub webhooks

The installation receives `push` and `pull_request` webhooks from its own GitHub
App to keep branches and pull requests reconciled. Repository grants do
not depend on webhooks: BranchRunner rechecks the App's repository access every
4 minutes. The SCM service sets the App's webhook secret itself and verifies
each delivery. You do not configure these webhooks by hand.
