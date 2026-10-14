# Claude Sign in again, several accounts and execution blockers

The product pages now describe how a Claude subscription is renewed and when it
can run agents, as 1.0.0 ships them. Verified against the product repository at
`7d193d37`: its changelog entries of 2026-10-13 (Sign in again on a Claude
connection, several Claude accounts per installation, Claude subscriptions
report execution availability, failed Claude operations leave nothing behind,
Claude plan usage records the reported windows, Claude passes run past the
worker lease window), `packages/contracts/src/api/providerConnections.ts`, the
renewal route and the web source of the Claude card, the Sign in again dialog
and the execution blocker texts.

## Changed

- **Renewing a Claude token** (agent providers, troubleshooting, readiness
  reasons, secrets and permissions, data retention, release status): **Sign in
  again** creates a new one-year token on the same connection. The connection,
  its agent profiles and its plan usage history stay, and the account may
  change. Any provider administrator with a recent authenticator sign-in can
  do it, also after the token expired. The pages no longer say that renewing
  means removing the sign-in and adding a new connection; that is now only the
  path after the organization revoked provider credentials, or after a
  removal.
- **Signing administrator** (agent providers, troubleshooting): the
  administrator who signs a Claude connection in, first or again, is the only
  one whose runs can use it and the only one who can verify it. Sign in again
  is refused while Claude operations on the connection are still running.
- **Plan usage** (agent providers): the 5-hour and weekly windows are always
  listed; a window missing from the latest reading shows "unknown", never 0 %.
- **Removal** (agent providers, data retention): a removed Claude connection
  cannot be signed in again; removing one never touches another Claude
  connection.

## Added

- **When a Claude subscription can run agents** (agent providers): the
  conditions for **Execution available** and a table of every
  `executionBlockers` code, what the card says and the one action that fixes
  it. Readiness reasons lists the codes and links to the table; troubleshooting
  and the quickstart point to it.
- **Several Claude accounts** (agent providers, data retention, release
  status): an installation can hold several Claude subscription connections,
  each with its own token and custody records.
- **Claude pass length** (agent providers): a pass runs until the profile's
  time limit, the 15-minute session maximum, its session approval or 10
  minutes before the token expires, whichever comes first; a lost worker lease
  stops it.
- **Claude operation files** (data retention): each operation's staged home and
  workspace under `/run/branchrunner/claude-subscription-native-operations` is
  erased once the operation has settled and its stop is proved, including a
  failed one.
