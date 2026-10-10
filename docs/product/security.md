---
title: Security
description: Report a vulnerability, understand the isolation BranchRunner applies, and see which protections are still being verified.
group: Operate
order: 145
---
## Report a vulnerability

Email `security@branchrunner.com`, the contact published in
[security.txt](https://branchrunner.com/.well-known/security.txt). Never put a
finding in a public issue, pull request, discussion or commit message. Once the
public source repository exists, GitHub private vulnerability reporting will
also be available.

Include the version or commit you tested, how it was installed, the boundary
you crossed (tenant isolation, worker isolation, egress, credential custody,
identity or evidence integrity), reproduction steps and the impact you can
demonstrate. Test only against an installation you own or operate.

The disclosure window is 90 days from acknowledgement, or until a fix is
generally available, whichever comes first. Reports are accepted without
signing the Contributor Licence Agreement, and reporters are credited unless
they ask not to be. The product's `SECURITY.md` has the full policy, including
scope and a safe-harbour statement that is still pending legal review.

Only the most recent published version receives fixes. Earlier `0.1.0-pre.*`
versions get no backports.

## Isolation model

BranchRunner treats repository content, agent output and model behaviour as
untrusted input.

- Coding agents and validation run on workers in containers as non-root users,
  with all capabilities dropped, `no-new-privileges`, enforcing AppArmor
  profiles and a BranchRunner seccomp profile.
- Egress from protected containers is denied by default; agents reach their
  provider through BranchRunner's egress controls. Validation checks have no
  network access.
- The API, database and internal services listen on loopback. Host nginx is the
  only public application listener, and the firewall opens only SSH, 80 and
  443, plus the worker channel for a named source range when you enable remote
  workers.
- Workers connect over mutual TLS with certificates from the installation's own
  Worker CA, and an administrator admits each worker explicitly.
- Provider credentials, subscription logins, GitHub App keys and signing keys
  stay in root- or service-owned custody on your server.
- Local sign-in uses a password plus an authenticator app (TOTP). Publication
  approval needs a recent authenticator sign-in.

## Still being verified

Host-level checks of these controls pass on the development host. Proof that
they hold for workloads during a real run, and exercises of revocation, worker
replacement, interrupted execution and cleanup, are part of the pending release
acceptance. See [release status](/docs/release-status).
