# Retrieval

Retrieve code and documentation evidence with a measured flat lexical+dense baseline and optional hierarchy-aware expansion.

- `src/`: query construction, lexical/dense adapters, rank fusion, structural neighbors, reranking, trace construction.
- `tests/`: version filters, fusion behavior, evidence IDs, abstention, and provider failures.

Depend on domain contracts and index/model ports. Keep the baseline available for every experiment. Preserve candidate ranks, selected evidence, corpus version, and retriever version. This is the STAIR-inspired adaptation; a trained generative STAIR router is future research. Experiments belong in [evals/](../../evals/README.md).
