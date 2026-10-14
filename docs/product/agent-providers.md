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

Add connections under **Integrations > Agent providers**. Each connection is
one card that shows its name, status, health and type. Choose the name to open
its details: health, sign-in or key state, when it was last checked and last
used by a run, plan usage, and its actions (**Sign in**, **Verify**, **Plan
value**, **Remove sign-in** or **Remove key**). Every action opens a dialog that
names the connection it acts on.

- **Codex with a subscription** signs in with a Codex device code. BranchRunner
  keeps the login encrypted and runs Codex with it in protected containers. The
  installation selects protected Codex execution automatically once the worker
  installer has attested the host.
- **Claude Code with a subscription** signs in through the official Claude Code
  CLI. See [Claude subscription sign-in](#claude-subscription-sign-in) below.
  BranchRunner runs that official CLI with the connection's token inside its
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
time. Each run gets its own CLI process and home directory. A concurrent Codex
session cannot rotate the login: only an exclusive verification refreshes it. A
Claude token is never refreshed, and each run's copy must come back unchanged.
Concurrent sessions are supported but not yet live-verified.

A saved login records authentication state. It does not prove a current
account entitlement, execution authority on a worker, or a successful agent
step. The run-start check verifies those before a run starts.

## Claude subscription sign-in

Choose **Add Claude subscription** to create the connection. Then a provider
administrator who signed in to BranchRunner with their authenticator within the
last five minutes chooses **Sign in** on it:

1. The dialog shows the Claude sign-in link. Open it, sign in to Claude and
   approve access.
2. Copy the code the Claude page shows and paste it into the dialog. The code
   goes only to the waiting official CLI; BranchRunner does not store, log or
   return it.
3. The CLI creates a token for this connection with `claude setup-token`, valid
   for one year. BranchRunner checks the account with the CLI, then confirms
   the new token and the date until which it is valid.

Each sign-in accepts one code. A wrong or expired code ends the sign-in; choose
**Start a new sign-in** for a fresh link and code.

The token is held encrypted in custody on the control-plane host and is passed
only to the official CLI that BranchRunner runs. It carries no account email,
organization or plan type, so the card identifies it only by a short
fingerprint (`token …` and eight hex digits).

The administrator who signed the connection in is its signing administrator.
Only runs that they start can use the connection. A run that someone else
starts is refused at its Claude session approval.

An installation can hold several Claude subscription connections, each signed
in to its own Claude account. Each connection keeps its own token and custody
records, so signing in, verifying or removing one never changes another.

The connection's details show "Long-lived token, valid until" and the date. In
the last 30 days they add "Sign in again before" the date "to renew the token".
A Claude session is refused with `sign_in_required` when the token would expire
before the session's deadline plus 10 minutes. After the token expires, the
details say "The token expired on" the date, the connection reads **Needs
attention** and **Verify** is no longer offered. Choose **Sign in again** to
renew it.

**Verify** checks that custody still holds the same token and that it works,
using your current BranchRunner session; it asks for no new sign-in. Only the
signing administrator can verify the connection. A successful Verify also
records the plan usage the CLI reports.

A Claude subscription pass (Plan, Implement or a Review pass) runs until the
first of these ends: the agent profile's time limit, the 15-minute session
maximum, its session approval, or 10 minutes before the token expires. The
worker must keep its lease for the whole pass. If the lease is lost, for
example because the worker stopped or restarted, the pass is stopped.

### Sign in again

**Sign in again** renews a Claude subscription token in place. It runs the same
link-and-code sign-in, and the CLI creates a new one-year token for the same
connection. The connection, the agent profiles that use it and its plan usage
history stay. BranchRunner no longer uses the previous token. You can sign in
with a different Claude account.

- Any provider administrator can sign in again, also after the token expired.
  Like the first sign-in, it needs an authenticator sign-in to BranchRunner
  within the last five minutes.
- The administrator who signs in again becomes the connection's signing
  administrator. From then on only runs that they start can use the
  connection, and only they can verify it. The dialog says so before the
  sign-in starts, and the audit record names the previous and the new signing
  administrator.
- It is refused while Claude operations on the connection are still running:
  "Claude subscription operations on this connection are still running. Sign
  in again after they settle." Wait until the Claude stages that use the
  connection have finished or stopped, then try again.
- If the new sign-in does not finish, the earlier sign-in stays unchanged.
- It is not offered after the organization revoked provider credentials. Then
  choose **Remove sign-in** and add a new Claude subscription connection.

Signing in again does not revoke the previous token at Anthropic. See
[remove a saved key or sign-in](#remove-a-saved-key-or-sign-in).

### When a Claude subscription can run agents

A Claude subscription card reads **Execution available** once the server can
run agents with it. That needs all of these:

- protected Claude execution is selected on the installation host;
- the worker on the control-plane host is online and reports Claude execution;
- the connection's latest sign-in was made for the connection as it is now,
  the organization has not revoked it, and its token has not expired.

Otherwise the card names the first thing to fix. Its details list each failed
condition under **Agent execution**, with the one action that fixes it. The
first-run checklist and **Coding agents** show the same text. Every run start
checks these conditions again.

The API reports the same causes in the connection's `executionBlockers`: at
most one sign-in code, then at most one installation code. The list is empty
exactly when `executionAvailable` is true.

| Code | What the card says | What to do |
| --- | --- | --- |
| `custody_unreadable` | The server could not read the connection's sign-in records. | Reload the page. |
| `not_signed_in` | No Claude account is signed in on the connection. | Choose **Sign in**. |
| `credentials_revoked` | The organization revoked provider credentials after this account signed in. | Choose **Remove sign-in**, then add a new Claude subscription connection. |
| `sign_in_outdated` | The sign-in was made for an earlier version of the connection. | Choose **Sign in again**. |
| `token_expired` | The one-year token expired, so runs cannot use it. | Choose **Sign in again** to create a new token. |
| `protected_claude_not_selected` | Protected Claude execution is not selected on the installation host. | The worker installer selects it once it has prepared the Claude runtime. If an owner opted out, run `sudo bash /opt/branchrunner/current/infra/deploy/branchrunner-configure-execution.sh claude-protected` on the server. |
| `claude_worker_offline` | No worker that runs Claude stages is online. | Install or restart the worker on the control-plane host as root: `sudo bash /opt/branchrunner/current/scripts/install.sh worker`. |
| `claude_worker_capability_missing` | The worker on the control-plane host does not report Claude execution. It runs another release, or the protected Claude runtime is not answering it. | Run `sudo bash /opt/branchrunner/current/scripts/install.sh worker` on that host. |

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
under **Settings → Admitted provider hosts**. This needs a recent sign-in. The
**Admit host** action on such a connection opens that section.
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

After saving the key, choose **Verify** on the connection. The dialog shows the
key's exact API target and sends one small, authenticated Responses request
through the provider service with the protected key. The request asks for at
most 256 output tokens and expects a 64 KiB response or less, so it may use a
little quota. The check passes only when the service returns a completed
Responses result that contains a one-time nonce.

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

- **Remove key** on an API-key connection disconnects the connection,
  revokes every credential grant that names the key and closes their open
  sessions, which also cuts off a provider call in flight. The provider service
  then deletes the stored key and signs a removal statement. Saved keys are
  sealed at rest, each encrypted with AES-256-GCM under its own data key.
- **Remove sign-in** on a Claude subscription connection archives the
  connection, so no run can use the token and a running Claude stage stops at
  its next check. Custody then deletes the sealed token and keeps a tombstone.
  A removed connection cannot be signed in again; to use Claude again, add a
  new Claude subscription connection. To replace only the token and keep the
  connection, choose **Sign in again** instead.
- **Remove sign-in** on a Codex subscription connection revokes the saved login
  and stops its active sessions.

The removal shows as pending until custody confirms the deletion; a pending
credential is already revoked and unusable. Removal does not revoke the key or
token at the provider: also revoke an API key in the provider's console.
BranchRunner cannot revoke a Claude token at Anthropic; Anthropic's help
center says the Claude Code tokens of your Claude account are managed under
**Settings > Claude Code** on claude.ai
([how do I log out of all active sessions](https://support.claude.com/en/articles/10310342-how-do-i-log-out-of-all-active-sessions)).
To keep using an API-key connection, save a new key on it. See
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

The details of a Codex or Claude subscription connection show the plan-limit
usage the provider reported: a reached limit first, then the 5-hour window, the
weekly window and any other window, for example "5-hour: 62% used, resets
20:15", and when the figures were observed. For Claude, the weekly window is
the one the Claude Code CLI reports as `seven_day`. Each sign-in, **Verify** and
run records a new reading.

A connection without a reading shows "Not measured yet". A 5-hour or weekly
window that the latest reading did not include shows "unknown". Neither is
ever shown as 0 %. An API-key connection has no plan windows.

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

Record a subscription's monthly fee or an API key's discount with **Plan value**
on the connection (**Set plan fee** or **Set API discount**). The same dialog
shows the connection's period value and the model list prices. A model without
a list price shows "list price not set" and is left out of totals, which are
then marked incomplete. When the provider reports no cache
detail, the figure is marked as an upper bound. Budget enforcement counts API
charges only, not subscription fees.

## Review independence

An independent reviewer does not inherit the implementing session's context.
It evaluates the diff and evidence against the stated requirements.

The reviewer can be a signed-in Codex or Claude subscription or an API-key
connection. A subscription reviewer runs both review passes through the
protected subscription runtime, each pass in its own isolated session, and
needs no API key. The workflow's Review stage runs a published profile with the
reviewer role, and that profile decides who reviews. The installation derives
its review signing policy from the published reviewer profiles: a signed-in
subscription first, otherwise the OpenAI API-key connection a reviewer profile
uses. Nothing is chosen on the server. Publish the reviewer profile on the
second provider before review signing first turns on: once review signing is
installed, publishing a different reviewer profile does not change the
installed policy.

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
