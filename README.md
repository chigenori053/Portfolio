# Portfolio — Software / Backend Engineer

*[English version →](README.en.md)*

**Python / Rust を中心に、Backend、Runtime、Persistence、Developer Toolingの設計・実装を行っています。**
個人開発では ReasonScript という DSL / Runtime を設計・実装し、言語処理・永続化・Transaction・Cluster実行・CLI・Testingを構築しています。実務ドメインに接続した小規模なフルスタック開発（後述の KuKKA）もあります。現在は Software Engineer / Backend Engineer としてのキャリア形成を目指しています。

- **GitHub** — [@chigenori053](https://github.com/chigenori053)
- **主言語** — Python · Rust · TypeScript
- **対象職種** — 下記「[Target Roles](#target-roles)」を参照

<sub>本ポートフォリオでは、**主張ごとに証拠の強さ（VALIDATED / TESTED / IMPLEMENTED / PROTOTYPE / EXPERIMENTAL / DESIGN / RESEARCH / CONCEPT）を明示**しています。裏付けの弱い主張を強い成果として提示しないことを方針としています。全体は [Evidence Index](evidence/evidence-index.md) にまとめています。</sub>

---

## Target Roles

| | |
|---|---|
| **Primary** | Software Engineer · Backend Engineer |
| **Secondary** | Platform Engineer · Developer Tools Engineer · AI Backend Engineer |
| **Long-term** | R&D Engineer / Research Software Engineer |

---

## Core Engineering Skills

| Area | Skills | Evidence |
|---|---|---|
| **Language** | Python, Rust, TypeScript | [ReasonScript](case-studies/reasonscript.md) |
| **Compiler / Runtime** | 字句・構文解析、AST設計、中間表現(IR)設計、決定論的実行計画生成 | [ReasonScript Case Study](case-studies/reasonscript.md) |
| **Persistence / Data Modeling** | 構造レベルのトランザクション・世代管理、リレーショナルDB設計（Prisma/PostgreSQL） | [Persistent Graph Runtime](case-studies/persistent-graph-runtime.md) · [Backend Project: KuKKA](projects/backend-service.md) |
| **API Design** | REST API、クロス言語DTO契約（5言語） | [Backend Engineering Evidence](backend-engineering/overview.md#1-api--interface-design) |
| **Distributed Execution** | worker調整・retry・timeout付きクラスタ実行（Rustクレート、単体テストで確認済み） | [Cluster Runtime Case Study](case-studies/cluster-runtime.md)（TESTED） |
| **Testing / CI** | Unit/Integration/Golden/差分テスト、9ステージCIパイプライン | [ReasonScript Case Study](case-studies/reasonscript.md) — 1,240テストPASS実行確認済み |
| **Debugging / Failure Analysis** | 根本原因分析、構造的診断、限界の正直な開示 | [Debugging & Failure Analysis](case-studies/debugging-and-failure-analysis.md) |
| **Tooling** | CLI、LSP、VS Code拡張、ブラウザPlayground | [ReasonScript Case Study](case-studies/reasonscript.md) |

---

## Featured Project — ReasonScript

**推論を状態遷移として記述する言語**を、Python製コンパイラ/ツールチェーンと単一のRustランタイムホストによる Hybrid DSL として設計・実装しています。

- 実装約154,300行（Python 98,518行 / Rust 55,826行）、仕様書102本
- `./reason ci` を実行し、**全9ステージ PASS・1,240件のテスト通過を確認済み**（2026-08-31, commit `edfd477`）
- 5言語（Rust/Python/TypeScript/Go/Java）が単一のDTO契約を共有
- 状態を書き込む操作を `apply`/`rollback` の2つに限定し、証明失敗時の自動ロールバックを言語意味論に組み込み
- Apache-2.0

→ [Case Study（SE視点の要約）](case-studies/reasonscript.md) · [詳細（研究的背景を含む全体像）](docs/projects/reasonscript.md)

---

## Backend Engineering Evidence

「バックエンドエンジニアとして採用できる根拠」を12項目で整理しています。証拠が薄い項目（Security・Observability）は誇張せずGapとして明記しています。

| 領域 | 状態 |
|---|---|
| API/Interface Design, Data Modeling | IMPLEMENTED |
| Persistence, Transaction Management | VALIDATED（構造レベル）/ PROTOTYPE（RDB） |
| Concurrency | TESTED（部分的） |
| Distributed Execution | TESTED（単一マシン上の複数プロセス協調。ネットワーク分散は未確認） |
| Fault Tolerance, Error Handling | IMPLEMENTED〜VALIDATED |
| Security, Observability | **Gapとして明記**（本番運用レベルの実装例なし） |
| Testing, CI/CD | VALIDATED |

→ [Backend Engineering Evidence 全項目](backend-engineering/overview.md)

---

## Selected Case Studies

すべて Problem → Requirements → Constraints → Architecture → Design Decisions → Implementation → Testing → Problems Found → Root Cause → Fix → Verification → Result → Known Limitations という統一フォーマットで記述しています。

| Case Study | 見せる能力 | Status |
|---|---|---|
| **[ReasonScript](case-studies/reasonscript.md)** | 決定論的コンパイラ・ランタイム設計、CI/テスト基盤 | VALIDATED |
| **[Persistent Graph Runtime](case-studies/persistent-graph-runtime.md)** | 構造レベルの永続化・トランザクションモデル（VisionWorldModel） | VALIDATED |
| **[Cluster Runtime](case-studies/cluster-runtime.md)** | Worker調整・Retry・Timeoutを備えた分散実行（ソース調査・テストで確認済み） | TESTED |
| **[Debugging & Failure Analysis](case-studies/debugging-and-failure-analysis.md)** | 障害の根本原因分析と、限界を正直に開示する判断 | 3件の実例 |

## Backend Project

| プロジェクト | 概要 | Status |
|---|---|---|
| **[KuKKA — プログラミング教室 予約・運営システム](projects/backend-service.md)** | 実在する教室のNext.js+Prisma+PostgreSQLによる予約・管理システム。認証・テスト未実装のまま開発停止中 | **PROTOTYPE**（一時停止中） |

---

## Software Engineering Process

```
Requirement → Specification → Architecture → Implementation
   → Automated Test → Failure Analysis → Specification Revision → Regression Test
```

仕様書を実装より先に書き、Phase単位で刻んで検証する進め方を採用しています（詳細: [設計思想](docs/design-philosophy.md)）。

### AIエージェントとの協働

Coding Agentを活用した開発ですが、役割は明確に分離しています。

| 担当 | 役割 |
|---|---|
| **Human** | Requirements、Architecture、Specification、Review、Failure classification、Acceptance decision |
| **Coding Agents** | Implementation support、Refactoring、Test implementation、Static analysis support |

---

## Advanced R&D

Software/Backend Engineeringの基盤の上に、MRA（Molecular Reasoning Architecture）という推論アーキテクチャの研究開発があります。COHERENT・VisionWorldModel・LanguageModel・Design_BrainModelがその構成要素です。

→ [Advanced R&D 全体像](advanced-rd/overview.md)

---

## Tech Stack

| 領域 | 技術 |
|---|---|
| **言語処理系** | 字句・構文解析、AST設計、中間表現(IR)、実行計画生成、型仕様、名前空間解決 |
| **Rust** | ランタイム実装、Cargo によるマルチクレート管理、Safe-Rust、LSPサーバ |
| **Python** | 処理系・ツールチェーン実装、pytest、uv |
| **Web/Backend** | Next.js（App Router）、Prisma、PostgreSQL、REST API設計 |
| **クロス言語** | Rust / Python / TypeScript / Go / Java の共通DTO契約 |
| **品質保証** | CI パイプライン、Conformance framework、Golden コーパス、決定論ゲート |

---

## Status

| プロジェクト | 状態 |
|---|---|
| **ReasonScript** | v0.5.5.8リリース済み（Apache-2.0）。`./reason ci` 全ステージPASS・1,240テスト確認済み（2026-08-31） |
| **KuKKA (backend-service)** | Next.js+Prisma+PostgreSQLによる予約・管理システム。2026-03-14を最後に開発停止中。認証・テスト未実装 |
| **VisionWorldModel** | Phase 3C-1まで検証完了 |
| **MRA / LanguageModel** | Phase 0完了（基盤固定・仕様策定）。実装はこれから |
| **Design_BrainModel** | v1は未完成プロダクト（推論爆発の抑制に安定の根拠なし）。ReasonScript + MRA Baseによるv2再設計を予定 |
| **COHERENT** | 検証継続中。単語・多言語の想起100%（実測）が最も確度の高い成果。記憶再利用による計算削減は未測定 |
| **mathlang** | 2025-11で更新停止（Apache-2.0） |

---

## Licensing

**基盤ツールは開き、研究アーキテクチャ本体・実務プロジェクトは留保する**方針です。

| 対象 | 方針 |
|---|---|
| **ReasonScript / mathlang** | Apache-2.0 |
| **MRAドメインモデル**（VisionWorldModel / LanguageModel / Design_BrainModel）· **COHERENT** | 全権利留保（開発中/研究プロジェクトのため） |
| **KuKKA (backend-service)** | 実務プロジェクトのため非公開ライセンス方針。コードは閲覧・評価目的で公開 |

留保しているリポジトリのコードも、閲覧と評価のために公開しています。利用をご希望の場合は、各リポジトリのIssueでご相談ください。

---

## Documentation

- **[Backend Engineering Evidence](backend-engineering/overview.md)** — バックエンドエンジニアとしての根拠12項目
- **[Case Studies](case-studies/)** — ReasonScript・Persistent Graph Runtime・Cluster Runtime・Debugging & Failure Analysis
- **[Backend Project](projects/backend-service.md)** — KuKKA（実務プロジェクト）
- **[Evidence Index](evidence/evidence-index.md)** — 全主張のClaim/Evidence/Statusテーブル
- **[Advanced R&D](advanced-rd/overview.md)** — MRA系譜の全体像
- **[設計思想](docs/design-philosophy.md)** · **[開発年表](docs/timeline.md)** · **[技術経歴書](docs/technical-profile.md)**
- **プロジェクト詳細（研究的背景を含む全体像）** — [ReasonScript](docs/projects/reasonscript.md) · [VisionWorldModel](docs/projects/visionworldmodel.md) · [COHERENT](docs/projects/coherent.md) · [LanguageModel](docs/projects/languagemodel.md) · [Design_BrainModel](docs/projects/design-brainmodel.md) · [mathlang](docs/projects/mathlang.md)
