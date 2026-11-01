# Migration rules

Define supported Spring Boot 2.7 to 3.0 findings and approved migration recipes, including version preconditions and manual-review classifications.

- `src/`: rule registry, analyzers, recipe definitions, precondition checks.
- `tests/`: supported/unsupported pattern detection, false positives, already-target no-op.

Depend on domain contracts and parsed snapshot data. Start with Boot/Java build properties and allowlisted Jakarta patterns. Do not globally replace `javax`; Java SE namespaces such as `javax.sql` and `javax.crypto` must remain unchanged. Require explicit evidence and recipe scope for an automatic fix.
