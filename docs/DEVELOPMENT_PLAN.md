# AI Software Migration Engineer — Phased Development Plan

**MVP:** Spring Boot 2.7.x → 3.0.x, Java 17, Maven  
**Delivery principle:** every phase ends with a complete manual end-to-end flow test. No phase is complete until its pass/fail gate passes or the scope is reduced and documented.

## 1. Implementation posture

Build a modular monolith first: one API/application process, a relational metadata store, a local/object artifact store, and an isolated verifier worker. Keep parsing, retrieval, patching, and verification behind interfaces so they can later scale independently. Do not begin with a trained STAIR clone. The first usable system is an inspectable hybrid retriever with explicit repository/document hierarchies.

### Suggested stack

- **Control plane:** Java 21 or Python 3.12 service, REST API, PostgreSQL, queue/outbox; choose the team’s strongest operational stack.
- **Code intelligence:** Maven effective-model reader; JavaParser/Spoon or Eclipse JDT; XML parser with source spans; Tree-sitter only where a dedicated parser is absent.
- **Retrieval:** PostgreSQL full-text/BM25-compatible engine or OpenSearch plus a vector store; a single store is preferred initially if it supports metadata filters and hybrid fusion.
- **Models:** embedding model with local/approved hosted provider; patch model behind an adapter; deterministic test double for CI.
- **Isolation:** rootless OCI runtime or hardened VM runner; per-run directory/volume; read-only source mount.
- **Observability:** structured run events, OpenTelemetry traces, redacted logs, prompt/model/corpus version tags.

## 2. Target architecture and module boundaries

```text
api ────── run-orchestrator ────── run-ledger / artifact-store
                 │
       ┌─────────┼─────────┬─────────────┐
       ▼         ▼         ▼             ▼
 repository   docs-corpus  analyzers    patch-planner
 ingestor       manager       │             │
       │            │         ▼             ▼
       └────> structure + hybrid retrieval  diff-validator
                                             │
                                             ▼
                                      isolated verifier worker
```

**Non-negotiable boundaries**

- `repository-ingestor` reads source snapshots only; it never runs project code.
- `patch-planner` returns diffs and never writes the original tree.
- `verifier-worker` is the only component allowed to execute the build; it runs in a new sandbox from a snapshot.
- `docs-corpus-manager` is the only writer of versioned official-document records.
- The model adapter receives redacted excerpts and evidence IDs, not a broad filesystem mount.

## 3. Repository layout and delivery mechanics

```text
asme/
  apps/api/                 # HTTP and orchestration entry point
  modules/domain/           # entities, state machine, interfaces
  modules/ingestion/        # Maven, Java, XML, hierarchy extraction
  modules/corpus/           # docs fetch/normalize/version/governance
  modules/retrieval/        # lexical, dense, structure expansion, fusion
  modules/rules/            # findings and deterministic recipes
  modules/patching/         # plan, LLM adapter, diff policy/apply
  modules/verifier/         # runner client, parsers, limits, artifacts
  fixtures/                 # versioned input repositories + expected manifests
  evals/                    # gold datasets, metrics, experiment configs
  infra/                    # local runner, DB, CI images, policy
  docs/adr/                 # architecture decisions
```

Every change uses: issue/ADR → tests/fixture manifest → implementation → automated checks → mandatory manual E2E checkpoint → release note. Feature flags protect retrieval experiments and LLM patching.

## 4. Fixture suite and expected manifests

Create these tiny, intentionally legible Maven repositories under `fixtures/`. Pin all dependencies and include a Maven wrapper where practical.

| Fixture | Composition | Primary expected result |
|---|---|---|
| `sb27-web-jakarta` | Boot 2.7.18, Java 11, `javax.servlet`, `javax.validation`, MVC test | Detect Java 17/Boot changes and approved Jakarta import changes; patched sandbox test passes. |
| `sb27-data-jpa` | Boot 2.7.18, JPA entity/repository, `javax.persistence`, H2 test | Detect approved JPA import migration; test verifies persistence behavior. |
| `sb27-multi-module` | parent + `api` + `service`, inherited Boot parent | Correct module graph; no duplicate edits; selected module patch verifies. |
| `sb27-manual-security` | Spring Security customization with removed/deprecated patterns | Report manual review only; generate no Security auto-patch. |
| `sb27-blocked-dependency` | build requiring an uncached private/dependency artifact | Verification reports `BLOCKED_DEPENDENCY`, not a false failure/success. |
| `sb30-noop` | Healthy Boot 3.0/Java 17 project | Eligibility says already-target/no-op; zero mutations. |
| `non-maven` | Gradle or missing POM | Cleanly reject as unsupported. |
| `malicious-build` | POM/profile that attempts network/host-sensitive execution | Runner blocks/does not execute unsafe command; original fixture unchanged. |

