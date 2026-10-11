---
title: Agent providers
description: Connect Codex and Claude subscriptions or API keys, order them per agent, and read plan usage, failover and cost.
group: Configuration
order: 50
---
## Supported agents

BranchRunner 1.0.0 supports two coding agents: Codex (OpenAI) and Claude Code
(Anthropic). Other adapters shown in product concepts are outside this release.
[Release status](/docs/release-status) lists which connection types were
verified live; the others are supported but not yet live-verified.

## Connections

Add connections under **Integrations**.

- **Codex with a subscription** signs in with a Codex device code. BranchRunner
  keeps the login encrypted and runs Codex with it in protected containers. The
  installation selects protected Codex execution automatically once the worker
  installer has attested the host.
- **Claude Code with a subscription** signs in through the official Claude Code
  CLI: open the Claude sign-in link, approve access, then paste the code Claude
  shows. BranchRunner runs that official CLI with your login inside its
  protected containers, as Anthropic's
  [Agent SDK and Claude plan article](https://support.claude.com/en/articles/15036540-use-the-claude-agent-sdk-with-your-claude-plan)
  describes. Usage draws from your Claude plan's limits. The installation
  selects protected Claude execution automatically once the worker installer
  has prepared the Claude runtime. In 1.0.0, Claude
  subscription stages run only on the worker on the control-plane host, not on
  a worker on another host; see
  [runtime workers](/docs/runtime-workers#agent-stages-on-another-host).
- **Codex with an API key** can use OpenAI, an OpenAI-compatible service at a
  custom base URL, or Azure OpenAI. See
  [Codex API-key services](#codex-api-key-services) below. A direct run can be
  pinned to one exact key revision.
- **Claude Code with an API key** uses the Anthropic Messages API.

API-key connections run BranchRunner's built-in agent and need no execution
selection. The direct `codex` and `claude` agent-image adapters are retired and
refused with `adapter_retired`; subscriptions run only through the protected
runtimes, and the owner can opt out of those with
`branchrunner-configure-execution.sh none`.

Up to four runs can use one Codex or Claude subscription connection at the same
time. Each run gets its own CLI process and home directory, and a concurrent
session cannot rotate the login: only an exclusive verification refreshes it.
Concurrent sessions are supported but not yet live-verified.

A saved login records authentication state. It does not prove a current
account entitlement, execution authority on a worker, or a successful agent
step. The run-start check verifies those before a run starts.

## Codex API-key services

Choose **Add Codex API key**, then pick the API service. Each service needs
these fields:

| Service | Required fields | What BranchRunner calls |
| --- | --- | --- |
| **OpenAI** | Exact model name | `https://api.openai.com/v1/responses`, key as a Bearer token |
| **OpenAI-compatible (custom base URL)** | API base URL (for example `https://llm.example.com/v1`) and exact model name | The base URL plus `/responses`, key as a Bearer token |
| **Azure OpenAI** | Resource URL (`https://<resource>.openai.azure.com`), exact deployment name, API version `v1` | `/openai/v1/responses` on the resource, key in the `api-key` header, deployment as the model |

The base URL must use HTTPS on port 443 and a public DNS name. It cannot
contain credentials, a query, `..` segments or an operation path such as
`/responses`. The service, endpoint and exact model or deployment are fixed
with the key revision; to change them, save a new key.

**Responses only.** Codex 0.156.1 talks to providers only through the OpenAI
Responses API. A service that offers only Chat Completions cannot run Codex.
DeepSeek and dated Azure API versions (for example `2024-10-21`) are refused
when you save the key. Other Chat Completions-only services are caught by the
check below, which reports that the Responses operation was not found.

### Admitted provider hosts

The provider service contacts an OpenAI-compatible service directly, so its host
is an outbound destination of your installation. Before a key for a custom host
can be saved or checked, an owner of the organization admits that exact host name
under **Integrations → Admitted provider hosts**. This needs a recent sign-in.
There are no wildcards: admitting `llm.example.com` admits only that host.
OpenAI and Azure OpenAI resource endpoints are built in and need no entry.

Saving a key for a host that is not admitted is refused with
`provider_host_not_admitted`. Withdrawing a host stops the provider service from
contacting it from the next call on. Connections that target it stay saved but
read as not ready until the host is admitted again. Every admission and
withdrawal is audited, and the history is kept.

BranchRunner always verifies the provider's TLS certificate. It refuses
redirects and refuses a host whose DNS answer includes a private, loopback or
other special-purpose address.

### Check the connection

After saving the key, open **Exact API target and provider check** on the
connection and choose **Check Responses access**. This sends one small,
authenticated Responses request through the provider service with the protected
key. The request asks for at most 256 output tokens and expects a 64 KiB
response or less, so it may use a little quota. The check passes only when the
service returns a completed Responses result that contains a one-time nonce.

A passing check keeps the key revision usable by runs for 24 hours, and each
successful agent call extends that window. When the window lapses, the next run
start, or a provider manager opening the connection, checks the target again.
**Execution available** appears only after a real agent call has succeeded
through your installation.

A failed check names one of these causes:

| Failure | Meaning |
| --- | --- |
| `unauthorized` | The provider rejected the key (HTTP 401 or 403). |
| `operation_not_found` | The service has no Responses operation at that base URL, or the model or deployment name does not exist (HTTP 404 or 405). |
| `rate_limited` | The provider rate-limited the check (HTTP 429). Try again later. |
| `server_error` | The provider reported a server error (HTTP 5xx). |
| `timeout` | The provider did not answer within the 45-second limit. |
| `invalid_response` | The provider answered, but not with a completed Responses result containing the nonce. |
| `network_refused` | The endpoint could not be reached safely: its address, TLS connection or a redirect was refused. |

No failed check counts as ready. Provider response bodies and keys are never
stored or shown.

## Remove a saved key or sign-in

A provider administrator with a recent authenticator sign-in can remove a
credential under **Integrations**:

- **Remove saved key** on an API-key connection disconnects the connection,
  revokes every credential grant that names the key and closes their open
  sessions, which also cuts off a provider call in flight. The provider service
  then deletes the stored key and signs a removal statement. Saved keys are
  sealed at rest, each encrypted with AES-256-GCM under its own data key.
- **Remove sign-in** on a Claude subscription connection archives the
  connection, so no run can use the sign-in and a running Claude stage stops at
  its next check. Custody then deletes the sealed sign-in and keeps a tombstone.
  To sign in again, add a new Claude connection.

The removal shows as pending until custody confirms the deletion; a pending
credential is already revoked and unusable. Removal does not revoke the key or
end the session at the provider, so also revoke the key in the provider's
console or sign out in your Claude account. To keep using an API-key connection,
save a new key on it. See
[data retention, deletion and custody](/docs/data-retention-deletion-custody).

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
It evaluates the diff and evidence against the stated requirements.

The reviewer can be a signed-in Codex or Claude subscription or an API-key
connection. A subscription reviewer runs both review passes through the
protected subscription runtime, each pass in its own isolated session, and
needs no API key. The installer's default review policy prefers a signed-in
subscription that runs the model of a published agent profile, and otherwise
uses the installation's oldest Codex API-key connection. Owner choices under
**Runtime workers** take precedence.

A model family is the reviewer's upstream provider: every OpenAI model, through
Codex, is one family, and every Anthropic model, through Claude, is another. A
Review stage for high-risk work is refused before any pass runs when the
reviewer shares the model family with an implementer, and for critical-risk work
when it shares the provider. The refusal reads "High-risk changes need a
reviewer from a different provider or model than the implementer."

In 1.0.0 every change is reviewed as Critical, because no change is
semantically classified yet. Every run therefore needs a reviewer from a
different provider and model family than every Implement step, plus a human
technical review, and an installation with connections from only one provider
cannot complete review. Connect both a Codex and a Claude connection. See
[review requirements for every run](/docs/validation-and-approvals#review-requirements-for-every-run).
