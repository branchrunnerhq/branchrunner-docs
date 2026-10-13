---
title: Data retention, deletion and custody
description: What self-hosted BranchRunner deletes on its own, what it keeps, which removals you can make, and what it does not guarantee.
group: Operate
order: 142
---
This page states what self-hosted BranchRunner 1.0.0 deletes, what it keeps,
and what it does not guarantee, on a single-host installation. You run the
host, so you also own its backups, snapshots, logs and disks. BranchRunner
provides no managed backup, restore or deletion service.

## What BranchRunner deletes on its own

Retention is on by default. The API runs a maintenance pass at startup and then
every 60 seconds (`BR_RECONCILE_INTERVAL_MS`), and a separate retention pass at
startup and then every 5 minutes. Each sweep is bounded and fails on its own, so
one failing sweep does not stop the others.

| Data | What happens | When | What stays |
| ---- | ------------ | ---- | ---------- |
| Run log text (encrypted log bodies in the artifact store) | The ciphertext is deleted from `/var/lib/branchrunner/artifacts`. | 30 days after the log stream started for a succeeded run; 90 days for a failed, cancelled or unclassified run. A stream with a hold is skipped until the hold ends. At most 200 streams per pass. | Chunk metadata: sizes, digests, sequence ranges and the deletion reason. |
| Uploads that never completed | The unpublished object and its reservation are removed. | 24 hours after the reservation, at most 100 per pass. Partial upload directories older than 1 hour are removed when the API starts. | Nothing. |
| Expired sign-in transactions and login-throttle entries | The rows are deleted. | When they expire. | Nothing. |
| Completed request-replay records (idempotency keys) | The rows are deleted. | 24 hours after completion, at most 500 per pass. Records still in progress are never deleted. | Nothing. |
| Worker log-delivery receipts | The rows are deleted. | 24 hours after receipt, at most 1,000 per pass. | Nothing. |
| Browser sessions | The session is revoked and stops working. | When its idle or absolute lifetime ends, or on sign-out. | The session row, with the revocation time and reason. |
| Credential grants, credential sessions, worker leases and egress exceptions | Moved to an expired or ended state. | When they lapse. | The rows and their audit events. |
| GitHub App installation tokens | Kept in memory only, and revoked at GitHub after use. | After each use, or at the token's lifetime. | Nothing. |

### Run artifacts

Diffs, validation reports, evidence archives, archive-inspection output,
provider output and other encrypted run artifacts are deleted when their
retention ends. Each artifact's class sets its retention when it is stored:

