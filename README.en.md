# Portfolio — Research & Engineering on Verifiable Reasoning Systems

*[日本語版 →](README.md)*

**From "trusting what AI outputs" to "verifying how AI reasons."**

This portfolio collects a connected body of work addressing three problems with large language models:
**non-determinism, unverifiability, and the loss of design intent.** The answer proposed here is a
**deterministic language runtime paired with a reasoning architecture that separates memory from truth.**

At the center is **ReasonScript**, a programming language built from scratch, with applied research projects layered on top.

---

## Project Lineage

A concept formed during an early research phase became an exploratory implementation; findings
from that phase converged into a foundation (ReasonScript), on top of which
**MRA — Molecular Reasoning Architecture** is now being built.

```
  ■ Research ──────────────────────────────────────────────────────

   2025-01   Considered adding a math course at the programming school I run
                 │                 Idea: use an LLM as a learning coach
                 │                 → Consulted an education expert. Feedback: "too hard"
                 │                   (LLMs of the time were unreliable at math, unstable output)
                 │
                 │  Learned of SymbolicAI
                 │  ▼
                 │  Learned of Wolfram Alpha (an existing education-domain service), tested it
                 │  → Found it could not evaluate an input expression step by step,
                 │    including intermediate calculations
                 ▼
             Arrived at the MathLang concept:
             "a dedicated educational language that understands and evaluates
              expressions including their intermediate steps"

  ■ Exploration ───────────────────────────────────────────────────

   2025-11   mathlang              A math-learning support language (Python DSL)
                 │                 Expression correctness judgment
                 ▼
   2025-11   COHERENT              Theory-validation project (reasoning model: BrainModel)
                 │                 "Can non-Transformer reasoning match an LLM?"
                 │                 HolographicMemory + MemorySpace ── precursor to MRA memory
                 ▼
   2026-01   Design_BrainModel v1  A coding agent that recalls code
                 │                 Rust / 60+ crates ── incomplete product
                 │
                 │  hit reasoning explosion and memory-core limits;
                 │  decided to rebuild from the foundation
                 ▼
  ■ Foundation ────────────────────────────────────────────────────

   2026-04   ★ ReasonScript        A state-transition language for reasoning (Apache-2.0)
                 │                 Hybrid DSL: Python front end + Rust runtime
                 │                 Surface AST → … → ExecutionPlan
                 │
  ■ MRA ───────┴───────────────────────────────────────────────────

            ★ MRA — Molecular Reasoning Architecture   ← in development
              A reasoning model representing knowledge as Molecules
              built from typed Atoms and Bonds
                 │
     ┌───────────┼────────────────────────┐
     ▼           ▼                        ▼
 VisionWorldModel   LanguageModel      Design_BrainModel v2
  Vision domain     Language domain     Software-design domain
  2026-07           2026-08             (redesign planned)

  Observation vs    Truth Boundary       Design → system
  inference         Associative recall   structure → code
  ACCEPT/REVISE/    vs canonical         recall
  DEFER/ABSTAIN     knowledge
```

### Why the research phase isn't on GitHub

The months from January 2025 were spent mostly on defining requirements and consulting experts,
not writing code. While considering a new math course at the programming school I run, I got
the idea of using an LLM as a learning coach — but an education expert I consulted said it would
be "too hard," citing the unreliability of contemporaneous LLMs at math. I then learned of
SymbolicAI, and afterward of Wolfram Alpha as an existing education-domain service; testing it
showed it could not evaluate an input expression step by step, including intermediate
calculations. That gap led directly to the MathLang concept: a dedicated educational language
that understands and evaluates expressions including their intermediate steps. There was no
repository to keep from this period — it produced a concept, not code — so it leaves no trace
on GitHub. **mathlang, the starting point of the exploration phase, is that conclusion put
directly into implementation.**

**One foundation supporting three distinct application domains** — vision, language, and software
design. A foundation with a single application only ever demonstrates that application; three
independent domains are what actually test whether the design generalizes.

Domain models never modify the foundation — they consume only ReasonScript's public CLI and its
deterministic reasoning contract. `LanguageModel` pins the foundation to an exact commit (`7f29c1c`).

---

## Projects

### Foundation

