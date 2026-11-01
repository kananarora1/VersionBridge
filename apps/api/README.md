# API application

HTTP entry point and run orchestration for the migration workflow. Implement the `/v1/runs`, findings/evidence, patchset, verification, and artifact endpoints specified in [the PRD](../../docs/PRD.md).

- `src/`: routes, request/response mapping, composition, background-job dispatch.
- `tests/`: application-level validation, idempotency, and state-conflict tests.

Delegate analysis, retrieval, patching, and verification to modules. Authentication and configuration belong at this boundary; the API must not execute repository builds. Framework/bootstrap files will follow the Phase 0 stack decision.
