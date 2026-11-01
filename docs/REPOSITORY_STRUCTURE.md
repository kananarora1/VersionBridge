# Repository structure and module boundaries

The scaffold follows the [phased development plan](DEVELOPMENT_PLAN.md). It reserves application, module, fixture, evaluation, infrastructure, and testing locations without choosing a runtime framework or adding application behavior.

## Dependency direction

Applications compose modules. Domain entities and ports are the core; ingestion, corpus, retrieval, rules, patching, verification, and storage use those contracts. Infrastructure adapters implement ports. Modules must not import an application entry point or rely on a global service container.

The API and CLI coordinate a run. Repository ingestion reads snapshots. Analysis creates findings; retrieval attaches approved evidence; patching creates selected, reviewable diffs; the separate verifier worker executes build jobs inside the sandbox. Storage persists run events, evidence traces, and artifacts. Inject model/search/storage/runner adapters rather than coupling domain logic to a provider.

## Directory conventions

- `apps/<name>/src/` and `tests/`: process entry points and their boundary tests.
- `modules/<name>/src/` and `tests/`: implementation and focused behavior tests.
- `modules/storage/migrations/`: ordered database changes.
- `fixtures/<scenario>/`: future sample input repository, scenario README, expected manifest.
- `evals/datasets/`, `experiments/`, `reports/`: versioned labels, experiment definitions, reviewed aggregate results.
- `tests/contracts/`, `integration/`, `e2e/`: tests crossing a single module boundary.
- `tests/manual/records/`: sanitized manual checkpoint evidence.
- `infra/`: runner, persistence, and observability deployment configuration.
- `config/`: documented non-secret configuration and policy examples.
- `scripts/`: implemented developer operations with documented side effects.
- `docs/adr/`: architectural decisions and their rationale.

Small `.gitkeep` files retain otherwise empty directories in Git. Replace them when real source or tests arrive. The guides in fixture directories are scenario specifications, not runnable fixture repositories.

## First implementation tasks

1. Record the control-plane stack, parser, persistence/index, model adapter, and verifier-runtime choices using the [ADR template](adr/0000-template.md).
2. Define domain IDs, run transitions, result/error vocabulary, and the worker protocol.
3. Implement reproducible fixture applications and meaningful expected manifests.
4. Prove source immutability and isolation, then record the mandatory P0 checkpoint.
5. Build read-only Maven/Java analysis as the first user-visible vertical slice.

Use the PRD for scope and the development plan for acceptance criteria. No build, test suite, container runtime configuration, or Phase 0 gate is claimed as implemented by this scaffold.
