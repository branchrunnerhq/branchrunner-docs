# Align product pages with V1 publication decisions

The public pages now describe version 1.0.0 as a release candidate and follow
the licensor's 2026-10-08 decisions.

**Changed**

- `licensing.md`: names version 1.0.0, explains that the 2029-10-02 Change Date
  is a placeholder set to three years after 1.0.0's first public release, CLA
  version 1.1 naming Savelkouls Ventures, LLC, closed contribution intake and
  later trademark clearance.
- `contributing.md`: external contributions are not accepted until counsel has
  reviewed the CLA; the steps apply once intake opens.
- `release-status.md`: 1.0.0 release candidate; lists all five provider
  connections; built-in signing, handover and retention custody; the installer
  detects Ubuntu release and architecture and builds BranchRunner images on the
  host, pulling only upstream base images; agent execution is off until the
  owner turns it on.
- `quickstart.md`: release candidate wording, platform detection and on-host
  image builds.
- `agent-providers.md`: a direct Codex run can be pinned to an exact API-key
  revision from the task page as well as the API; Codex subscription uses a
  device-code sign-in and protected execution.
- `secrets-and-permissions.md`: new section on built-in key custody without an
  external Vault.
