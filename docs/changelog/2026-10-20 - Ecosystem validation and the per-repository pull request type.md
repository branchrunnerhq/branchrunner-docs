# Ecosystem validation and the per-repository pull request type

The product pages now describe validation and delivery as 1.0.0 ships them.
Verified against the product repository at `1d71c7f5`: `docs/validation.md`,
`packages/contracts/src/api/validationEcosystems.ts`,
`validationDeclarations.ts` (D17, `59e7d597`), and the repository publication
policy (`defaultPullRequestType`, the **Pull requests** panel).

## Changed

- **Validation** (validation and approvals, release status, troubleshooting,
  MCP and tools): validation is no longer described as npm only. The pages list
  the supported ecosystems (npm including workspaces, pnpm 9 and 10, Yarn
  classic, zero-install Yarn Berry, Python with hashed lockfiles and wheels
  only, Go modules) and their checks, the baseline hygiene and documentation
  checks, worker-prepared dependencies from admitted hosts, `dependency-audit`,
  declared checks with up to four disposable services, and the projects that
  are reported as unsupported.
- **Pull request type** (quickstart, validation and approvals, workflows,
  release status, introduction, secrets and permissions, tasks and runs,
  troubleshooting, webhooks, MCP and tools, data retention): BranchRunner opens
  a pull request as a draft or ready for review as the repository's pull
  request policy sets. A new repository defaults to drafts, or to ready for
  review for a private repository owned by a personal GitHub account. The
  repository page's panel is **Pull requests**.
