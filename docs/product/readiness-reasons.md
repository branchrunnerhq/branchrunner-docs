---
title: Readiness reason codes
description: Read the bounded reason codes in /ready, the control-plane logs and the worker journal, and act on them.
group: Operate
order: 152
---
## What a reason code is

Every readiness check and every installed authority reports a short reason code
when it is not available, instead of a generic "unavailable". A code contains no
paths, keys, hosts or credentials, so you can include it in a bug report as it
is.

Codes are joined from fixed segments with `:`. Read them from left to right:
the first segments name the subsystem, the last one names the cause. A few
endings are shared by every subsystem:

- `system_<errno>`, for example `system_enoent` or `system_econnrefused`: a
  missing file, refused connection or denied permission.
- `sqlstate_<code>`, for example `sqlstate_42501`: a PostgreSQL error, here a
  missing grant.
- `env_missing:<VARIABLE>` or `env_invalid:<VARIABLE>`: the installer has not
  written that setting, or it is malformed.
- `directory:<rule>` or `file:<name>:<rule>`: protected installation files have
  the wrong owner, mode or type.
- `error_class_<Name>` or `unclassified`: no more specific cause is known.
  Include the full line in a bug report.

An error whose message is itself a short code, such as
`handover_custody_peer_refused`, is reported as that code rather than as
`error_class_Error`. A chain is cut at 160 characters, so in a long chain the
last segment can be cut short. Match it by its beginning.

Source: `server/src/application/boundedFailureReason.ts`,
`server/src/application/installedAuthorityReason.ts` and
`packages/worker/src/boundedFailureReason.ts`.

## Where to read them

On the server:

```bash
# Every check, one line each.
curl -s http://127.0.0.1:5180/ready | jq -r '.checks[] | "\(.name)\t\(.status)\t\(.detail)"'

# Only the checks that are not ok.
curl -s http://127.0.0.1:5180/ready | jq '.checks[] | select(.status != "ok")'

# Control-plane warnings and errors.
docker logs branchrunner-api --since 30m 2>&1 | grep -E '"level":"(warn|error)"'

# Phase 6 observer, handover services and publication authority.
docker logs branchrunner-phase6-observer --since 30m 2>&1
docker logs branchrunner-handover-custody --since 30m 2>&1
docker logs branchrunner-handover-receiver --since 30m 2>&1
docker logs branchrunner-publication-authority --since 30m 2>&1

# Evidence signing activation.
journalctl -u branchrunner-signing-activation.service -n 200 --no-pager
```

On a worker host:

```bash
journalctl -u branchrunner-worker.service --no-pager \
  | grep -E 'Phase 6 worker capability|execution family|handover source authority'
```

`/ready` returns HTTP 503 while any blocking check is not `ok`. A failed Phase 6
check ends with `Cause:` and the code. The control-plane log keeps only the
message and record identifiers, so take the code from `/ready` when a log line
names only the subsystem. A result may be cached for a few seconds; read it
again after a fix.

## Phase 6 runtime checks

`phase6_validation`, `phase6_source_repair`, `phase6_independent_review` and
`phase6_archive_inspection` name the failing bundle, authority and evidence
source. For example:

```text
Cause: validation:unavailable:runtime:unavailable:worker:admission:runtime_identity_not_admitted.
```

This reads: the `validation` bundle's `runtime` authority is unavailable because
the `worker` evidence source refused admission with
`runtime_identity_not_admitted`.

