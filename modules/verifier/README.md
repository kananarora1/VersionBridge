# Verification

Own the isolated-runner protocol, command profiles, resource limits, cancellation, Maven result parsing, and outcome classification.

- `src/`: runner adapter, job contract, command profiles, result/log parsers.
- `tests/`: test-report failures despite zero exit, timeouts, dependency blocks, log redaction, cleanup.

Depend on domain contracts and runner/storage ports. Treat all repository code, Maven plugins, and wrappers as untrusted execution inputs. A configured test command does not by itself constrain Maven plugin behavior; isolation must contain it. Runner deployment belongs in [infra/runner/](../../infra/runner/README.md).