| Project | Summary | Stack | Scale | License |
|---|---|---|---|---|
| **[ReasonScript](https://github.com/chigenori053/ReasonScript)** | A **state-transition language for describing reasoning**, guaranteeing deterministic execution and rollback safety at the specification level | **Hybrid DSL** — Python front end / Rust runtime | ~133.6k LOC · **1,116 CI tests, verified by running them** | Apache-2.0 |

#### External Verification

Beyond ReasonScript's own CI, a separate project tests whether its core guarantees — determinism,
non-interfering observation, and independent verifiability — hold up under a realistic workload.

| Project | What it tests | Result | Status |
|---|---|---|---|
| **[Transformer_Test](https://github.com/chigenori053/Transformer_Test)** (RS-DT-JP-GREET-001) | A Transformer implemented entirely in `.rsn`, trained on an 8-class Japanese greeting classification task. Tests determinism, observation non-interference, and independent (Rust) verification | **Determinism holds** (bit-identical checkpoint SHA-256 across runs); **observation does not affect computation**, both measured directly. Found and fixed 3 type-checker bugs in the foundation. The structural advantage of Models A–D themselves remains **unverified** | Phases 0–6 implemented and measured; Phases 7–9 (the full ablation experiment) are paused on a performance constraint |

### MRA Domain Models

Domain-specific models composing MRA, the reasoning architecture currently under development.

| Project | Domain | Summary | Scale | Status |
|---|---|---|---|---|
| **[VisionWorldModel](https://github.com/chigenori053/VisonWorldModel)** | Vision | A world model separating observation from inference, able to withhold judgment | ~7.7k LOC | Phase 3C-1 |
| **[LanguageModel](https://github.com/chigenori053/LanguageModel)** | Language | A foundation separating associative recall from canonical, evidence-backed knowledge | Specification-led | Phase 0 |
| **[Design_BrainModel](https://github.com/chigenori053/Design_BrainModel)** | Software design | A coding agent that generates system structure from an agreed design, then **recalls** code from that structure | ~57k LOC · 60+ crates | **v1 incomplete / v2 redesign planned** |

### Exploration

Predecessor projects that led to ReasonScript and MRA.

| Project | Summary | Stack | Scale | Status |
|---|---|---|---|---|
| **[COHERENT](https://github.com/chigenori053/COHERENT)** | A **theory-validation project** (reasoning model: **BrainModel**) testing whether reasoning can hold without Transformers, via simulated optical interference and memory reuse | Python | ~43k LOC | Validation ongoing |
| **[mathlang](https://github.com/chigenori053/mathlang)** | A **math-learning support language** (Python-based DSL). Normalizes human-written notation via its own parser and embeds SymPy in the runtime to **judge the correctness of expressions** | Python | ~9.2k LOC | Inactive since 2025-11 (Apache-2.0) |

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
through canonical data and Evidence validation. The same split appears in COHERENT / BrainModel
(recall → verify, implemented as `Accept` / `Review` / `Reject` with a mandatory decision log)
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
| **Language implementation** | Lexing/parsing, AST design, intermediate representation, execution planning, type specification, namespace resolution, **state-transition semantics** |
| **Rust** | Language runtime implementation, 60+ crate workspace, Safe-Rust, Cargo, LSP server |
| **Python** | Language front end & toolchain implementation, symbolic computation via SymPy, pytest, uv |
| **Cross-language** | A single normative DTO contract shared across Rust / Python / TypeScript / Go / Java |
| **Numerics** | Complex-valued tensors, Conv2d/MaxPool2d/AvgPool2d, reverse-mode autodiff |
| **AI & reasoning** | HRR / VSA distributed representations, symbolic reasoning, causal inference, fuzzy decision, multimodal integration |
| **Tooling** | CLI, REPL, IDE, VS Code extension, LSP, browser playground, CI pipeline |
| **Quality** | Conformance framework, golden corpus, determinism gates, schema validation |

---

## Status

| | State |
|---|---|
| **ReasonScript** | v0.5.4.5 released. **`./reason ci` was executed for this portfolio: all stages PASS, 1,116 tests** (2026-08-12, commit `0efb2ab`, Python 3.14.0). ReasonGraph/World viewers, package registry, and the SDK public API manifest remain open |
| **Transformer_Test (external verification)** | RS-DT-JP-GREET-001. Measured: determinism (bit-identical checkpoint SHA-256) and observation non-interference; found and fixed 3 type-checker bugs in the foundation. Verifying the structural advantage of Models A–D (Phases 7–9) is paused on a performance constraint |
| **MRA** | In development. The Molecule / Evidence / Provenance data model and the Truth Boundary are established as specification |
| **VisionWorldModel** | Validated through Phase 3C-1; adaptive structural reasoning underway |
| **LanguageModel** | Phase 0 (foundation pinning, specification) complete; Holographic Core implementation next |
| **Design_BrainModel** | **v1 is an incomplete product** — (1) reasoning explosion caused system freezes; it is currently only forcibly suppressed, with no basis to claim stable operation, (2) the memory core did not learn as expected, (3) HolographicMemory proved too large to be practical. These three limits directly motivated **ReasonScript**. **v2 redesign planned on ReasonScript + MRA Base** |
| **COHERENT** | The strongest result is **100% recall over 60 words across two languages, backed by measured data**. Equation correctness judgment also holds. Character generation reached 100% for katakana and 60–80% for kanji, but the generating scripts are not in the repository. The recall-instead-of-inference mechanism has only a **5-case design check** — its performance is unmeasured |
| **mathlang** | Inactive since 2025-11. The problems solved for learners — recording, replaying, and equivalence-checking a reasoning process — became the foundation for everything that followed |

---

## Licensing

The policy is to **open the foundational tooling and reserve the research architecture itself.**

| Scope | Policy |
|---|---|
| **ReasonScript** | **Apache-2.0.** It is the means of implementing MRA, not the research object itself |
| **mathlang** | **Apache-2.0.** An early experiment, independent of MRA |
| **MRA domain models** (VisionWorldModel / LanguageModel / Design_BrainModel) | **All rights reserved.** They compose an architecture still under development |
| **COHERENT** | **All rights reserved.** A research and validation project |

Reserved repositories are still published **for reading and evaluation.**
If you are interested in using any of them, please open an issue on that repository.
