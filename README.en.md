# Portfolio — Research & Engineering on Verifiable Reasoning Systems

*[日本語版 →](README.md)*

**From "trusting what AI outputs" to "verifying how AI reasons."**

This portfolio collects a connected body of work addressing three problems with large language models:
**non-determinism, unverifiability, and the loss of design intent.** The answer proposed here is a
**deterministic language runtime paired with a reasoning architecture that separates memory from truth.**

At the center is **ReasonScript**, a programming language built from scratch, with applied research projects layered on top.

---

## Project Lineage

```
                        ┌──────────────────────────────┐
   2025-11              │  mathlang                    │  A DSL for reasoning processes
   ────────────         │  MathLang                    │  Education → research
                        └──────────────┬───────────────┘
                                       │
                        ┌──────────────▼───────────────┐
   2025-11              │  COHERENT                    │  Recall (System 1) × Reasoning (System 2)
   ────────────         │  Optical Holographic Memory  │
                        └──────────────┬───────────────┘
                                       │
                        ┌──────────────▼───────────────┐
   2026-01              │  Design_BrainModel           │  Safety & integrity layer for AI agents
   ────────────         │  Rust / 60+ crates           │  Design-intent persistence
                        └──────────────┬───────────────┘
                                       │
                        ┌──────────────▼───────────────┐
   2026-04              │  ★ ReasonScript              │  Foundation language & runtime
   ────────────         │  v0.5.4.5 / 6 lang bindings  │  Surface AST → … → ExecutionPlan
                        └──────────────┬───────────────┘
                                       │
                    ┌──────────────────┴──────────────────┐
      ┌─────────────▼─────────────┐       ┌───────────────▼──────────────┐
      │  VisionWorldModel         │       │  LanguageModel               │
      │  2026-07                  │       │  2026-08                     │
      │  Observation vs inference │       │  MRA Holographic Semantic    │
      │  ACCEPT/REVISE/DEFER/     │       │  Memory / Truth Boundary     │
      │  ABSTAIN                  │       │                              │
      └───────────────────────────┘       └──────────────────────────────┘
```

The two downstream projects never modify the foundation — they consume only ReasonScript's public CLI
and its deterministic reasoning contract. `LanguageModel` pins the foundation to an exact commit (`7f29c1c`).

---

## Projects

| Project | Summary | Stack | Scale |
|---|---|---|---|
| **[ReasonScript](https://github.com/chigenori053/ReasonScript)** | A reasoning-first language guaranteeing deterministic execution and rollback safety at the specification level | Python / Rust / TS / Go / Java | ~135k LOC · 1,085 tests |
| **[Design_BrainModel](https://github.com/chigenori053/Design_BrainModel)** | A reasoning-and-control layer giving AI coding agents design intent and execution safety | Rust | ~57k LOC · 60+ crates |
| **[COHERENT](https://github.com/chigenori053/COHERENT)** | A Reasoning LM fusing optical holographic memory with action-based reasoning | Python | ~43k LOC |
| **[mathlang](https://github.com/chigenori053/mathlang)** | A DSL that captures, replays, and verifies the *process* of mathematical reasoning | Python | ~9.2k LOC |
| **[VisionWorldModel](https://github.com/chigenori053/VisonWorldModel)** | A world-model validation system that separates observation from inference and can withhold judgment | ReasonScript / Python | ~7.7k LOC |
| **[LanguageModel](https://github.com/chigenori053/LanguageModel)** | A language-model foundation separating associative memory from canonical knowledge (Phase 0, spec stage) | Python / ReasonScript | Specification-led |

Detailed write-ups are in Japanese under [`docs/projects/`](docs/projects/).

---

## Design Principles

### 1. Determinism guaranteed by specification, not by effort

"The same input produces the same execution plan" is implemented as a **language guarantee and a validation gate**.
Design_BrainModel freezes the hash algorithm (FNV-1a 64-bit), the seed, the float formatting precision (`{:.6}`),
and even the template-selection ambiguity threshold (`1e-6`) in its specification.

### 2. Separating memory from truth — the Truth Boundary

Associative (holographic) memory is a **semantic activation field**, not a store of truth.

> A relation MUST NOT be asserted on the basis of vector similarity, decoding results,
> or neural model confidence alone.
> — *MRA Holographic Semantic Language Model Specification v0.1, §2.1*

"Semantically close" and "this relation holds" are strictly distinguished; any factual response must pass
through canonical data and Evidence validation. The same split appears in COHERENT (recall → verify)
and VisionWorldModel (observation → inference).

### 3. Withholding judgment is a first-class output

VisionWorldModel returns one of four decisions: `ACCEPT` / `REVISE` / `DEFER` / `ABSTAIN`.
When evidence is insufficient, **deferring or abstaining is the correct behavior**, not a failure —
a deliberate counterproposal to models that always produce an answer.

### 4. Specification first, progress in phases

Specifications using MUST / MUST NOT / SHOULD / MAY precede implementation, and work advances through
Phase 0 → 1 → 2 → 3A → 3B-1 → 3B-2 → 3B-3 → 3C-1. Every phase ships a validation command
and a machine-readable artifact.

### 5. Treating human–AI collaboration itself as a design problem

`AGENT.md` / `Rule.md` / `AGENTS.md` codify an authority hierarchy and role separation:

```
Authority:  specs/  >  Rule.md  >  TASK_STATE.yaml  >  implementation code
Roles:      Architect (human) / ResearchAgent / CodingAgent / ValidationAgent
```

> *"DBM doesn't compete with Claude Code — it provides the structure and safety that Claude Code tends to lack."*
> — Design_BrainModel README

---

## Tech Stack

| Area | Technologies |
|---|---|
| **Language implementation** | Lexing/parsing, AST design, intermediate representation, execution planning, type specification, namespace resolution |
| **Rust** | 60+ crate workspace, Safe-Rust runtime, Cargo, LSP server |
| **Python** | Compiler & runtime implementation, symbolic computation via SymPy, pytest, uv |
| **Cross-language** | A single normative DTO contract shared across Rust / Python / TypeScript / Go / Java |
| **Numerics** | Complex-valued tensors, Conv2d/MaxPool2d/AvgPool2d, reverse-mode autodiff |
| **AI & reasoning** | HRR / VSA distributed representations, symbolic reasoning, causal inference, fuzzy decision, multimodal integration |
| **Tooling** | CLI, REPL, IDE, VS Code extension, LSP, browser playground, CI pipeline |
| **Quality** | Conformance framework, golden corpus, determinism gates, schema validation |

---

## Status

| | State |
|---|---|
| **ReasonScript** | v0.5.4.5 released; 1,085 CI tests passing. ReasonGraph/World viewers, package registry, and the SDK public API manifest remain open |
| **Design_BrainModel** | Autonomous execution loop, REPL, and execution safety implemented; Git integration and DesignUnit integration in progress |
| **COHERENT** | In development toward Phase 2 (Coding Agent Integration) |
| **VisionWorldModel** | Validated through Phase 3C-1; adaptive structural reasoning underway |
| **LanguageModel** | Phase 0 (foundation pinning, specification) complete; Holographic Core implementation next |

---

<sub>Licenses are per-repository. ReasonScript has no finalized root LICENSE yet (only `vscode-extension/` is MIT).</sub>