| Code at the end | Meaning | What to do |
| --- | --- | --- |
| `admission:runtime_identity_not_admitted` | No runtime identity has been admitted for the worker. | Admit the runtime identity under **Runtime workers** after comparing it with what the worker installer printed. |
| `admission:runtime_identity_changed` | The worker reports a change outside its images, for example another Docker version, security packs or egress path. Every capability on the worker is refused. | Compare both identities under **Runtime workers**, then admit the new one if you expected the change. |
| `admission:runtime_identity_image_not_admitted`, `admission:runtime_identity_image_changed` | This capability's image became ready after the identity was admitted, or it now reports a different image. Only this capability is refused; the worker's other capabilities keep running. | Admit the reported runtime identity under **Runtime workers**, which names the image awaiting admission. |
| `admission:runtime_identity_admission_invalid` | The stored admission no longer matches its own digest. | Include the line in a bug report. |
| `admission:capability_unavailable` | The worker reports this capability as not ready. | Read the worker's `Phase 6 worker capability` line. |
| `admission:capability_identity_mismatch` | The worker's runtime, image or profile differs from what its admitted identity requires. | Rerun the worker installer from the control plane's release, then admit the identity if it changed. |
| `admission:payload_version_unsupported`, `admission:capability_protocol_required`, `admission:worker_protocol_unsupported` | The worker is older than the control plane. | Upgrade the worker with the worker installer. |
| `admission:capability_receipt_stale` and other `capability_receipt_*` codes | The worker's capability receipt is old, missing or invalid. | Check that the worker service runs; check the host clocks; restart the worker. |
| `no_worker`, `worker_not_eligible`, `no_current_receipt` | No worker is enrolled, admitted and heartbeating with a current receipt. | Install and admit a worker; resolve host drift under **Runtime workers**. |
| `handover:agent.prepare_api_source:<reason>`, `handover:agent.prepare_subscription:<reason>` | The workflow Review source handover is not admitted. | Fix the inner reason, as above. |
| `database:migration_ledger_not_current`, `database:catalog_objects_missing` | The database schema is behind this release. | Rerun the control-plane installer. |
| `database:catalog_definition_drift` | A database object was changed by hand. | Include the line in a bug report. Do not edit database objects. |
| `artifact_store:probe:filesystem_unavailable` | The artifact volume cannot be read or written. | Check the volume is mounted and has free space. |
| `artifact_store:probe:key_service_unavailable` | The artifact encryption key is unavailable. | Rerun the control-plane installer. |
| `review_provider:unavailable`, `review_executor:unavailable` | The review policy, reviewer connections or reviewer profiles are not current. | If a Claude reviewer connection lapsed, sign in the subscription again by adding a new connection; otherwise rerun the installer. |
| `probe_failed:transport_unavailable`, `probe_deadline` | The API cannot reach the Phase 6 observer. | Check `branchrunner-phase6-observer` is running and read its log. |
| `probe_failed:peer_identity_mismatch`, `authority_unconfigured` | The observer's registration or certificate is not the installed one. | Rerun the control-plane installer. |

Source: `server/src/application/phase6RuntimeProductionComposition.ts`,
`phase6ComponentEvidence.ts` and `workerCapabilityAdmission.ts`.

## Worker capability checks

| Check and code | Meaning | What to do |
| --- | --- | --- |
| `worker_capability_authority`: no current receipt | No schedulable worker has a current capability receipt. | Check the worker runs and is admitted. |
| `worker_capability_authority`: privilege missing | The database role lacks a required grant. | Rerun the control-plane installer. Do not grant by hand. |
| `worker_capability_requirement_policy`: failed | The server-owned runtime policy is unavailable. | Rerun the control-plane installer. |
| `runtime_workers`: degraded | Workers are enrolled but none is schedulable. | Check the worker service and its journal. |

Source: `server/src/application/workerCapabilityAuthorityReadiness.ts` and
`server/src/http/routes.ts`.

## Completion and evidence

`phase6_completion` lists unavailable authorities until every completion
authority is verified. It recovers without an API restart.

| Code | Meaning | What to do |
| --- | --- | --- |
| `completion_production_policy_disabled` | Signing activation has not enabled completion yet. | Complete owner setup, then check signing activation in its journal. |
| `completion_authorities_unavailable:env_missing:BR_PHASE6_COMPLETION_REGISTRY` and the other completion settings | Signing activation has not written the completion configuration. | Complete owner setup; rerun the installer if it persists. |
| `completion_authorities_unavailable:evidence_registry_owner_unavailable:<cause>` | The evidence verification registry could not be admitted. | Read the signing activation journal. |
| `completion_authorities_unavailable:signer_installation_rejected:<cause>` | The signer client installation differs from configuration or has the wrong file custody. | Rerun the control-plane installer. |
| `completion_authority:<kind>:authority_unavailable:registry_owner_not_current` | The verification registry owner is no longer current. | Read the signing activation journal. |
| `completion_authority_withdrawn:<cause>` | A ready authority became unavailable. | Fix the cause; the API recomposes on its own. |

