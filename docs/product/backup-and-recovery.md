---
title: Backup and recovery
description: Protect a self-hosted BranchRunner installation with operator-owned backups, and understand what built-in retention does and does not delete.
group: Operate
order: 140
---
## Your responsibility

Backups are operator-owned. BranchRunner supplies no managed backup or restore
service. The product source includes an operator backup, restore and deletion
runbook (`docs/runbooks/operator-backup-restore.md`) with the exact commands for
the single-host installation.

## What to protect

Capture these together, as one recovery set, while their writers are stopped:

- the PostgreSQL database volume `branchrunner-quickstart-postgres`;
- product data in `/var/lib/branchrunner`: encrypted artifacts, prepared
  handover objects, the GitHub App private key and state, and the Codex and
  Claude sign-in custody;
- installation configuration and private identity in `/etc/branchrunner`:
  database passwords, the TOTP and artifact keys, the Worker CA, the signing,
  handover and retention keys, and provider custody;
- each enrolled worker's files under `/etc/branchrunner`.

A database-only restore is not a product restore: database references cannot
recreate the encrypted artifacts or keys. Workspaces and caches are working
state and are not restored. Repository source remains authoritative in your Git
host.

## Retention and deletion

Built-in retention is on by default. It automatically deletes expired run
artifacts that are not on hold from the local artifact store. It does not erase
backups, snapshots, database pages or WAL, and it reports backup expiry as
unavailable. Deleting data from your backups follows your own retention policy.

## Recovery practice

Restore into an isolated environment, verify users and configuration, inspect
historical evidence, and run a non-production workflow. Record recovery time and
any manual steps. Database migrations are forward-only: once an upgrade has
applied a migration, going back means restoring the recovery set taken before
it.

A capture on a live installation has been exercised. A complete restore onto a
clean host has not yet been exercised as a live drill; see
[release status](/docs/release-status).
