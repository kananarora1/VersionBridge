# Cross-cutting tests

Module-local tests live beside their module. These directories cover boundaries and complete workflows:

- `contracts/`: API schemas, verifier protocol, result/artifact contracts.
- `integration/`: persistence, indexes, model stubs, and cross-module behavior.
- `e2e/`: automated snapshot-to-report flows, fixture assertions, original-tree integrity.
- [`manual/`](manual/README.md): mandatory recorded P0-P5 human checkpoints.

Use deterministic provider stubs for normal CI. External-provider evaluation requires a separately configured fixture lane. No executable tests have been added in this scaffolding step.
