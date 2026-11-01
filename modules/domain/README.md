# Domain

Own migration runs, repository/document nodes, findings, recipes, evidence references, patchsets, verification outcomes, and run events.

- `src/`: entities, value types, run state machine, ports/interfaces.
- `tests/`: valid/invalid transitions and business invariants.

This module depends on no application, database, model provider, or container runtime. Define repository and runner ports here when needed; infrastructure modules implement them. Refer to the [PRD data model](../../docs/PRD.md).