Source: `server/src/application/completionProductionComposition.ts` and
`completionInstalledAuthorities.ts`.

## Trusted publication

`phase6_publication` shows `Trusted publication unavailable: <code>.`

| Code | Meaning | What to do |
| --- | --- | --- |
| `publication_flag_off` | Publication was turned off with `--no-publication`. | Rerun the control-plane installer with `BR_INSTALL_PUBLICATION=1`. |
| `operations_registration_rejected:<cause>`, `webhook_authority_rejected:<cause>` | The publication operations or webhook installation was refused. | Rerun the control-plane installer. |
| `completion_evidence_verifier_unavailable`, `completion_authority_not_current` | Completion evidence is not available yet. | Fix `phase6_completion` first. |
| `webhook_secret_custody_not_current`, `webhook_verifier_not_current`, `webhook_association_not_current` | The GitHub App webhook authority is not current. | Rerun the installer and check the GitHub App webhook settings. |
| `publication_root_unavailable`, `publication_root_not_current` | The installed publication root could not be composed or was withdrawn. | Read the API log; rerun the installer if its files changed. |

Source: `server/src/scm/installedPublicationApiRuntime.ts` and the installed
publication authorities in `server/src/scm/`.

## Evidence signing

Signing activation prints
`Offline signer artifact-read provisioning refused (<code>); retained custody was preserved.`
when it cannot provision the signer's artifact reader. Codes start with
`signer_artifact_read_`. `signer_artifact_read_approval_invalid` and
`signer_artifact_read_identity_mismatch` mean owner setup must be completed
again; directory, file, key and registration codes mean the control-plane
installer must be rerun.

Source: `server/src/application/signerArtifactReadProvisioning.ts`.

## Handover services

The handover services log one line when they cannot start:

- `Built-in handover custody could not safely start (<stage>: <reason>).` with
  stage `configuration`, `custody_files`, `database` or `listener`;
- `Prepared handover receiver could not safely start (<stage>: <reason>).` with
  stage `configuration`, `receiver_files`, `database`, `object_storage`,
  `custody_client` or `listener`.

For `configuration`, file and `custody_client` stages, rerun the control-plane
installer. For `database`, check PostgreSQL. For `object_storage`, check disk
space. For `listener`, check for a port conflict.

While running, they log refused requests at most once a minute:
`<service> refused a <route> request (<reason>).` Reasons start with
`handover_`, for example `handover_receiver_quota_exceeded`. If a refusal
repeats, include the line in a bug report.

Source: `server/src/handoverCustody/serviceMain.ts`,
`server/src/handover/serviceMain.ts` and `server/src/handover/refusalLog.ts`.

## Worker journal

| Line | Meaning | What to do |
| --- | --- | --- |
| `Phase 6 worker capability <name>: failed — <detail>` | That capability is not offered, so the control plane reports `capability_unavailable`. | Follow the detail; usually rerun the worker installer. |
| `Installed handover source authority is unavailable (<token>)` | Diff analysis, validation and independent review wait for the handover source. | `handover_active_work` clears after the running work; otherwise check the server's handover services. |
| `Installed API execution family is unavailable (api_plan_setup_<step>_<cause>)` | API execution setup failed at that step. | Rerun the worker installer. |
| `Installed Claude execution family is inactive (not_selected).` | Protected Claude is not selected on this host. On the control-plane host the worker installer selects it automatically once it has prepared the Claude runtime, unless an owner recorded the opt-out with `none`. | Rerun the worker installer, or select it by hand: `sudo bash /opt/branchrunner/current/infra/deploy/branchrunner-configure-execution.sh claude-protected`. |
| `Installed Claude execution family is unavailable (claude_<step>_<cause>)` | Protected Claude could not start. | Rerun the worker installer, which selects Claude again, and sign in the subscription again if its connection lapsed. |
| `... failed its readiness probe (runtime_unavailable: archive_docker_daemon_unavailable)` | Docker is not answering the archive probe. | Check `systemctl status docker`. |
| `... failed its readiness probe (runtime_unavailable: archive_boundary_archive_canary_<cause>)` | The archive isolation canary failed. | A `report_deadline` points to an overloaded host; profile or identity causes mean the worker installer should be rerun. |

