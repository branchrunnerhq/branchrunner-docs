---
title: Runtime workers
description: Operate the execution hosts that provision isolated workspaces and run agent steps.
group: Runtime
order: 30
---
## Responsibility boundary
Runtime workers receive signed work instructions from the control plane. Repository checkouts, agent processes, build services, and validation commands run on workers rather than in the control plane.

## Readiness
A worker needs a current certificate and incarnation, explicit administrative
admission, compatible capabilities, eligible capacity, and healthy runtime,
storage, network policy, and control-plane connection. A registered or
heartbeating worker can still be ineligible for work. Repository source transfer,
runtime image pins, and provider execution authority are separate prerequisites.
Keep these checks bound to the exact release and worker; see
[release status](/docs/release-status).

## Add a worker on another host
1. On the management server, run the installer once with
   `--remote-workers <CIDR>`, naming the address range your workers connect
   from. The installer publishes the mutual-TLS worker channel and a public
   onboarding profile, both reachable only from that range. It then prints the
   worker command and the Worker CA SHA-256. Allow the same port in any cloud
   firewall in front of the server.
2. On the worker host, from a source checkout that matches the server's
   release, run `sudo ./scripts/install-worker.sh --server https://<host>`.
   You can pin the CA with `--worker-ca-sha256 <hex>`.
3. When the host checks pass, the installer asks for a token at a hidden
   prompt. Generate it in **Runtime workers > Add a worker**. The token is
   single-use and never accepted as an argument or file.
4. The worker enrolls, starts its service and reports its first heartbeat. It
   then shows as **Awaiting admission**. Review it and admit it explicitly;
   nothing is scheduled before that.

Rerunning the installer on an enrolled host restarts the same identity and does
not enroll again. This flow is implemented, and its live acceptance on a fresh
remote host is still pending.

## Operations
- Keep workers disposable and replaceable.
- Separate pools when repositories require different network or trust boundaries.
- Send worker logs to your normal operations system.
- Drain a worker before maintenance so active work remains attributable.
