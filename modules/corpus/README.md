# Documentation corpus

Manage allowed official sources, version/revision pins, normalization, section hierarchy, provenance, checksums, and immutable corpus releases.

- `src/`: source manifest ingestion, normalization, document/section extraction.
- `tests/`: version isolation, anchor stability, checksums, source-policy behavior.

Depend on domain contracts and storage ports. Distinguish original source text from derived index passages. The retrieval module consumes published corpus releases; it does not silently fetch moving documentation.
