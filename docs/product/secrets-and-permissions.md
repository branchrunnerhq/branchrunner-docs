---
title: Secrets and permissions
description: Control credential custody, runtime injection, and human approval boundaries.
group: Configuration
order: 70
---
## Credential custody

Provider API keys and subscription logins are held by the installation's
provider service, encrypted on your server. Each saved API key is encrypted with
AES-256-GCM under its own data key, which is wrapped with the provider service's
enrollment key, kept apart from the key files. API-key stages call the provider
through the installation's provider gateway, and subscription logins are used
only by the official CLIs inside protected containers. GitHub
installation tokens stay inside the SCM service and are never returned to the
API, worker, agent, database or logs.

Credential enforcement starts in shadow mode. An owner or administrator can
switch it to enforcing under **Integrations**. This cannot be switched back.

## Built-in key custody

The installer creates the signing, handover and retention keys on the server
itself, in root-owned configuration directories with restricted file modes. No
external Vault is required. Vault remains an optional integration that you
select explicitly. Include these keys in your backups; see
[backup and recovery](/docs/backup-and-recovery).

## Permission layers

- The GitHub App's repository selection and permissions control which source
  BranchRunner may read and where it may open draft pull requests.
- Each repository's draft pull request policy limits the allowed base branches.
- Organization roles control who may configure repositories, providers and
  workflows, and who may approve.
- Agent profiles request tool classes within installed platform limits.
- Network policy controls which services containers can reach.
- Human approvals control the plan, publication and other sensitive
  transitions. Publication approval needs a recent authenticator sign-in.

## Rotation

Rotate a provider key by adding a new key revision to its connection. Review
which agent profiles use a connection before archiving it to avoid unexplained
run failures.

To withdraw a leaked or unwanted credential, use **Remove saved key** or
**Remove sign-in** under **Integrations**. It revokes the credential first and
then deletes it from custody. Also revoke the key at the provider. See
[agent providers](/docs/agent-providers).
