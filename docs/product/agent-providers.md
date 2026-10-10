---
title: Agent providers
description: Connect Codex and Claude subscriptions or API keys, order them per agent, and read plan usage, failover and cost.
group: Configuration
order: 50
---
## Supported agents

BranchRunner 1.0.0 supports two coding agents: Codex (OpenAI) and Claude Code
(Anthropic). Other adapters shown in product concepts are outside this release.
Every connection type below still needs live release acceptance; see
[release status](/docs/release-status).

## Connections

Add connections under **Integrations**.

- **Codex with a subscription** signs in with a Codex device code. BranchRunner
  keeps the login encrypted and runs Codex with it in protected containers once
  the installation owner has selected protected Codex execution on the server.
- **Claude Code with a subscription** signs in through the official Claude Code
  CLI: open the Claude sign-in link, approve access, then paste the code Claude
  shows. BranchRunner runs that official CLI with your login inside its
  protected containers, as Anthropic's
  [Agent SDK and Claude plan article](https://support.claude.com/en/articles/15036540-use-the-claude-agent-sdk-with-your-claude-plan)
  describes. Usage draws from your Claude plan's limits. The owner must first
  select protected Claude execution on the server.
- **Codex with an API key** can target OpenAI, an OpenAI-compatible Responses
  endpoint (for example DeepSeek) or Azure OpenAI. Each key revision records its
  service, endpoint and exact model or deployment. The target check contacts the
  endpoint and reports whether Responses access is verified, expired, refused or
  failed. A direct run can be pinned to one exact revision.
- **Claude Code with an API key** uses the Anthropic Messages API.

A saved login records authentication state. It does not prove a current
account entitlement, execution authority on a worker, or a successful agent
step. The run-start check verifies those before a run starts.

## Agent profiles and connection priority

An agent profile under **Coding agents** names a coding agent and sets its
prompt, AGENTS.md instructions, model, and time and token limits. Workflows use
an exact published version of the profile.

A profile can list up to six provider connections of the same agent in priority
order. Reorder them by drag and drop or with the **Move up** and **Move down**
buttons. Each entry has its own exact model and shows whether it is a
subscription or a paid API key. Put subscriptions first and API keys last, so
paid tokens are used only when the subscriptions are exhausted.

Two routing modes decide which connection a stage uses:

- **Priority** takes the first eligible connection in list order.
- **Balance** takes the least-used subscription and uses an API key only when no
  subscription is eligible.

A connection is skipped when it is archived, its login is no longer usable, the
installation cannot run its route, or any of its plan windows is at or above the
profile's usage threshold (50–100 %; the editor starts at 90 %). The choice is made at the start of
each stage and recorded with that stage. When no connection is eligible, the
stage waits and the attention queue shows **Provider unavailable** with the
earliest reset time.

## Plan usage

Subscription cards under **Integrations** show the plan-limit usage the provider
reports, for example the five-hour and weekly windows and their reset times. A
connection without a measurement shows "not measured yet", never 0 %. Claude
reports usage only after a warning threshold is passed.

## Automatic failover

When a stage fails because its connection hit a plan limit, a rate limit, its
quota or an unusable login, and the failed attempt changed nothing in the
workspace, the stage moves to the next eligible connection in the list. Each
connection is tried at most once per stage, and the run console shows the
switch and its cause. Other failures wait for a person to retry. A list cannot
mix Codex and Claude connections.

## Theoretical and effective cost

Each run and each connection shows two figures:

- **Theoretical cost** is what the measured tokens would cost at the provider's
  API list price when the session ran.
- **Effective cost** is what the work actually cost: an API key pays list price
  minus your discount, and a subscription's monthly fee is shared across that
  month's sessions.

Record a subscription's monthly fee or an API key's discount on the connection.
A model without a list price shows "list price not set" and is left out of
totals, which are then marked incomplete. When the provider reports no cache
detail, the figure is marked as an upper bound. Budget enforcement counts API
charges only, not subscription fees.

## Review independence

An independent reviewer does not inherit the implementing session's context.
It evaluates the diff and evidence against the stated requirements. By default,
review runs through the installation's oldest OpenAI API-key connection, and
every OpenAI model counts as one model family. To have high-risk work written by
an OpenAI model reviewed independently, choose a review policy with a reviewer
from another provider under **Runtime workers**.
