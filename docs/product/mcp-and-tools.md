---
title: MCP and tools
description: Scope MCP servers and command tools by role, workflow, and approval requirement.
group: Configuration
order: 60
---
## Least privilege by role
Attach tools to the roles that need them. A planner may read repository context while an implementer can edit and run approved commands; a reviewer can remain read-only.

## Destructive actions
Mark destructive or externally visible calls as approval-required. A denied call is evidence: the run records the request, decision, and next action.

## Network access
Begin with required package registries, Git remotes, providers, and internal services. Log denials so policy can be refined from observed needs.

