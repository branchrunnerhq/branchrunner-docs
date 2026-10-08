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

The initial release supports Codex/OpenAI and Claude Code/Anthropic, subject to
live acceptance. Other adapters shown in product concepts are outside that
offering. A saved Codex subscription login records authentication state; it does
not prove a current account entitlement, runtime execution authority, or a
successful agent step. Verify those conditions against the accepted release and
configured worker before starting a run. See [release status](/docs/release-status).

## Supported connections
- **Codex with an API key** can target OpenAI, an OpenAI-compatible Responses
  endpoint (for example DeepSeek) or Azure OpenAI. Each key revision records
  its service, endpoint and exact model or deployment. Through the API, a
  direct run can select one exact revision, which is frozen into the run; the
  UI has no control for that yet. The target check contacts
  the endpoint and reports whether Responses access is verified, expired,
  refused or failed.
- **Codex with a subscription login** uses a managed, encrypted login.
- **Claude Code with an API key** uses the Anthropic Messages API.
- **Claude Code with a subscription login** is on hold pending an owner
  decision: either an official CLI-managed persistent home directory or written
  authorization from Anthropic.

All four connections still need live acceptance.

## Review independence
An independent review role should not inherit the implementing session's hidden context. It evaluates the diff and evidence against the stated requirements.
