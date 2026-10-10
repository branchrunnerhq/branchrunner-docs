---
title: MCP and tools
description: What agent profiles may request for tools and MCP servers in this release, and how approvals and network policy bound them.
group: Configuration
order: 60
---
## Tools by agent

An agent profile requests the tool classes its agent may use. Installed platform
limits always apply on top of that request. Give each agent only what its stage
needs: a planner can read repository context, an implementer can edit and run
approved commands, and a reviewer can stay read-only.

## MCP servers and skills

Only installed, reviewed capabilities can be selected; arbitrary MCP endpoints
cannot be added. In 1.0.0, runs deliver an agent's prompt, instructions, model,
and time and token limits. Skill assets and installed MCP servers are not
delivered to runs yet, so a profile that selects them cannot run.

## Destructive actions

BranchRunner has no merge or deploy capability. Externally visible repository
changes happen only through a draft pull request after human approval in the
completion review. A denied request is evidence: the run records the request,
the decision and the next action.

## Network access

Protected agent containers reach their provider through BranchRunner's egress
controls. Validation checks run without network access; their npm dependencies
are downloaded beforehand by the worker, only from `registry.npmjs.org` and only
as pinned in the lockfile.
