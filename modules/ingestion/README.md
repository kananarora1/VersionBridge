# Repository ingestion

Read immutable source snapshots, resolve supported Maven metadata, parse Java/XML source ranges, and extract module/file/symbol hierarchy.

- `src/`: eligibility checks, snapshot readers, parsers, exclusions, structural extraction.
- `tests/`: source ranges, parent/property resolution, multi-module graphs, exclusions.

Depend on domain contracts. Reading a POM must not execute its lifecycle, extensions, wrapper, or project scripts. Unresolved parents/profiles must be reported rather than guessed. Fixtures live in [fixtures/](../../fixtures/README.md).
