# Design_BrainModel (DBM)

> **AIを活用したソフトウェア開発のための、安全指向の推論システム。**
> Claude Code や Codex CLI のようなコーディングエージェントを「補完する」制御レイヤー。

| | |
|---|---|
| **リポジトリ** | https://github.com/chigenori053/Design_BrainModel |
| **開始** | 2026-01 |
| **主要言語** | Rust（約49,900行 / 539ファイル）· Python（約7,200行） |
| **規模** | **60を超える crate** によるワークスペース · ドキュメント55本 |
| **対象環境** | macOS（Apple Silicon 最適化）· Cargo · zsh |
| **状態** | **v1 実装完了 / v2 再設計予定** |
| **ライセンス** | 全権利留保（再設計中のため） |

---

## 本書の位置づけ — v1 と v2

**本書が説明しているのは Design_BrainModel v1（現在の Rust 実装）です。**

DBM は今後、**ReasonScript + MRA Base による v2 として再設計される予定**です。
再設計後は、[MRA（Molecular Reasoning Architecture）](../../README.md#系譜--project-lineage) の
**ソフトウェア設計ドメインモデル**として、[VisionWorldModel](visionworldmodel.md)（視覚）·
[LanguageModel](languagemodel.md)（言語）と並ぶ位置に入ります。

v1 は破棄されるものではありません。**MRA と ReasonScript が満たすべき要件を洗い出した先行実装**です。
特に後述の決定論ゲート設計（FNV-1a、`{:.6}` の固定精度、`1e-6` の閾値）は、
ReasonScript の決定論的コンパイルパイプラインに直接つながっています。

---

## 解こうとしている問題

フロンティアのコーディングエージェントはコード生成には強い。しかし——

| 課題 | DBM のアプローチ |
|---|---|
| エージェントは設計意図を保持しない | `design.md` → `DesignUnit` による意図の永続化 |
| LLM出力は非決定的 | 構造化された実行・検証フローによる制御 |
| AI生成の変更がアーキテクチャを壊しうる | 設計・コード・実行を横断した推論 |
| 修正ループがアドホックになりやすい | Repair を推論プロセスの正式な一部として扱う |
| 安全でない実行がプロジェクトを破壊しうる | コマンド分類による実行安全制御 |

> *"DBM doesn't compete with Claude Code — it provides the structure and safety that Claude Code tends to lack."*

競合ではなく補完である、という立ち位置が明確に打ち出されています。

---

## アーキテクチャ

```
┌─────────────────────────────────────┐
│          Design Intent              │
│      design.md / DesignUnit         │   ← 設計意図を構造化して永続化
└──────────────────┬──────────────────┘
                   │ anchored to
┌──────────────────▼──────────────────┐
│        DesignBrainModel (DBM)       │
│                                     │
│  Planner → Executor → Validation    │
│  Repair Loop → Convergence Control  │
│  Execution Safety                   │
└──────────────────┬──────────────────┘
                   │ operates on
┌──────────────────▼──────────────────┐
│       Development Environment       │
│    Source Code / Cargo / Git / REPL │
└─────────────────────────────────────┘
```

設計意図が最上位にあり、そこに**アンカーされた**推論ループが開発環境を操作する、という三層構造です。
コードが設計から乖離することを、構造的に防ぐことを狙っています。

---

## 決定論ゲート — 仕様で凍結された再現性

DBM の際立った点は、**再現性の実現手段を仕様レベルで凍結している**ことです。
`DESIGN.md` から抜粋します。

### スナップショット形式

`MeaningLayerSnapshotV2` を正規形式とし、比較キーを3つだけに限定：

```
l1_hash · l2_hash · version
```

`timestamp_ms` は**ログ専用**で、差分判定では明示的に無視されます。

### ハッシュポリシー（固定）

| 項目 | 値 |
|---|---|
| アルゴリズム | FNV-1a 64bit |
| シード | `0xcbf29ce484222325` |
| 素数 | `0x100000001b3` |
| バイト順 | 決定論的な UTF-8 バイト列 |

正規化規則も明文化されています：

- ベクトルと浮動小数点値は**固定精度の文字列**に整形（`{:.6}`）
- リスト型フィールドはハッシュ前に**ソート**
- `Option` は `null` または `some:<value>` として符号化
- **空文字列と `None` は区別する**

### 決定論ゲート

同一入力に対して、以下が完全に一致することを要求します。

- `snapshot_v2` のハッシュ
- テンプレート選択
- 説明テキスト

テンプレート選択の曖昧性閾値も固定：`TEMPLATE_SELECTION_EPSILON = 1e-6`

**浮動小数点の丸め方や空文字列の扱いまで仕様に書く**というのは、
「決定論を努力目標にしない」という姿勢の具体的な現れです。

### API凍結と非推奨化

PhaseA-Final で V2 API を既定に昇格。非推奨APIには
`since = "PhaseA-Final"` と `note = "Will be removed in PhaseC"` を付与し、移行先を明示します。

```
HybridVM::snapshot            → HybridVM::snapshot_v2
HybridVM::compare_snapshots   → HybridVM::compare_snapshots_v2
HybridVM::explain_design      → HybridVM::explain_design_v2
HybridVM::rebuild_l2_from_l1  → HybridVM::rebuild_l2_from_l1_v2
```

---

## Agent Operational Charter — AI協働の統制設計

`AGENT.md` は、**人間とAIエージェントの協働ルールそのものを仕様化**した文書です。

### 権限階層

```
1. specs/
2. Rule.md
3. TASK_STATE.yaml
4. 実装コード
```

> **実装が仕様を上書きしてはならない。**

### 役割定義

| 役割 | 担当 | できること | できないこと |
|---|---|---|---|
| **Architect** | 人間 | 仕様の承認、アーキテクチャ上の対立の解決、最終決定 | — |
| **ResearchAgent** | Gemini CLI | 仕様案の生成、代替設計の提示 | 本番コードを書かない、実装ファイルを変更しない |
| **CodingAgent** | Codex CLI | 承認済み仕様の実装、構造整合性のためのリファクタ、テスト生成 | アーキテクチャを再定義しない |
| **ValidationAgent** | — | 仕様と実装の整合検証、性能・回帰チェック | — |

複数のAIエージェントを**役割で分離し、権限境界を引く**という発想は、
単なる「AIに手伝ってもらう」を超えて、AIを含むチーム設計になっています。

---

## crate 構成

60を超える crate が、責務ごとに切られています。主要な系統：

| 系統 | crate |
|---|---|
| **アーキテクチャ推論** | `architecture_reasoner` `architecture_evaluator` `architecture_knowledge` `architecture_memory` `architecture_metrics` `architecture_rules` `architecture_state` `architecture_behavior` |
| **記憶空間** | `memory_space` `memory_space_api` `memory_space_complex` `memory_space_eval` `memory_space_index` `memory_space_recall` `memory_graph` `memory_store` |
| **意味層** | `chm` `dhm` `shm` `semantic_dhm` `language_dhm` `concept_engine` `concept_field` `meaning_extractor` |
| **実行** | `hybrid_vm` `execution_graph` `execution_simulator` `runtime` `engine` |
| **設計推論** | `design_reasoning` `code_ir` `code_generation_core` `recomposer` |
| **制御** | `policy_engine` `search_controller` `search_verification` `evaluation_engine` |
| **モデル** | `world_model` `system_model` `performance_model` `workload_model` `brain_core` |

アプリケーション層は `apps/` に：**cli · desktop · gui · lsp · server**。

---

## 使ってみる

```bash
cargo build -p design_cli --bin dbm
```

```bash
cargo run -p design_cli -- --help
cargo test -p design_cli
```

REPL 起動：

```bash
cargo run -p design_cli -- --repl
```

REPL での基本操作：

```
> analyze current project structure
> inspect target module
> propose repair plan
> apply safe modification
> validate result
```

---

## 実装状況

| 機能 | 状態 |
|---|---|
| 自律実行ループ | ✅ 実装済み |
| REPL ベースの対話 | ✅ 実装済み |
| Planner → Executor フロー | ✅ 実装済み |
| 収束制御（Convergence control） | ✅ 実装済み |
| 実行安全性（コマンド分類） | ✅ 実装済み |
| デバッグフロー | ✅ 実装済み |
| Git 読み取り専用統合 | 🔧 進行中 |
| 制限付き git add / commit | 🔧 進行中 |
| 設計意図統合（DesignUnit） | 🔧 進行中 |
| アーキテクチャ認識リファクタ | 📋 計画中 |
| GitHub / PR 統合 | 📋 計画中 |
| UI / ダッシュボード | 📋 計画中 |

---

## 評価レポート

リポジトリルートには、フェーズごとの機械可読な評価レポートが並んでいます。

`architecture_benchmark_report.json` · `critical_risk_report.json` ·
`designgraph_explosion_report.json` · `memoryspace_verification_report.json` ·
`worldmodel_verification_report.json` · `phase6_large_scale_report.json` ·
`phase7_real_repository_report.json` · `phase8_human_evaluation_report.json` ·
`trace_depth100_phase6.csv`

Phase 7 で**実リポジトリ**に対する検証、Phase 8 で**人間による評価**まで進んでいます。

---

## この設計の背景にある考え方

DBM は「AIにコードを書かせる」ツールではなく、**AIが書いたコードが設計から逸脱していないかを推論で監視する**
レイヤーです。そのために設計意図を構造化データとして永続化し、実行を安全性で分類し、
修正ループを推論プロセスに組み込んでいます。

Rust を選んでいるのは、この制御レイヤー自体が壊れないこと——つまり**監視する側の信頼性**が
前提条件になるためだと読めます。

---

## v2 への再設計

v1 は独立した Rust プロジェクトとして、推論・記憶・アーキテクチャ評価の機構を
すべて自前で構築しました。60を超える crate という規模は、その結果です。

v2 では、これらを共通基盤に委ねます。

| 関心事 | v1 | v2 |
|---|---|---|
| 決定論的実行 | `hybrid_vm` 等で自前実装 | **ReasonScript** の ExecutionPlan |
| 知識・記憶表現 | `memory_space_*` · `chm` / `dhm` / `shm` を自前実装 | **MRA Base** の Molecule / Evidence / Provenance |
| 設計意図の永続化 | `DesignUnit`（独自形式） | MRA の型付き Molecule として表現 |
| 検証 | 独自の評価エンジン | ReasonScript の検証契約 + MRA の Evidence 検証 |

これにより DBM は、**ソフトウェア設計というドメインに固有の推論**に集中できます。
視覚（VisionWorldModel）· 言語（LanguageModel）と同じ基盤の上で、
同じ Molecule 表現を使って設計知識を扱う——という構図です。

v1 で確立した以下の設計は、v2 にも引き継がれる想定です。

- Planner → Executor → Validation → Repair Loop → Convergence Control という推論ループ
- コマンド分類による実行安全制御
- Agent Operational Charter による権限階層と役割分担

→ [設計思想の詳細](../design-philosophy.md) · [開発年表](../timeline.md)
