# Infrastructure

- [`runner/`](runner/README.md): verification isolation, image/toolchain, cache policy, limits.
- [`database/`](database/README.md): local database/index services and non-secret defaults.
- [`observability/`](observability/README.md): structured logs, metrics, traces, retention.

Deployment files follow Phase 0 architecture decisions. This directory does not currently start services, execute builds, or create cloud resources.
