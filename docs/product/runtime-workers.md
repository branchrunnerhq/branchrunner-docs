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
admission, an admitted runtime identity, compatible capabilities, eligible
capacity, and a healthy runtime, storage, network policy and control-plane
connection. A registered or heartbeating worker can still be ineligible for
work. Repository source import, runtime image pins and provider execution
authority are separate prerequisites. The run-start check reports each missing
item.

## Host requirements

The host needs Linux with AppArmor enforcing, Docker Engine 24 or newer, at
least 20 GiB of free disk and 100,000 free inodes, and a clock within 5 seconds
of the control plane. Docker Desktop cannot be used.

## Install the worker

The worker installs exactly the source the control plane runs. On a server
installed from a published release, use the published worker command. It is the
release's `install.sh` in worker mode, with the installer and the source archive
pinned by SHA-256:

```bash
wget --https-only -qO /tmp/branchrunner-install https://branchrunner.com/releases/1.0.0/install.sh && echo '<installer SHA-256>  /tmp/branchrunner-install' | sha256sum --check --strict && sudo bash /tmp/branchrunner-install worker --server https://branchrunner.example.com --source-sha256 <source archive SHA-256>
```

Do not type this command yourself. The server installer prints the exact command
for your release, and **Runtime workers > Add a worker** shows the same command.
It needs no Git checkout. It downloads the release's `SHA256SUMS`, source-commit
record, `RELEASE.json` and source archive, and checks them against each other and
against the release that the server advertises. It then rebuilds the published
commit from the archive and runs that release's worker installer. It refuses a
server on another release, a different archive or a file that does not match
its checksum, and in that case it installs nothing. You can also pin the Worker
CA with `--worker-ca-sha256 <hex>`, using the value the server installer printed.

A control plane installed from a Git checkout advertises that commit instead.
Install its workers from a clean checkout of the same commit, outside
`/opt/branchrunner`, with `sudo ./scripts/install-worker.sh --server <url>`.

By default the installer runs fast installation checks: release identity, a
smoke run of the installed worker and a local host preflight. It builds the
diff-analysis, validation, source-repair and archive-inspection images on the
host, checks each image's exact ID, role and non-root user, and pins that local
image. Nothing is pulled from a registry. `--verify` adds the full product test
suite, which takes about two hours and is meant for developers and release
candidates. A rerun on an enrolled host keeps the same identity and never
enrolls again.

## Add a worker on the same server

The published worker command reads the server's onboarding profile through
the worker channel, which is published only to the `--remote-workers` ranges.
On the same server, include the server's own address in those ranges and run
the printed command there, or follow the manual configuration in the product's
worker installation runbook (`docs/runbooks/worker-installation.md`).

The worker shares a service account with the control panel. The installer
therefore stops those containers, installs the worker, and starts them again
with the API last. The control panel is unavailable for that window. Pass
`--no-stop-control-plane` if you prefer to stop them yourself; the installer
then refuses while they run.

## Add a worker on another host

1. **Open the worker channel.** On the management server, run the installer with
   `--remote-workers <CIDR>[,<CIDR>...]`, naming up to 16 IPv4 address ranges
   that your workers connect from. The channel needs the organization, so a
   first installation records the request and publishes it on the next rerun,
   after owner setup. The installer publishes the mutual-TLS worker channel
   (port 5181 by default) and a public onboarding profile, both reachable only
   from those ranges, with one firewall rule per range. It then prints the
   worker command and the Worker CA SHA-256. Allow the same port in any cloud
   firewall in front of the server.
2. **Run the worker command** on the worker host. Before it installs anything,
   it checks that it can reach the worker channel and that the channel's
   certificate verifies against the Worker CA. If not, it stops at once and
   names the port, before any token is needed.
3. **Paste the token.** When the host checks pass, the installer asks for a
   token at a hidden prompt. Generate the token in **Runtime workers > Add a
   worker** just before you need it; it is single-use and expires after ten
   minutes. For automation, pipe the token from a protected producer to the
   installer's standard input instead. The installer reads it once and keeps it
   only in memory. A token passed as an argument or redirected from a file is
   refused.
4. **Wait for the first heartbeat.** The worker enrolls, starts its service and
   reports its first heartbeat. It then shows as **Awaiting admission**.

Installer exit code `4` means the worker is installed and checked but enrollment
did not complete, for example because the token expired. Run
`sudo br-worker-enrol` to finish it. A refused token is explained on the
terminal, together with whether it was spent.

## Change or withdraw the channel

Rerun the server installer with a new `--remote-workers` list to change the
source ranges; the firewall rules for removed ranges are deleted. The channel's
address and port stay fixed, because enrolled workers connect to them.
`--remote-workers none` withdraws the channel and the onboarding profile, and
workers on other hosts lose their connection.

## Admit the worker

A new worker receives no work until an owner or administrator admits it under
**Runtime workers**. Admission requires a recent sign-in. The
page shows two reports, both measured and signed by the worker:

- **The reported host:** operating system, kernel, mandatory access control
  mode, AppArmor and seccomp pack versions and hashes, trust classes,
  certificate fingerprint and enrollment time.
- **The reported runtime identity:** container runtime and version, AppArmor
  profile version, AppArmor and seccomp pack hashes, the locally built image IDs
  by role, the controlled-egress pair and the identity digest. The worker
  installer prints the same identity on the worker host, so you can compare
  them.

**Admit worker** admits the host and exactly that runtime identity. Workers with
different Docker versions or image IDs are each admitted with their own
identity, and the control plane needs no configuration change for source
import, agent runs or diff analysis on them.

If an upgrade changes what a worker reports, for example a new image or another
Docker version, **Runtime workers** shows that the worker reports a changed
runtime identity, with both identities side by side. Source import, agent runs
and the dependent workspace commands are refused on that worker until an owner
or administrator chooses **Admit runtime identity**. A rerun from the same
sources keeps the same image IDs and therefore the same identity.

## Known limitations

- Validation, source repair and archive inspection still take their images from
  server-wide pins. On a single host the worker installer sets them. For a
  worker whose control plane runs elsewhere, the worker installer prints the
  three values and the command, and the control-plane operator sets them.
- There is no screen that lists outstanding enrollment tokens; the audit log
  records each issue and revocation.
- Live acceptance of a worker on a separate host is still pending; see
  [release status](/docs/release-status).

## Operations

- Keep workers disposable and replaceable.
- Separate pools when repositories require different network or trust
  boundaries.
- Send worker logs to your normal operations system.
- Drain a worker before maintenance. A draining worker still delivers pending
  results so active work remains attributable.
- Decommissioning a worker revokes its certificates, fences its live leases
  and ends their credential sessions. It changes nothing on the worker host;
  remove the host's data under your own procedure.
- After a control-plane upgrade, run the new release's worker command on every
  worker host. The worker channel refuses a mismatched protocol version.
