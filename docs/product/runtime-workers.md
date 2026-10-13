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
work. Repository source import and provider execution authority are separate
prerequisites. The images a worker runs come from its admitted runtime
identity, so the control plane needs no image configuration. The run-start
check reports each missing item.

## Host requirements

The host needs Linux with AppArmor enforcing, Docker Engine 24 or newer, at
least 20 GiB of free disk and 100,000 free inodes, and a clock within 5 seconds
of the control plane. Docker Desktop cannot be used.

## Install the worker

The worker installs exactly the source the control plane runs. On a server
installed from a published release, use one of the two published worker
commands. Both run the release's `install.sh` in worker mode, with the installer
and the source archive pinned by SHA-256. They differ only in `--server`:

```bash
# On the control-plane host itself:
wget --https-only -qO /tmp/branchrunner-install <release URL>/install.sh && echo '<installer SHA-256>  /tmp/branchrunner-install' | sha256sum --check --strict && sudo bash /tmp/branchrunner-install worker --source-sha256 <source archive SHA-256>

# On another host:
wget --https-only -qO /tmp/branchrunner-install <release URL>/install.sh && echo '<installer SHA-256>  /tmp/branchrunner-install' | sha256sum --check --strict && sudo bash /tmp/branchrunner-install worker --server <control-plane URL> --source-sha256 <source archive SHA-256>
```

