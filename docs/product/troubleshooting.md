---
title: Troubleshooting
description: Diagnose worker readiness, workspace, provider, validation, and delivery failures.
group: Operate
order: 150
---
## Start with preserved state
Read the run transition, worker logs, failed command, and reference identifier before retrying. A retry should not erase the first failure.

## Common checks
- **Worker unavailable:** verify runtime health, capacity, and control-plane connectivity.
- **Workspace unhealthy:** inspect service health checks, ports, and pinned toolchain versions.
- **Provider rejected:** verify the secret reference, provider quota, and role mapping.
- **Validation failed:** reproduce the exact command inside the preserved workspace.
- **Delivery failed:** verify Git permissions and branch policy; the workspace and evidence should remain available.