Each fixture has `expected.json` containing detection IDs, approved/manual classification, expected touched paths, expected diff checksum or semantic assertions, expected verifier status, and prohibited mutations. Store a `README.md` explaining the intentional migration problem. Add mutated variants (moved imports, renamed classes, reordered dependencies) to reduce rule overfitting.

## 5. Data model and state machine

### Relational core

| Table | Minimum columns | Notes |
|---|---|---|
| `migration_runs` | ID, source hash, target, status, corpus version, policy version | Durable run root. |
| `code_nodes` | ID, run ID, parent ID, kind, path/symbol, range, content hash | Hierarchical source representation. |
| `doc_artifacts`, `doc_nodes` | version/publisher/URL/checksum; hierarchy/path/passage | Immutable corpus release. |
| `findings`, `recipes` | rule, location, state, evidence IDs; preconditions/transform | Recipes are version-gated. |
| `retrieval_traces` | query hash, channel ranks, selected evidence | Required to debug quality. |
| `patchsets`, `verification_runs` | diff hash/apply state; runner image, limits, outcome | Immutable artifacts referenced by checksum. |
| `run_events` | sequence, actor, event, payload hash | Append-only audit/state transitions. |

### State transitions

```text
CREATED → ELIGIBLE → INGESTED → ANALYZED → AWAITING_SELECTION
       ↘ REJECTED
AWAITING_SELECTION → PATCH_PLANNED → APPLIED_IN_SANDBOX → VERIFYING → VERIFIED_PASS
                                                           ├────────→ VERIFIED_FAIL
                                                           ├────────→ BLOCKED
                                                           └────────→ TIMED_OUT
Any disposable state → DISCARDED
```

Transitions are idempotent and event-sourced. `VERIFIED_PASS` means the configured command and result parser passed; it does not mean semantic production correctness.

## 6. API contract tasks

Implement these in the order shown, returning RFC 7807-style errors and correlation IDs:

| Order | Endpoint | Essential behavior |
|---|---|---|
| 1 | `POST /v1/runs` | Snapshot reference validation; no build execution. |
| 2 | `POST /v1/runs/{id}/analyze` | Idempotently starts eligibility/ingestion/rules; work is observable. |
| 3 | `GET /v1/runs/{id}/findings` and `/findings/{id}/evidence` | Stable locations, docs/corpus metadata, score breakdown. |
| 4 | `POST /v1/runs/{id}/patchsets` | Requires explicit selected finding IDs and policy acknowledgement. |
| 5 | `POST /v1/patchsets/{id}/apply` | Creates a new workspace, records diff, rejects unsafe scope. |
| 6 | `POST /v1/patchsets/{id}/verify` | Executes exactly one allowlisted profile; records limits/results. |
| 7 | `GET /v1/patchsets/{id}/artifacts` | Redacted logs, report, diff, export instructions. |

Contract tests must cover malformed IDs, repeat requests, state conflicts, unsupported input, worker crash/retry, and no leakage of absolute source paths or secrets in API responses.

## 7. Phased work plan

### Phase 0 — Foundations, threat model, and fixtures (Week 1–2)

**Tasks**

- Write ADRs for source immutability, corpus provenance, verifier isolation, model-provider data handling, and supported-rule taxonomy.
- Define run state machine, database schema migrations, artifact retention/redaction policy, and error contract.
- Create all fixture repositories and `expected.json` manifests from §4.
- Build a deterministic fake patch/model adapter so CI never depends on a model provider.
- Create runner image, resource limits, command allowlist, secret/environment scrubber, and host-network denial test.
- Establish CI: unit, integration, fixture, SBOM/dependency scan, lint, and image scan.

**Acceptance criteria**

- All fixtures are reproducible and checked into version control; expected manifests validate.
- A hash taken before and after every test proves the source fixture is unchanged.
- Unsafe execution test is blocked before project code runs; the event ledger explains why.
- CI can run deterministic tests without a network/model credential.

**Dependencies:** container/VM runner decision, database/artifact storage, legal-approved documentation sources, target OS/CI runners.

**Mandatory manual E2E checkpoint P0 — Safety baseline**

