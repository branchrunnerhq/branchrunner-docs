# Supported platforms are Ubuntu 24.04, Ubuntu 26.04 and Debian 13 on amd64

The product pages now name exactly the platforms that 1.0.0 supports: Ubuntu
24.04 LTS, Ubuntu 26.04 LTS and Debian 13, on amd64 (owner decision of
2026-10-15). They match the product installers at `6ee09c12` and `223495b7`,
which accept exactly these three and refuse every other platform before
anything is installed.

## Changed

- **Release status:** the supported-configurations table has one row per
  platform: Ubuntu 26.04 LTS, Ubuntu 24.04 LTS and Debian 13 (trixie), all on
  amd64. Each row is "Supported" and lists the release acceptance that is
  recorded before 1.0.0 is published; no row claims to be verified yet. The
  refused platforms are named (Ubuntu 22.04 and interim releases, Debian 12 and
  other Debian releases, arm64 and every other architecture, Arch Linux,
  AlmaLinux, FreeBSD), with the reason Debian 12 is refused: its AppArmor 3.0
  parser cannot read the worker's profile pack, which is written for AppArmor
  ABI 4.0. A new known limitation says 1.0.0 supports three platforms on amd64
  only.
- **Quickstart:** the server prerequisite names the three platforms on amd64
  and says arm64 is not supported in 1.0.0; the installer paragraph says it
  refuses every other platform, including Ubuntu 22.04, Debian 12 and arm64,
  before it changes anything.
- **Runtime workers:** the host requirement names the three platforms on amd64
  instead of "Linux".

## Removed

- The arm64 preview row and every "amd64 or arm64" claim.