`<release URL>` stands for the release's address, which is published with the
release notes, and `<control-plane URL>` for your server's HTTPS address. Do not type these commands
yourself. On a single server you normally need neither: the server installer
runs the same-host worker installer itself after owner setup (see
[Add a worker on the same server](#add-a-worker-on-the-same-server)). The server
installer prints the exact commands for your release: the same-host command
under "Next steps" when run with `--no-worker`, and the remote command with
`--remote-workers`. **Runtime workers > Add a worker** shows both, under
**On this server's own host** and **On another host**. Neither
needs a Git checkout. It downloads the release's `SHA256SUMS`, source-commit
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
host and checks each image's exact ID, role and non-root user. Nothing is
pulled from a registry. The worker reports those image IDs in its runtime
identity, which you admit; no image pins are configured on the control plane.
The installer never runs test suites; `--verify` adds the full host
verification suite, which takes about two hours and is meant for developers
and release candidates. A rerun on an enrolled host keeps the same identity
and never enrolls again.

## Add a worker on the same server

The server installer installs this worker itself, so a single-server
installation takes one command. After the server is installed, it prints the
setup link and waits for the browser owner setup, for at most the link's
two-hour lifetime, with a progress line every minute. It then waits for the
server's activation and runs this release's same-host worker installer in the
same terminal. When that installer asks for the enrollment token, generate one
under **Runtime workers > Add a worker** and paste it at the hidden prompt. The
server installer never creates or reads the token.

Run the same-host command yourself only when the server installer did not run
it. With `--no-worker` it prints that command under "Next steps". When no
terminal was attached, or the installer stopped before the worker step
finished, the control plane stays installed and the installer prints the
command that resumes,
`sudo bash /opt/branchrunner/current/scripts/install.sh worker`. Run either on
the control-plane host after owner setup. A rerun of the server installer skips the
worker step while this host's worker is installed and enrolled from the same
release.

The same-host worker needs no `--remote-workers`, no published onboarding
profile and no firewall rule. Without `--server`, the installer takes the
expected release from the control plane on this host and checks that it agrees
with the installation's state. It joins over the loopback worker channel
(`127.0.0.1:5181`) and reads the installation, Worker CA and owner organization
from the local control plane. It refuses a control plane without an owner, a
loopback channel that does not verify, and a local release record that
disagrees with the installation or the published release. On a control plane
installed from a Git checkout, run `sudo ./scripts/install-worker.sh --local`
from a clean checkout of the same commit instead.

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
identity. Every capability, including validation, source repair and archive
inspection, uses the images of the worker that runs it, so the control plane
needs no image configuration for any of them.

Admission is checked per capability:

- **Host and runtime fields** (platform, Docker version, security packs,
  controlled egress and the protected runtime) apply to the whole worker. If an
  upgrade changes one of them, **Runtime workers** shows that the worker reports
  a changed runtime identity, with both identities side by side, and every
  capability on that worker is refused until an owner or administrator chooses
  **Admit runtime identity**.
- **Each image** applies only to its own capability: diff analysis, validation,
  source repair or archive inspection. If one of them becomes ready after
  admission, or reports a different image, only that capability is refused and
  **Runtime workers** names the image that awaits admission. The worker's other
  capabilities keep running. A capability can only ever run the image admitted
  for it.

A rerun from the same sources keeps the same image IDs and therefore the same
identity.

The worker on the control-plane host is the exception for upgrades whose only
change is images: when every changed image is one the installers built from the
release the control plane runs, the new identity is admitted automatically and
recorded as **BranchRunner (release images)** with an audit event
(`sudo journalctl -u branchrunner-release-image-admission` shows the outcome).
Any other change, such as the security profiles, the Docker version or the
keys, and every change on a worker on a separate host, still needs **Admit
runtime identity**.

## Agent stages on another host

In 1.0.0, a worker on another host runs Codex subscription stages, once the
owner has admitted it for subscription execution as described below, and
API-key stages. Claude subscription stages
run only on the worker on the control-plane host: run placement skips a worker
on another host for any run that uses a Claude subscription connection, with
the reason `claude_remote_execution_unavailable`. A Claude subscription's token
stays in custody on the control-plane host and is never delivered to another
host. If you use a Claude subscription, also install a worker on the
control-plane host.

### Protected Codex on another host

A Codex subscription stage on a worker on another host runs in that host's
protected launcher. The control plane's custody delivers the session credential
over the worker channel; sign-in, refresh and exclusive verification stay on
the control-plane host. Nothing is copied between hosts by hand:

1. **Install the worker.** After enrollment, the worker installer fetches the
   control plane's execution document over the worker channel and, when the
   control plane has protected Codex selected, selects remote execution on the
   worker host itself. The worker opens its tunnels to the control plane within
   a few seconds. If protected Codex was not selected on the control plane yet,
   or the host enrolled later with `br-worker-enrol`, select it on the worker
   host once the control plane is ready:

   ```bash
   sudo bash /opt/branchrunner/current/infra/deploy/branchrunner-configure-execution.sh codex-protected --control-plane
   ```

2. **Admit the worker and its runtime identity** under **Runtime workers**. The
   identity names the host attestation key and the protected Codex images.
3. **Admit for subscription execution.** On the same page, the installation
   owner, signed in with MFA within the last 15 minutes, chooses **Admit for
   subscription execution**. Custody then delivers sessions to that host, and
   refuses any host that is not admitted or whose signed proof names other
   images. The page shows the outcome: `admitted`, or a reason such as
   `owner_approval_invalid`, `host_key_changed` or
   `protected_runtime_not_admitted`.

After an upgrade, admitting the worker's new runtime identity is enough. A new
owner approval is needed only when the host attestation key changes
(`host_key_changed`).

To withdraw the host, the owner chooses **Withdraw** under **Runtime workers**,
or Root runs this on the control-plane host; both stop new sessions, and only a
later owner approval admits the host again:

```bash
sudo bash /opt/branchrunner/current/infra/deploy/branchrunner-configure-execution.sh codex-remote-host withdraw WORKER_ID
```

`none` on the worker host stops its protected Codex stack and
closes the tunnels, and later installer runs leave the host unselected until
`codex-protected --control-plane` is run there again.

## Known limitations

- There is no screen that lists outstanding enrollment tokens; the audit log
  records each issue and revocation.
- A worker on a separate host is supported but not yet live-verified; see
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