| Item | Procedure | Expected result | Pass/fail/rollback gate |
|---|---|---|---|
| Fixture | `malicious-build`, `non-maven`, clean `sb27-web-jakarta` | Create runs and request analysis/verification only where allowed. | Original tree hashes match before/after; non-Maven is rejected; malicious build is blocked without execution; clean fixture has no modifications. |
| Manual action | Inspect event log, sandbox directory, redacted environment, and returned error. | User can identify why action was blocked and how to discard it. | **Pass:** all assertions. **Fail:** any source mutation, egress, secret exposure, or unclassified error. **Fix:** disable verifier route, preserve artifacts, patch boundary/policy, rerun from fresh snapshots. |

### Phase 1 — Repository ingestion and deterministic analysis (Week 3–4)

**Tasks**

- Implement Maven parent/BOM/property resolution and Java version detection without running Maven lifecycle goals.
- Parse POM XML and Java AST with line/column ranges; create module/path/file/symbol hierarchy.
- Exclude build/generated/vendor output with transparent policy and report exclusions.
- Implement rules: Boot 2.7 eligibility, Java <17, `javax.servlet`, `javax.validation`, `javax.persistence`, target already 3.0, and unsupported Security customizations.
- Add finding schema, finding API, source anchor hashing, rule-level unit tests, and fixture assertions.

**Acceptance criteria**

- `sb27-web-jakarta`, `sb27-data-jpa`, and multi-module fixture findings match expected manifests.
- `sb30-noop` is no-op; Security fixture is manual-review, not auto-fixable.
- Every finding points to a stable source range and a rule ID; no source is edited.

**Dependencies:** Phase 0 gates, Java/XML parser choice, Maven-model compatibility tests.

**Mandatory manual E2E checkpoint P1 — Read-only analysis**

| Item | Procedure | Expected result | Pass/fail/rollback gate |
|---|---|---|---|
| Fixture | `sb27-web-jakarta`, `sb27-data-jpa`, `sb27-multi-module`, `sb27-manual-security`, `sb30-noop` | Create and analyze one run per fixture; open every finding and source anchor. | Correct module ownership/locations; expected patterns found; Security remains manual; Boot 3 gets zero mutable findings. |
| Manual action | Compare API/UI findings to `expected.json`; inspect original fixtures after analysis. | Locations open the intended import/POM/property and tree hashes do not change. | **Pass:** manifest match and read-only proof. **Fail:** missed required finding, auto-fix classification for excluded class, duplicate multi-module finding, or mutation. **Fix:** rollback deployment to Phase 0, add a failing fixture assertion, repair rule/parser, rerun all P1 flows. |

### Phase 2 — Versioned documentation corpus and hybrid retrieval (Week 5–6)

**Tasks**

- Define a signed/approved source manifest for official Spring Boot 3.0 migration docs, Spring Framework 6 docs, and Jakarta API guidance.
- Fetch/normalize only allowed documents; preserve URL, date, version/revision, checksum, heading tree, anchors, and license note.
- Index lexical text, embeddings, hierarchy edges, source identifiers/properties, and doc heading paths.
- Implement baseline flat BM25+dense retrieval, then hybrid reciprocal-rank fusion with metadata/version filters and hierarchy-neighbor expansion.
- Persist retrieval traces and add a retrieval inspection endpoint/UI.
- Build a hand-labeled query/evidence set from fixture findings; do not use a patch success as a retrieval label.

**Acceptance criteria**

- Every finding from Phase 1 has at least one current, version-matched documentation candidate or is explicitly `NO_APPROVED_EVIDENCE`.
- Trace exposes candidates, ranks, channel scores, selected passages, and corpus version.
- Hybrid retrieval matches/exceeds the flat baseline per the PRD quality gate; no unpinned web result is silently used.

**Dependencies:** approved doc manifest, embedding provider/model, search store, Phase 1 findings.

**Mandatory manual E2E checkpoint P2 — Evidence flow**

| Item | Procedure | Expected result | Pass/fail/rollback gate |
|---|---|---|---|
| Fixture | `sb27-web-jakarta`, `sb27-data-jpa` | Analyze, select each Jakarta/Java/Boot finding, open its evidence trace. | At least one official, 3.0-pinned passage supports each proposed recipe; hierarchy path is intelligible; scores/versions visible. |
| Manual action | Deliberately disable a corpus artifact or request a non-existent property migration. | System produces `NO_APPROVED_EVIDENCE` or manual review; it does not fabricate a citation. | **Pass:** correct evidence and abstention behavior. **Fail:** stale/wrong-version/unofficial citation, hidden retrieval, or fabricated support. **Fix:** revert corpus release/feature flag to flat baseline, quarantine source, add gold test, repair normalizer/ranker, rerun P2. |

