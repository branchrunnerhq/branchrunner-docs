# Product pages follow the one-year Claude token, Settings and review signing

The product pages now describe the Claude subscription sign-in, the
Integrations and Settings pages, plan usage and review signing as 1.0.0 ships
them. Verified against the product repository's changelog entries of 2026-10-12
and 2026-10-13, `docs/runbooks/execution-selection.md`,
`docs/self-hosted-quickstart.md`, `docs/runbooks/credential-revocation.md` and
the web source of the Integrations, Settings and first-run checklist pages.

## Changed

- **Claude subscription sign-in** (agent providers, release status, quickstart,
  data retention, secrets and permissions, readiness reasons, troubleshooting):
  a sign-in has the official Claude Code CLI create a token with
  `claude setup-token`, valid for one year, with the same link and code paste.
  The connection shows "valid until" and a renewal notice in its last 30 days;
  the token carries no account email or plan type; **Verify** needs no new
  sign-in and only the administrator who signed in can run it. An expired
  token reads **Needs attention**; signing in again means removing the sign-in
  and adding a new Claude subscription connection. BranchRunner cannot revoke
  the token at Anthropic.
- **Claude subscription runs on the control-plane worker only** (release
  status, runtime workers): the limitation uses the release wording and adds
  that the token never leaves the control-plane host.
- **Integrations and Settings** (agent providers, secrets and permissions,
  quickstart, release status): connection cards open their details on click and
  every action opens a dialog. Keys are checked with **Verify**, removed with
  **Remove key**, and fees and discounts are set with **Plan value**. Admitted
  provider hosts, protected credential enforcement and API-key credential
  approval are under **Settings**.
- **Plan usage** (agent providers, release status): Codex and Claude cards show
  the provider's plan windows after sign-in, **Verify** and each run. The claim
  that Claude reports usage only past a warning threshold is gone.
- **Review signing** (quickstart, agent providers, validation and approvals,
  workflows, upgrade): the policy is derived from the published reviewer
  profiles that workflow Review stages run; nothing is chosen under **Runtime
  workers**. With `--signing-approval owner` the owner approves the review
  signing proposal on **Home** and evidence signing under **Runtime workers**.
- **Separate-host Codex troubleshooting**: when the worker installer did not
  select remote execution, run `codex-protected --control-plane` on the worker
  host, then admit the host for subscription execution. Runtime workers shows
  the full withdrawal command and notes that `none` on the worker host keeps
  later installer runs from selecting it again.
- **Upgrade**: the server installer upgrades the enrolled same-host worker
  itself; subscription connections stay signed in across upgrades; open pages
  reload themselves.
- **Webhooks**: repository grants are rechecked every 4 minutes.
- **Readiness reasons**: the dated "added in October 2026" section is now
  "Other reason families".

## Removed

- The "Sign out of Codex" removal row, which no longer matches an action in the
  product; Codex and Claude sign-ins are both removed with **Remove sign-in**.
