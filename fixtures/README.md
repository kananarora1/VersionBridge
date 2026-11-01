# Migration fixtures

These directories reserve the fixture suite defined in the development plan. They contain scenario guides only; no sample application or passing expected manifest has been implemented yet.

Phase 0 must add a reproducible sample repository, pinned dependency/toolchain metadata, a versioned `expected.json`, and a baseline source fingerprint for each scenario. The manifest must define expected finding IDs/classifications, allowed touched paths, semantic diff assertions, verification outcome, and prohibited mutations. Never run intentionally hostile fixtures on the host.

- [Web / Jakarta](sb27-web-jakarta/README.md)
- [Data / JPA](sb27-data-jpa/README.md)
- [Multi-module Maven](sb27-multi-module/README.md)
- [Manual Security migration](sb27-manual-security/README.md)
- [Blocked dependency](sb27-blocked-dependency/README.md)
- [Already-target no-op](sb30-noop/README.md)
- [Unsupported input](non-maven/README.md)
- [Malicious build](malicious-build/README.md)
