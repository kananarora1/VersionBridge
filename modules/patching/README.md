# Patch planning and generation

Build human-reviewable plans from selected findings and evidence. Prefer deterministic transforms; keep model-based generation behind an adapter.

- `src/`: plans, deterministic transforms, model adapter, prompt versions, diff validation, disposable-workspace apply.
- `tests/`: selected-only changes, evidence linkage, path/range allowlists, minimality, invalid diff rejection.

Depend on domain contracts and injected model/workspace ports. Record finding, recipe, evidence, model/prompt, and diff identities. Generation never writes the original source tree. Verification is dispatched through the verifier boundary.
