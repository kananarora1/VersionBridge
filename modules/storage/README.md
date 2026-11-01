# Storage

Implement persistence ports for run metadata, append-only events, code/document nodes, retrieval traces, patchsets, verifier results, and checksum-addressed artifacts.

- `src/`: database/index/artifact adapters and retention/deletion support.
- `tests/`: transaction boundaries, event ordering, idempotency, artifact integrity, deletion scope.
- `migrations/`: ordered schema migrations after the persistence/tooling decision.

Depend on domain contracts. Keep source/model secrets out of stored traces and logs. Local database deployment settings belong in [infra/database/](../../infra/database/README.md), not in schema migrations.
