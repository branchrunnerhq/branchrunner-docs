---
title: Runtime workers
description: Install, enroll and admit the execution hosts that provision isolated workspaces and run agent steps.
group: Runtime
order: 30
---
## Responsibility boundary

Runtime workers receive signed work instructions from the control plane.
Repository checkouts, coding agents, build services and validation commands run
on workers, in hardened containers, rather than in the control plane. A worker is
the only BranchRunner component with Docker authority on its host.

## Readiness

A worker needs a current certificate and incarnation, explicit administrative
admission, compatible capabilities, eligible capacity, and a healthy runtime,
storage, network policy and control-plane connection. A registered or
heartbeating worker can still be ineligible for work. Repository source import,
runtime image pins and provider execution authority are separate prerequisites.
The run-start check reports each missing item.

## Install the worker

`scripts/install-worker.sh` runs from a clean Git checkout of the server's
release commit (`workerSourceCommit` in
`/etc/branchrunner/quickstart/state.json`). Keep that checkout outside
`/opt/branchrunner`. The host needs Linux with AppArmor enforcing, Docker Engine
24 or newer, at least 20 GiB free disk, and a clock within 5 seconds of the
control plane. Docker Desktop cannot be used.

By default the installer runs fast installation checks: release identity, a
smoke run of the installed worker and a local host preflight. It also builds
the validation and diff-analysis images on the host. `--verify` adds the full
product test suite, which takes about two hours and is meant for developers and
release candidates. A rerun on an enrolled host keeps the same identity and
never enrolls again.

## Add a worker on the same server

Run the worker installer on the server with `--repo-dir <checkout>` and
`--server https://<host>`. The worker shares a service account with the control
panel, so the installer stops those containers, installs the worker, and starts
them again with the API last. The control panel is unavailable for that window.

## Add a worker on another host

1. On the management server, rerun the installer once after owner setup with
   `--remote-workers <CIDR>`, naming the address range your workers connect
   from. The installer publishes the mutual-TLS worker channel (port 5181 by
   default) and a public onboarding profile, both reachable only from that range.
   It then prints the worker command and the Worker CA SHA-256. Allow the same
   port in any cloud firewall in front of the server.
2. On the worker host, from the matching checkout, run
   `sudo ./scripts/install-worker.sh --server https://<host>`. You can pin the CA
   with `--worker-ca-sha256 <hex>`.
3. When the host checks pass, the installer asks for a token at a hidden prompt.
   Generate it in **Runtime workers > Add a worker**. The token is single-use
   and never accepted as an argument or file.
4. The worker enrolls, starts its service and reports its first heartbeat. It
   then shows as **Awaiting admission**. Review it and admit it explicitly;
   nothing is scheduled before that.

Installer exit code `4` means the worker is installed and checked but enrollment
did not complete; run `sudo br-worker-enrol` to finish it. This remote flow is
implemented; its live acceptance on a fresh remote host is still pending.

## Operations

- Keep workers disposable and replaceable.
- Separate pools when repositories require different network or trust
  boundaries.
- Send worker logs to your normal operations system.
- Drain a worker before maintenance. A draining worker still delivers pending
  results so active work remains attributable.
- After a control-plane upgrade, rerun the worker installer on every worker
  host. The worker channel refuses a mismatched protocol version.
