# StairMigration

Evidence-backed software migration with code analysis, structure-aware retrieval, reviewable patches, and sandboxed verification.

**Status:** Planning / documentation only. Application code has not been implemented.

The first milestone targets **Spring Boot 2.7.x → 3.0.x**, with Maven and Java 17. This deliberately bounded migration is the project's learning and evaluation target; it is not a recommendation to deploy an historical framework release in production.

## Project documents

- [Product Requirements Document - designed PDF](docs/PRD.pdf)
- [Product Requirements Document - editable source](docs/PRD.md)
- [Phased development plan](docs/DEVELOPMENT_PLAN.md)

The PRD defines scope, architecture, data models, endpoints, retrieval and patch policies, evaluation, and release criteria. The development plan breaks implementation into six phases with mandatory manual end-to-end checkpoints, sample fixture repositories, expected outcomes, pass/fail gates, and rollback criteria.

## What the product will do

1. Snapshot a local Maven repository and detect migration prerequisites.
2. Parse modules, source files, symbols, dependencies, and configuration.
3. Attach version-pinned official documentation to migration findings through lexical, dense, and structure-aware retrieval.
4. Present small patches and their supporting evidence for human selection.
5. Apply the selected patchset in a disposable workspace and verify the build and tests in isolation.
6. Export the diff, verification results, and remaining manual tasks.

## MVP boundaries

Supported candidates include Boot/Java build properties and approved Jakarta import/dependency recipes. Spring Security changes, complex semantic rewrites, Gradle, other language stacks, database migrations, deployments, and broader version chains are deferred. Review the PRD for the exact supported-pattern contract.

The system must preserve the original repository, record evidence for every patch, and distinguish a proposed patch from a verified migration.

## Architecture direction

Start with a modular monolith and a separately isolated verifier worker. The main modules will cover ingestion, versioned documentation, analysis/rules, hybrid retrieval, patch planning, run/artifact storage, and verification. The final technology choices are recorded during Phase 0 rather than implied by this documentation scaffold.

## How STAIR informs the project

IBM Research's [STAIR paper](https://arxiv.org/abs/2609.03874) motivates preserving hierarchy during retrieval. The MVP uses a STAIR-inspired adaptation with explicit code/document trees and an inspectable hybrid baseline. It does not reproduce the paper's trained generative retriever or claim its benchmark results for software migrations. A trained router is a later experiment, conditional on measured benefits.

Official migration evidence starts with a pinned revision of [Spring's 3.0 migration guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-3.0-Migration-Guide/eeac9539123659067e2918b9c225fb7798a46857) and [Spring Boot 3.0 reference documentation](https://docs.spring.io/spring-boot/docs/3.0.0/reference/html/getting-started.html).

## Implementation roadmap

| Phase | Deliverable | Mandatory manual checkpoint |
|---|---|---|
| 0 | Threat model, reproducible fixtures, runner contract | Safety baseline |
| 1 | Read-only Maven/Java analysis and findings | Analysis and location accuracy |
| 2 | Versioned corpus and hybrid retrieval | Evidence and abstention flow |
| 3 | Reviewable plans, constrained generation, sandbox apply | Selected patches only |
| 4 | Isolated build/test verification and artifacts | Complete migration flow |
| 5 | Evaluation, failure recovery, pilot preparation | Release-candidate drill |

The plan's indicative 12-week schedule is an initial sequencing assumption, not a delivery commitment. Each phase must pass its manual gate before progressing.

## Getting started

Read the PRD and development plan, then begin Phase 0: finalize the technology decisions, define supported migration recipes, create the fixture suite, and prove source immutability and verifier isolation. There is no application install or run command yet.

## Current repository structure

```text
stairmigration/
  README.md
  .gitignore
  docs/
    PRD.pdf
    PRD.md
    DEVELOPMENT_PLAN.md
```

This is a local Git repository. No remote or hosted repository has been configured. No project license has been selected yet.