| Artifacts | Retention |
| --------- | --------- |
| Diffs, validation reports, build and archive-inspection output, provider output | 30 days (the run's frozen source tree: 7 days) |
| Completion evidence (evidence archives, review records, publication proposals) | 180 days |
| Anything an evidence packet references | At least until 180 days after that packet's seal |

The retention pass deletes an artifact only when the built-in (or Vault)
retention custody is ready and all of these hold:

- its retention has ended, including the co-retention of any evidence packet
  that names it;
- no legal, evidentiary or operational hold covers it, its run, its task or its
  organization;
- its run has finished, no publication approval is open and no security
  incident is unresolved.

The custody then deletes the encrypted object, its wrapped key and its backup
reference, each with a signed, durable custody receipt, and seals a deletion
tombstone. A held, undue or blocked artifact is checked again at its deadline
(holds every 6 hours, active runs every hour). A failed deletion is retried with
backoff from 5 minutes. A deletion whose three attempts all fail, or that can no
longer be replayed exactly, stops and is reported for operator review. At most
25 artifacts are handled per pass, and a pass ends after 4 minutes. Nothing is
deleted while the custody is not ready.

What stays: the artifact's metadata row, its retention decision, the attempt
records and the tombstone. Artifacts stored before their organization had a
region pin carry no retention policy and are kept. On a fresh installation there
are none, because the API pins every organization when it starts.

### Other bounded records

| Data | What happens | When |
| ---- | ------------ | ---- |
| Revoked browser sessions | The session row is deleted. | 30 days after revocation, at most 1,000 an hour, unless an approval, waiver, hold, review or custody grant references it. |
| Activity events (the task activity history) | Archived to `/var/lib/branchrunner-event-archives`, removed online. | Daily, for events older than 90 days. An event that a review annotation references stays online. |
| Retention custody inventory observations | Older revisions are deleted. | Hourly, beyond the last 24 hours. The newest revision and any revision a backup authorization or receipt references stay. |
| Retention telemetry batches | Deleted. | Hourly, after 90 days, unless an alert signal references the batch. |
| Prepared handover bundles on a worker | Deleted by the worker. | Every 15 minutes, once expired, delivered past their retention, or orphaned. A legal hold keeps them. |
| A worker's per-repository source cache | Repacked and pruned (`git gc --prune=now`). | At most once a day, during a source import into that cache. |

## What BranchRunner keeps

BranchRunner 1.0.0 does not delete the following automatically:

- **Run artifacts under a hold or still within their retention**, and artifacts
  without a retention policy. The artifact store's quota bounds what remains
  (default 20 GiB, `BR_ARTIFACT_STORE_QUOTA_BYTES`).
- **Database records:** organizations, users, memberships, repositories, tasks,
  runs, conversations, approvals, workflows, reviews and evidence metadata. The
  product has no action that deletes an organization, user, repository, task,
  workflow, run or artifact.
- **Audit events.** They are append-only; the database refuses updates and
  deletes.
- **Activity-event archives.** The daily archival job moves old events into
  files under `/var/lib/branchrunner-event-archives`. BranchRunner never deletes
  those files; keeping or expiring them is up to you.
- **Retention evidence:** retention decisions, deletion attempts, tombstones,
  custody releases and receipts, backup authorizations and alert signals are
  append-only.
- **Prepared-handover receiver objects**, when that optional receiver is
  installed. The built-in custody does not delete them.
- **Worker host data:** each workspace's repository clone, the per-repository
  source cache (repacked, never removed), Docker images, dependency caches and
  the worker's journald log. Destroying a workspace removes its containers,
  volumes, networks and agent worktrees, but not the clone or the cache.

## Removal actions you can take

Every removal in the product is an explicit action. Most of them revoke or mark
a record rather than delete it.

| Action | What it does | What it does not do |
| ------ | ------------ | ------------------- |
| Sign out, or sign out everywhere | Revokes the browser session, or all of your sessions. | It does not delete session records. |
| Cancel a run | Revokes the run's credentials, sessions, egress exceptions. It fences the run's lease and queues workspace destruction, unless you chose to keep the workspace. | It does not delete the run, its conversation, logs or evidence. It does not touch GitHub. |
| Destroy a workspace | The worker removes the workspace's owned containers, volumes, networks and agent worktrees, then checks that they are gone. Anything it cannot prove it owns is quarantined, not deleted. | It leaves the repository clone and the source cache on the worker host. |
| Cancel a publication | Stops publication and records the cancellation. | The branch and draft pull request stay on GitHub. Close or delete them there yourself. |
| Revoke an approval, egress exception, credential grant, service identity or enrollment token | Marks it revoked and stops it working. | It deletes nothing. |
| Decommission a worker | Revokes its certificates, fences its live leases and ends the credential sessions those leases authorized, in one transaction. The identity can never register again. | It changes nothing on the worker host. |
| Remove a provider connection | Archives an unused connection. | It refuses a connection that has credentials, custody or history. |
| Remove a Codex sign-in | Replaces the custody record with a revoked record that holds no credential, and stops active sessions. | The old file is replaced, not securely overwritten. It cannot end the session at OpenAI. |
| Remove a saved API key | Disconnects the connection, revokes the key's credential grants and closes their sessions at once. The provider service then deletes the stored key and confirms it with a signed statement. | It keeps the enrollment receipt and audit history. The file is unlinked, not overwritten; it held only ciphertext. It cannot revoke the key at the provider. |
| Remove a Claude sign-in | Archives the connection, so no run can use its token, and a running Claude stage stops at its next check. Custody then deletes the sealed token, keeps a tombstone and confirms it with a signed statement. | It cannot revoke the token at Anthropic. |
| Uninstall the GitHub App at GitHub | The next grant check (normally within four minutes) marks the repository grants revoked and pauses affected runs. | It deletes nothing in BranchRunner or at GitHub. |

Previews are not available in 1.0.0, so a 1.0.0 installation has no preview
links or preview sessions to revoke.

Removing a saved API key or a Claude sign-in requires a provider administrator
with a recent authenticator sign-in. The removal shows as pending until custody
confirms the deletion; a pending removal is already revoked and unusable, and
the next reconciliation pass retries the deletion.

**Saved provider API keys** are stored sealed, never in plaintext. Each key is
encrypted with AES-256-GCM under its own data key, which is wrapped with the
provider service's RSA-OAEP-3072 enrollment key. That key is kept apart from the
key files, and the key files are readable only by the service account (mode
0600). **A Claude subscription sign-in** is a token valid for one year, kept
sealed on the control-plane host under
`/var/lib/branchrunner/claude-subscription-custody` until it is removed. Removal
deletes that file and leaves a tombstone, so the connection cannot be enrolled
again; add a new Claude subscription connection to sign in again.

Removing either one does not revoke it at the provider. Also revoke an API key
in the provider's console. BranchRunner cannot revoke a Claude token at
Anthropic.

## Holds

A log stream or artifact with a hold is not deleted until the hold ends.
Retention holds are append-only records; releasing one is a separate recorded
action. A hold prevents deletion. It does not grant anyone permission to read
the content.

## What BranchRunner does not guarantee

- **Backups and snapshots.** Deleting data online does not remove it from
  backups, host or volume snapshots, or copies you made. Restoring a backup
  brings deleted data back. The built-in custody never claims backup expiry, so
  it reports backup expiry as `unavailable`.
- **Database pages and WAL.** Deleted or updated rows can remain in PostgreSQL
  data files and WAL until PostgreSQL reuses that space.
- **Secure erasure of disks.** Files are unlinked or replaced, not overwritten.
  On SSDs and copy-on-write filesystems the old blocks can remain readable.
- **Third parties.** BranchRunner does not delete anything at GitHub (branches,
  pull requests, commits, comments, webhooks) or at model providers. Prompts,
  code and outputs sent to OpenAI or Anthropic are kept under that provider's
  own policy.
- **Downloaded copies.** Anything a user downloaded or copied is outside
  BranchRunner.
- **Logs outside the product.** Container logs are capped at 10 MiB × 5 files
  per container. The worker and helper units log to journald with the host's
  journald settings. nginx logs follow logrotate. None of these is an erasure
  guarantee, and any log shipping is yours.

## What you can observe

- **`/ready` on loopback** (`http://127.0.0.1:5180/ready`; nginx denies it
  publicly):
  - `artifact_storage` is `ok`, `not_configured` or `degraded`. A degraded
    detail names `artifact_storage_capacity_low` (free space, free inodes or
    quota headroom below its floor) or `artifact_storage_capacity_unreadable`.
    Low capacity does not fail readiness on its own.
  - `phase6_retention` reports the state of the built-in retention custody and
    the last retention pass: deleted, retained, refused and failed artifacts,
    how many are due, how many keep failing and how many need operator review.
    It fails when the custody is ready but no retention pass has completed for
    15 minutes.
- **API logs** (`sudo docker logs branchrunner-api`) record each maintenance and
  retention pass, each failed sweep with its name, and each artifact whose
  deletion needs operator review.
- **Activity archival:** `systemctl status branchrunner-activity-retention.timer`
  and `journalctl -u branchrunner-activity-retention.service`.
- **Worker log:** `journalctl -u branchrunner-worker.service`.

BranchRunner sends no alerts. The installer deploys no Prometheus, Alertmanager
or notification channel. Monitor `/ready`, disk and inode use, certificate
expiry, failed `branchrunner*` units, worker heartbeats and backup age with your
own tools.

## When storage fills up

- **Artifact store.** `/ready` reports `artifact_storage` degraded when free
  space beneath the store falls below 2 GiB, free inodes below 10,000, or the
  store's quota headroom below the smaller of 2 GiB and 10 % of the quota. A
  write that would cross the quota is refused with `store_quota_exceeded`, and a
  write to a full filesystem with `store_filesystem_full`. Nothing partial is
  published, artifact writes through the API return HTTP 503, and log chunks are
  recorded without their text. To recover, grow the filesystem, or raise
  `BR_ARTIFACT_STORE_QUOTA_BYTES` in
  `/etc/branchrunner/quickstart/api.local.env` and recreate the API. Do not
  delete individual files under `/var/lib/branchrunner/artifacts`: their
  database records would then point at missing evidence.
- **PostgreSQL volume.** PostgreSQL's own behaviour applies: writes fail, and
  the server stops if it cannot write WAL. Free space on that filesystem or grow
  it, then restart. Never delete WAL or other files in the data directory by
  hand.
- **Worker workspace disk.** Below 20 GiB or 100,000 free inodes at startup, the
  worker reports `disk_space` failed and runs no workloads. Destroy workspaces of
  finished runs and remove Docker images that no container uses.

## Recovery and backups

See [backup and recovery](/docs/backup-and-recovery) for the recovery set.
After a restore, re-apply any deletions and revocations recorded after the
capture before the installation serves users. To retire a worker host,
decommission it under [runtime workers](/docs/runtime-workers), then erase its
workspace root and cache under your own retention procedure.
