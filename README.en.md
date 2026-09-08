# Portfolio — Software / Backend Engineer

*[日本語版 →](README.md)*

**I build backend systems, runtimes, persistence layers, and developer tooling, primarily in Python and Rust.**
My main personal project is **ReasonScript**, a DSL and runtime I designed and implemented from scratch, covering language processing, persistence, transactions, cluster execution, CLI tooling, and testing. I also have a small full-stack project connected to a real business (KuKKA, below). I'm currently pursuing a career as a Software Engineer / Backend Engineer.

- **GitHub** — [@chigenori053](https://github.com/chigenori053)
- **Primary languages** — Python · Rust · TypeScript

<sub>This portfolio grades every claim by strength of evidence (VALIDATED / TESTED / IMPLEMENTED / PROTOTYPE / EXPERIMENTAL / DESIGN / RESEARCH / CONCEPT), and avoids presenting weakly-supported numbers as strong results. See the [Evidence Index](evidence/evidence-index.md) (Japanese) for the full breakdown.</sub>

---

## Target Roles

| | |
|---|---|
| **Primary** | Software Engineer · Backend Engineer |
| **Secondary** | Platform Engineer · Developer Tools Engineer · AI Backend Engineer |
| **Long-term** | R&D Engineer / Research Software Engineer |

---

## Featured Project — ReasonScript

A **state-transition language for describing reasoning**, implemented as a Hybrid DSL: a Python compiler/toolchain paired with a single Rust runtime host.

- ~154.3k LOC (Python 98,518 / Rust 55,826), 102 specification documents
- `./reason ci` was run for this portfolio: **all 9 stages PASS, 1,240 tests** (2026-08-31, commit `edfd477`)
- A single DTO contract shared across 5 languages (Rust/Python/TypeScript/Go/Java)
- Only two operations mutate state (`apply`/`rollback`); an invalid proof triggers automatic rollback, built into the language semantics
- Apache-2.0

→ [Case study (SE-focused summary)](case-studies/reasonscript.md) · [Full write-up](docs/projects/reasonscript.md) (Japanese)

---

## Backend Engineering Evidence

Twelve areas of backend engineering evidence, each mapped to something that actually exists. Thin areas (Security, Observability) are marked as explicit gaps rather than overstated.

| Area | Status |
|---|---|
| API/Interface Design, Data Modeling | IMPLEMENTED |
| Persistence, Transaction Management | VALIDATED (structural) / PROTOTYPE (RDB) |
| Concurrency | TESTED (partial) |
| Distributed Execution | TESTED (multi-process coordination on a single machine; network-distributed execution unconfirmed) |
| Fault Tolerance, Error Handling | IMPLEMENTED–VALIDATED |
| Security, Observability | **Explicit gap** — no production-grade implementation yet |
| Testing, CI/CD | VALIDATED |

→ [Full breakdown](backend-engineering/overview.md) (Japanese)

---

## Selected Case Studies

Each case study follows the same structure: Problem → Requirements → Constraints → Architecture → Design Decisions → Implementation → Testing → Problems Found → Root Cause → Fix → Verification → Result → Known Limitations.

| Case Study | Demonstrates | Status |
|---|---|---|
| **[ReasonScript](case-studies/reasonscript.md)** | Deterministic compiler/runtime design, CI/test infrastructure | VALIDATED |
| **[Persistent Graph Runtime](case-studies/persistent-graph-runtime.md)** | Structural persistence & transaction model (VisionWorldModel) | VALIDATED |
| **[Cluster Runtime](case-studies/cluster-runtime.md)** | Worker coordination, retry, and timeout in a distributed task runtime — confirmed via source inspection and unit tests | TESTED |
| **[Debugging & Failure Analysis](case-studies/debugging-and-failure-analysis.md)** | Root-cause analysis and honest disclosure of limitations | 3 real incidents |

## Backend Project

| Project | Summary | Status |
|---|---|---|
| **[KuKKA — Programming School Booking System](projects/backend-service.md)** | A real programming school's Next.js + Prisma + PostgreSQL booking/admin system. Development paused, no auth or tests yet | **PROTOTYPE** (paused) |

(Case studies and project write-ups are in Japanese; the structure above should make them navigable regardless.)

---

## Software Engineering Process

```
Requirement → Specification → Architecture → Implementation
   → Automated Test → Failure Analysis → Specification Revision → Regression Test
```

Specifications are written before implementation, and progress is cut into verifiable phases (see [Design Principles](docs/design-philosophy.md), Japanese).

### Working with AI coding agents

Coding agents are used extensively, but roles are kept explicit:

| Owner | Responsibilities |
|---|---|
| **Human** | Requirements, architecture, specification, review, failure classification, acceptance decisions |
| **Coding agents** | Implementation support, refactoring, test implementation, static analysis support |

---

## Advanced R&D

On top of the software/backend engineering foundation sits **MRA (Molecular Reasoning Architecture)**, a reasoning architecture built across three domain models: COHERENT, VisionWorldModel, LanguageModel, and Design_BrainModel.

→ [Advanced R&D overview](advanced-rd/overview.md) (Japanese)

---

## Tech Stack

| Area | Technologies |
|---|---|
| **Language implementation** | Lexing/parsing, AST design, IR design, execution planning, type specification, namespace resolution |
| **Rust** | Runtime implementation, multi-crate workspace via Cargo, Safe-Rust, LSP server |
| **Python** | Toolchain implementation, pytest, uv |
| **Web/Backend** | Next.js (App Router), Prisma, PostgreSQL, REST API design |
| **Cross-language** | A shared DTO contract across Rust / Python / TypeScript / Go / Java |
| **Quality** | CI pipeline, conformance framework, golden corpus, determinism gates |

---

## Status

| | State |
|---|---|
| **ReasonScript** | v0.5.5.8 released (Apache-2.0). `./reason ci`: all stages PASS, 1,240 tests (2026-08-31) |
| **KuKKA (backend-service)** | Booking/admin system on Next.js + Prisma + PostgreSQL. Paused since 2026-03-14; no auth or tests |
| **VisionWorldModel** | Validated through Phase 3C-1 |
| **MRA / LanguageModel** | Phase 0 complete (foundation pinning, specification); implementation next |
| **Design_BrainModel** | v1 is an incomplete product (reasoning-explosion suppression has no basis to claim stability); v2 redesign planned on ReasonScript + MRA Base |
| **COHERENT** | Validation ongoing. Strongest result: 100% word/multilingual recall (measured). Compute-reuse efficiency is unmeasured |
| **mathlang** | Inactive since 2025-11 (Apache-2.0) |

---

## Licensing

The policy is to **open the foundational tooling and reserve the research architecture / real-world project.**

| Scope | Policy |
|---|---|
| **ReasonScript / mathlang** | Apache-2.0 |
| **MRA domain models** (VisionWorldModel / LanguageModel / Design_BrainModel) · **COHERENT** | All rights reserved (under development / research) |
| **KuKKA (backend-service)** | A real-world project; reserved by policy, published for reading and evaluation |

Reserved repositories are still published for reading and evaluation. If you're interested in using any of them, please open an issue on that repository.
