# Runtime image prerequisite and separate-host Codex troubleshooting

## Changed

- **Runtime workers readiness.** The list of separate prerequisites no longer
  names runtime image pins. The images a worker runs come from its admitted
  runtime identity, so the control plane needs no image configuration.
- **Troubleshooting "Agent execution refused".** For protected Codex on a
  worker on another host, the entry now adds the host's own selection:
  `codex-remote-host admit` on the control-plane host and
  `codex-protected --control-plane DOCUMENT --worker-id WORKER_ID` on the
  worker host. Without both, a Codex subscription run is not placed on that
  worker or is refused with `remote_execution_tunnel_unavailable`.