### Phase 3 — Patch planning and controlled diff generation (Week 7–8)

**Tasks**

- Encode deterministic recipes with preconditions and file/range allowlists: Boot version property/parent, Java target/release, and selected Jakarta imports.
- Implement patch-plan API, grouped review UI/CLI, explanation template, evidence linkage, confidence/abstention rules, and user selection.
- Implement model adapter only as a constrained alternative for recipe-owned edits; include prompt/model/corpus IDs in artifacts.
- Build AST/XML-aware transforms where possible; use unified diff validation as final common interface.
- Reject binary, generated, secret-bearing, out-of-range, or unrelated formatting changes; create a reversible sandbox apply operation.

**Acceptance criteria**

- Selected supported findings generate a minimal diff that applies in a new workspace.
- Unselected/manual findings create no change.
- Every hunk maps to finding/recipe/evidence IDs; error states retain a reviewable plan but no partial source mutation.

**Dependencies:** Phase 2 corpus/evidence, deterministic transform library, model adapter policy, diff parser.

**Mandatory manual E2E checkpoint P3 — Human-reviewed patch flow**

| Item | Procedure | Expected result | Pass/fail/rollback gate |
|---|---|---|
| Fixture | `sb27-web-jakarta`, `sb27-data-jpa`, `sb27-manual-security` | Analyze; select only approved Jakarta + version findings; create and apply a patchset; then repeat with no selections and with the Security finding. | Supported diff changes only allowed POM/import lines in sandbox; empty selection is no-op; Security produces no automated diff. |
| Manual action | Read each hunk, evidence link, and applied-workspace path; compare source fixture hash. | Rationale is sufficient for review; original is untouched. | **Pass:** exact expected changed paths and valid reverse diff. **Fail:** scope creep, patch to unselected finding, invalid diff, source mutation, or missing evidence. **Fix:** discard workspace, disable offending recipe/model flag, add a negative fixture, correct transform/guard, rerun P3. |

### Phase 4 — Sandboxed verification and result reporting (Week 9–10)

**Tasks**

- Implement isolated worker contract, immutable input mount, empty-secret environment, read-only dependency cache, quotas, timeout, cancellation, and cleanup.
- Add command profiles: `mvn -B -DskipTests compile` and `mvn -B test`; choose only one explicitly per run.
- Parse Maven/Surefire/Failsafe output into structured failure categories: compilation, test failure, dependency blocked, policy blocked, timeout, infrastructure error.
- Produce a migration report with diff checksum, command/image/limits, verifier status, remaining findings, and export instructions.
- Create failure-reproduction and worker-crash recovery tests.

**Acceptance criteria**

- Patched supported fixtures are verified with the expected result; known blocked fixture is classified correctly.
- A timeout cannot leave a live sandbox; passing exit plus failing Surefire report is `FAILED`.
- Logs are redacted and original checkout hashes remain unchanged.

**Dependencies:** Phase 3 sandbox apply, runner image/cache policy, CI capacity and artifact retention controls.

**Mandatory manual E2E checkpoint P4 — Full migration verification**

| Item | Procedure | Expected result | Pass/fail/rollback gate |
|---|---|---|---|
| Fixture | `sb27-web-jakarta`, `sb27-data-jpa`, `sb27-blocked-dependency`, a deliberately failing-test variant | Create run → analyze → select → patch → apply → verify → inspect/export or discard. | First two build/test pass in isolated workspace; blocked dependency is `BLOCKED_DEPENDENCY`; failing test is `FAILED`, with useful logs; neither is falsely “migrated.” |
| Manual action | Inspect runner network/secrets, resource limits, reports, and reverse/discard action. | No network/secrets/source mutation; exact command/status/log link shown; discard deletes only disposable artifacts. | **Pass:** all four results and safety proof. **Fail:** false pass, unsafe runner behavior, non-redacted log, orphaned sandbox, or source change. **Fix:** stop pilot release, disable verification/export, preserve forensic artifacts, patch runner/parser, repeat P0 then P4. |

### Phase 5 — Evaluation, hardening, and pilot readiness (Week 11–12)

**Tasks**

- Freeze evaluation split by repository; create a holdout set and metamorphic/mutated variants.
- Run analyzer, retrieval, patch, and full-flow metrics separately; confidence intervals/bootstrap where sample size permits.
- Conduct adversarial tests: prompt-injected comments/docs, path traversal, poisoned POM/config values, huge files, model outage, corpus mismatch, and worker failure.
- Add performance limits, error budgets, retention/deletion verification, accessibility/review usability pass, and pilot documentation.
- Run a human review study on evidence sufficiency and patch minimality; record overrides as labels only with explicit consent.

