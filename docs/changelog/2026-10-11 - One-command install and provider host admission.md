# One-command install and provider host admission

## Changed

- **Quickstart and runtime workers: one command on a single server.** The
  server installer now installs the runtime worker on the same server itself.
  After owner setup and activation it runs the release's same-host worker
  installer in the same terminal, which asks for the enrollment token at its
  hidden prompt. The pages no longer tell you to run a second command. They
  describe when you still run it yourself: after `--no-worker`, without a
  terminal, or when the installer stopped before the worker step, using the
  printed `sudo bash /opt/branchrunner/current/scripts/install.sh worker`.
  A worker on another host still runs the published installer in `worker`
  mode with `--server`.

## Fixed

- **Agent providers: who admits a provider host.** Any owner of the
  organization admits a custom OpenAI-compatible API host, not only the
  installation owner.