Source: `packages/worker/src/main.ts`, `phase6ProductionComposition.ts`,
`isolatedArchiveRuntime.ts` and `dockerArchiveBoundaryCanary.ts`.

## Reason families added in October 2026

These families were added on 11 October 2026. Each one is summarised here;
include the full line in a bug report when the suggested action does not clear
it.

- **Per-capability image admission:**
  `runtime_identity_image_not_admitted` and `runtime_identity_image_changed`
  refuse one capability, not the whole worker. See
  [Phase 6 runtime checks](#phase-6-runtime-checks).
- **Host posture after an upgrade:** an upgrade that installs new AppArmor or
  seccomp packs holds the worker. It drains, shows as degraded and takes no
  work, and dependent checks report `worker_not_eligible`. The event
  `worker.host_facts_rebase_pending` records the hold. Admit the reported
  runtime identity under **Runtime workers**; the worker returns to service
  without re-enrollment. If the page says that admission would not clear the
  drift, for example `host_drift_not_installer_managed` after a kernel or
  Docker change, the host needs re-enrollment.
- **Handover custody identity refused:** the handover custody log shows
  `handover_custody_identity_refused`, and `phase6_publication` reports
  `registry_unavailable:custody:handover_custody_response_refused`. Custody
  serves an organization only while at least one of its workers is `active`
  with a current certificate. Return a drained worker to `active`, admit the
  identity of a held worker, or re-enroll a worker whose certificate expired or
  was revoked. The refusal clears on the next request.
- **Trusted publication SCM causes:** `phase6_publication` reports
  `scm_operations_not_current:<cause>`. A cause that starts with `peer:` came
  from the `branchrunner-scm` service; any other cause was found by the API.
  Connection and deadline causes (`rpc_connection_failed`,
  `rpc_deadline_exceeded`) mean you should start or restart `branchrunner-scm`
  and read its log. `peer:scm_operations_not_activated` means that the GitHub
  App is not registered or has another App ID. Certificate, custody and
  registration causes mean the control-plane installer should be rerun.
- **Retention custody:** `phase6_retention` reports
  `retention_custody:<stage>:<cause>` while the retention custody root is not
  installed. Destructive retention is withdrawn until it recovers, and the API
  retries in the background without a restart. Rerun the control-plane
  installer if the code persists. `provider_change_unapproved` means the
  configured custody provider differs from the one recorded at first start.
  Restore the previous provider settings, or approve a deliberate change with
  `BR_PHASE6_RETENTION_PROVIDER_CHANGE_APPROVAL=<previous>:<next>` in
  `api.local.env`.
- **Archive canary report incomplete:** the worker journal shows
  `archive_boundary_archive_canary_report_incomplete:<received>:<client>:<container>:<class>`.
  The archive isolation check ended before it reported, so archive inspection
  stays unavailable (`capability_unavailable`). A container state ending in
  `_oom` points to memory pressure. Otherwise check the host load and rerun the
  worker installer, then include the full line in a bug report if it repeats.

Source: the readiness runbook, `docs/runbooks/readiness-reasons.md` in the
product source.

## What to include in a bug report

BranchRunner has no support service for self-hosted installations. A bug report
should include the full `/ready` output, the exact log line with its
timestamp, the release version and the time of the last installer run. See also
[troubleshooting](/docs/troubleshooting) and
[runtime workers](/docs/runtime-workers).
