# Public documentation for V1

The product pages now describe the BranchRunner 1.0.0 release candidate as it is
built: installation, supported features, known limitations, security reporting
and licensing. Anything not yet proven live is stated as a limitation.

## Added

- **Security** page: the `security@branchrunner.com` reporting route, disclosure
  window, supported versions, the isolation model, and which protections are
  still being verified.

## Changed

- **Quickstart** follows the real path: server installer, owner setup and
  automatic activation, single-host or remote worker (with opt-in `--verify`),
  protected execution selection, then the first-run checklist from GitHub App
  setup to a draft pull request. It states the measured upgrade times and that
  the 1.0.0 installer is not published yet.
- **Release status** lists what 1.0.0 includes and its known limitations: the
  end-to-end draft-PR run still being proven, pending acceptance for all 16
  configurations, unmeasured fresh-install time, npm-only validation, the
  default OpenAI review policy, same-agent failover, cost upper bounds, and
  skills and MCP servers not yet delivered.
- **Agent providers** covers subscription and API-key connections, agent
  profiles with drag-and-drop connection priority, routing modes, plan usage,
  automatic failover, theoretical and effective cost, and the review default.
- **Workflows**, **Validation and approvals**, **Runtime workers**, **Workspace
  environments**, **Tasks and runs**, **Secrets and permissions**, **Backup and
  recovery**, **Upgrade** and **Troubleshooting** describe the six stages,
  detected npm validation, signed evidence, the draft pull request policy,
  worker installation, operator-owned backups with retention on by default,
  clean installs from pre-releases and where to find logs and readiness.
- **MCP and tools**, **API** and **Webhooks** no longer promise unreleased
  features: MCP servers and skills are not delivered to runs, there is no stable
  public API, and there is no public outbound webhook contract.
- **Licensing** drops the "three years after first public release" rule for
  1.0.0 in favour of its fixed Change Date 2029-10-01, and moves to order 155 so
  no two pages share an order.
- **Introduction** describes the six-stage model and where components run.

## Removed

- The statement that the Change Date is provisional.
