# Reconcile product pages with the V1 candidate

The public product pages now describe what the V1 candidate actually implements and what is
still pending.

**Added**

- `release-status.md` and `licensing.md`: candidate status, limitations and licensing.

**Changed**

- Quickstart: Ubuntu 24.04/26.04, at least 8 GB RAM and 60 GB free disk because images are
  built on the host; no public container images until a stable version.
- Runtime workers: remote enrollment with `install.sh --remote-workers` and
  `install-worker.sh --server`.
- Agent providers, workflows, validation/approvals and contributing pages follow the candidate
  (OpenAI-compatible and Azure targets, Anthropic connections, preserve-on-cancel publication).
  Live acceptance is still pending for every configuration.
