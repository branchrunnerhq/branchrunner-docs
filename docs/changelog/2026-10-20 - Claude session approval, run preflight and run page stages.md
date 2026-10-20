# Claude session approval, run preflight and run page stages

The product pages now describe the run behaviour that changed during the DIR-1
golden-path runs of 2026-10-19 and 2026-10-20. Verified against the product
repository at `6a673c3b` (`main`): its changelog entries of those two days, the
run page and preflight web source, the Claude session approval routes and
approval decision rules, the Claude source keeper, the worker shutdown path and
the receiver storage supervisor.

## Added

- **Claude session approval** (agent providers, tasks and runs, quickstart,
  secrets and permissions, troubleshooting): each Claude subscription stage,
  and each of the two Claude Review passes, starts only after its own session
  approval on the run page. The person who started the run, who must be the
  connection's signing administrator, requests it with a reason; an owner or
  administrator approves it with an authenticator sign-in from the last five
  minutes. The decision deadline is 15 minutes, or earlier so that it ends at
  least 10 minutes before the token expires; an expired, rejected or withdrawn
  request is replaced with **Request fresh Claude session approval**.
- **Repository registration for Claude** (agent providers): the source keeper
  registers each imported repository for Claude on the control-plane host
  every 15 seconds and restarts the Claude native services only while idle. No
  operator step; the keeper's journal line is shown.
- **Run page stages** (tasks and runs): the run page shows the run's own
  workflow stages in published order with their durable states.
- **Run preflight** (tasks and runs, readiness reason codes, upgrade,
  troubleshooting): the **Starting** state after a deploy, restart or upgrade,
  its named states, and how the managed Codex login row reads the standing
  verification instead of verifying on every refresh.
- **Stage retry** (tasks and runs): **Retry stopped stages**, an Implement on a
  Codex subscription that changed no files, and a Claude start refused before
  any session ran.
- **Secret scan** (validation and approvals): what completion readiness scans,
  that only a policy match fails, the streamed scan of long lines and large
  JSON, unavailable content, and the 8 MiB and 24 MiB capture limits.
- **Clean worker drain on upgrade** (upgrade, readiness reason codes,
  troubleshooting): the refusal "The worker stopped without a clean drain" and
  the journal lines that name its causes.
- **Receiver storage** (runtime workers, quickstart, release status, data
  retention): runs need a 64 GB receiver storage image on their worker host,
  created only when `/var/lib` is ext4 with 64 GB plus 24 GiB free.

## Changed

- **Claude pass deadline** (agent providers): the approval's decision window no
  longer ends a pass that has started; cancelling the run or withdrawing the
  approval still stops it, and a lost lease fails the stage for a retry.
- **Plan approval** (workflows, quickstart): the Plan stage waits for a person
  only when its **Plan approval** is **Wait for human plan approval**; the
  starter template continues by itself.
