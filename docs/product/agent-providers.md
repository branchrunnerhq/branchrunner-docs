---
title: Agent providers
description: Map workflow roles to model providers without coupling workflow logic to one vendor.
group: Configuration
order: 50
---
## Roles before models
Workflows refer to roles such as planner, implementer, and reviewer. Administrators map each role to a provider and model, so provider changes do not rewrite workflow definitions.

## Credentials
Store provider credentials as secret references. Values are injected only for authorized steps; reveal and use events belong in the audit stream.

## Review independence
An independent review role should not inherit the implementing session's hidden context. It evaluates the diff and evidence against the stated requirements.