**Acceptance criteria**

- PRD §4 thresholds and all earlier checkpoint gates pass on frozen fixtures; failure taxonomy is complete.
- Hybrid hierarchy-aware retrieval has a written A/B result versus flat baseline; retain flat baseline if no meaningful gain.
- No critical sandbox, provenance, original-tree-integrity, or prompt-injection failure remains open.

**Dependencies:** all earlier phases, security review, pilot repositories/owners and consent, product sign-off.

**Mandatory manual E2E checkpoint P5 — Release-candidate drill**

| Item | Procedure | Expected result | Pass/fail/rollback gate |
|---|---|---|
| Fixture | All §4 fixtures plus one held-out supported repository | Execute the exact pilot workflow with a new user account/profile, including review, export, and discard. | User can complete a safe verified migration or receive an actionable abstention; dashboards and artifacts agree. |
| Manual action | Re-run an already-target repo, injected/malicious case, and failed verifier; audit audit trail after discard. | No-op/injection/failure classifications are safe; retention and deletion policy operates as stated. | **Pass:** release checklist signed by engineering, security, and product. **Fail:** any red safety gate or metric miss. **Fix:** roll back feature flag to read-only analysis, triage by layer, add regression fixture, rerun affected checkpoint plus P5. |

## 8. Testing strategy and ownership

| Test level | Focus | Owner | Runs |
|---|---|---|---|
| Unit | parser ranges, rules, transforms, ranking fusion, state transitions | module owner | every commit |
| Contract | APIs, worker protocol, artifact schemas, idempotency | platform owner | CI + release |
| Integration | database/index/corpus/runner boundaries | platform + AI engineer | CI nightly |
| Fixture regression | expected findings/diffs/build outcomes | migration engineer | every PR |
| Retrieval evaluation | gold evidence Recall@k/MRR/citation-version correctness | AI engineer | corpus/ranker change |
| Security | egress/secrets/path traversal/prompt injection/sandbox escape | security owner | release and major infra change |
| Manual E2E | complete user flow per phase | product + engineer | hard phase gate |

Use deterministic model stubs for normal CI. Provider-backed model evaluations run on an explicit, redacted, non-production fixture lane and record model/version/config. Do not accept a model-output snapshot as the only test oracle; assert diff constraints and verifier behavior.

## 9. Evaluation dataset and experiment protocol

1. Label each fixture finding with source locations, permitted recipe IDs, required official document passages, forbidden auto-fixes, expected diff assertions, and verifier status.
2. Split entire repositories into train/dev/holdout. A module or near-duplicate cannot appear in both dev and holdout.
3. Compare retrieval variants with fixed analyzer/query set: flat hybrid baseline → path-text hybrid → hierarchy expansion. Report Recall@1/3/5, MRR, citation version correctness, latency, and failure modes.
4. Promote a variant only if it meets safety requirements and offers a predeclared practical improvement; otherwise keep the simpler baseline. Avoid metric fishing after seeing test results.
5. Evaluate patching only after evidence is frozen. Report apply rate, test-pass rate, untouched-file rate, human acceptance, and abstention quality.

## 10. Backlog priority and sequencing

**P0 (must ship before pilot):** all Phase 0–5 safety gates, immutable source workflow, deterministic supported rules, pinned corpus, inspectable hybrid baseline, constrained diffs, isolated verifier, audit trail, fixtures/evals.

**P1 (after pilot):** richer `javax` dependency matrix, Gradle discovery (not migration), configurable organization policy packs, pull-request export integration, multiple verifier profiles, limited automatic remediation loop with a hard iteration cap.

**P2 / research track:** learned constrained hierarchy router inspired by STAIR, code-aware cross-encoder, broader Spring Boot 3.x targets, multi-language build graph, auto-triage from consented historical failures. This track may not replace the baseline until it passes the same retrieval, patch, and manual-flow gates.

## 11. Definition of done for any milestone

- Code, configuration, and schema migration reviewed; ADR updated for an architectural/safety decision.
- Unit/contract/integration/fixture tests added and passing.
- Telemetry, error state, documentation, and feature flag supplied.
- No mutation of original source in automated test; sandbox/retention behavior tested when applicable.
- The phase’s mandatory manual E2E checkpoint has a recorded run ID, fixture revision, tester, evidence, result, and any remediation.
- Failure means: do not waive the gate. Reduce scope, fix the failure, add a regression, and rerun the checkpoint from a clean snapshot.

