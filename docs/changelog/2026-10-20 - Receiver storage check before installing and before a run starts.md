# Receiver storage check before installing and before a run starts

The product pages now describe the receiver storage check that product
`a3629c30` added to both installers and to the run preflight. Verified against
`infra/deploy/receiver-storage.py` (`host_check`, `capacity_record`),
`infra/deploy/receiver-storage-check.py`, `infra/deploy/receiver-storage-supervisor.py`,
`scripts/install.sh`, `scripts/install-worker.sh` and
`packages/contracts/src/api/receiverStorageCapacity.ts`.

## Changed

- **Host requirements** (runtime workers, quickstart, release status): every
  host that runs a worker, including the control-plane host when the server
  installer continues into its same-host worker, needs `/var/lib` on ext4 on a
  local disk of at least 89.8 GB with 4096-byte blocks and at least 89.8 GB
  available: the 64 GB receiver storage image plus a 24 GiB reserve. A host
  whose image is already allocated needs only the reserve.
- **Installation** (runtime workers, troubleshooting, upgrade): both installers
  check this before they stop or change any service, with no option to skip
  it, and refuse with one sentence and its code. A server that cannot hold it
  installs only the control plane with `--no-worker`.
- **Operations** (runtime workers, data retention, troubleshooting, readiness
  reason codes): the worker's receiver storage service re-measures every 30
  seconds and writes `/var/lib/branchrunner-receiver-view/capacity.json`. A
  refusal shows a **Receiver storage** line under **Runtime workers**, blocks
  the run preflight's **Worker capacity** row with `receiver_storage_<code>`
  and makes run start skip the worker; it clears within about a minute of
  freeing the space. The readiness page lists every code and the commands to
  read the measurement or run the check by hand on both host layouts.
