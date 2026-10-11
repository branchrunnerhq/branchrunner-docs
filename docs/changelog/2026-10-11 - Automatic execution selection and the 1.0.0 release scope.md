# Automatic execution selection and the 1.0.0 release scope

The product pages now describe 1.0.0 as released and match the setup the
installer performs today: nothing is selected by hand on the control-plane host,
credential enforcement is on from the start, and a separate worker host needs
one owner admission for Codex subscription turns. Verified against the product
repository's `README.md`, `docs/releases/1.0.0.md`, `docs/release-scope.md`,
`docs/self-hosted-quickstart.md`, `docs/runbooks/execution-selection.md`,
`docs/runbooks/worker-installation.md` and `scripts/install.sh`.

## Changed

- **Release status** follows the product release scope: the supported-platform
  table (Ubuntu 26.04 and 24.04 LTS amd64 live-verified, arm64 a preview the
  installers accept), the host size, and "supported but not yet live-verified"
  for the other combinations. The release-candidate, "16 configurations" and
  5-minute wording is gone; the installation time is published as measured, and
  installation never runs test suites.
- **Host size** (release status, quickstart): at least 8 GB RAM and 2 CPU cores,
  which the server installer enforces, and 60 GB free disk; 16 GB RAM and
  4 CPU cores are recommended. Below 16 GB the installer builds the protected
  images two at a time, and a smaller host may hold only one run at a time.
- **Protected execution is selected automatically** (quickstart, agent
  providers, troubleshooting, upgrade, readiness reasons): on the control-plane
  host `codex-protected` and `claude-protected` are selected once the worker
  installer has attested or prepared the host, upgrades keep both, and `none`
  is a recorded opt-out. API keys need no execution selection, and the direct
  `codex` and `claude` adapters are retired (`adapter_retired`).
- **Runtime identity admission** (runtime workers, upgrade, quickstart): on the
  control-plane host an identity whose only change is release-built images is
  admitted automatically; other changes and separate worker hosts still need
  **Admit runtime identity**. Workers no longer pin images on the control plane.
- **Protected Codex on another host** (runtime workers, release status,
  quickstart, troubleshooting): the worker installer selects remote execution
  itself and the owner chooses **Admit for subscription execution**. The removed
  `codex-remote-host admit` and `document` commands are no longer documented.
- **Credential enforcement** (quickstart, secrets and permissions): a fresh
  installation starts with enforcement on, so the first-run checklist has no
  enforcement step; only installations upgraded from an earlier default turn it
  on under **Integrations**.
- **Website links**: the changelog and use-case links point at
  `https://branchrunner.com/changelog` and `https://branchrunner.com/use-cases`,
  so they also resolve when the page is read on GitHub.
- Introduction, licensing, workflows, validation and approvals, and agent
  providers no longer call 1.0.0 a release candidate or describe acceptance as
  pending.
