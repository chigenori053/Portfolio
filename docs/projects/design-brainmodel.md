# Design_BrainModel (DBM)

> **設計意図からコードを「想起」するコーディングエージェント。**
> 人間と合意した設計案からシステム構造を生成し、その構造からコードを想起する。

| | |
|---|---|
| **リポジトリ** | https://github.com/chigenori053/Design_BrainModel |
| **開始** | 2026-01 |
| **主要言語** | Rust（約49,900行 / 539ファイル）· Python（約7,200行） |
| **規模** | **60を超える crate** によるワークスペース · ドキュメント55本 |
| **対象環境** | macOS（Apple Silicon 最適化）· Cargo · zsh |
| **状態** | **v1 は未完成プロダクト / v2 再設計予定** |
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

そして——**v1 が直面した3つの限界こそが、ReasonScript を作る直接の動機になりました。**
詳細は後述の「[v1 が直面した限界](#v1-が直面した限界)」を参照してください。

---

## 開発コンセプト — コードを「想起」する

DBM が目指しているのは、**設計意図からコードに至る三段階のパイプライン**です。

```
  ┌─────────────────────────────────────────────┐
  │  1. 開発コンセプト / 設計案                  │
  │     人間との協働で固める（合意形成）          │  ← 人間が起点
  └────────────────────┬────────────────────────┘
                       │  から生成
  ┌────────────────────▼────────────────────────┐
  │  2. システム構造                             │
  │     構築すべき構造の表現                     │  ← 検証可能な中間層
  └────────────────────┬────────────────────────┘
                       │  から想起
  ┌────────────────────▼────────────────────────┐
  │  3. コード                                   │
  └─────────────────────────────────────────────┘
```

### 「生成」ではなく「想起」

三段目の動詞が **想起（recall）** であることが、このプロジェクトの中心的な主張です。

一般的なコーディングエージェントは、プロンプトからコードを**生成**します。
DBM は、システム構造に対応するコードを**記憶から想起**することを目指しています。

これは [COHERENT](coherent.md) の **Resonance Recall**、および
[MRA](../timeline.md#mra-という到達点) の連想活性化と同じ機構です。
系譜全体で一貫している「想起は候補を出し、検証が確定する」という構図が、
コード生成というドメインに適用された形になります。

MRA の [Truth Boundary](../design-philosophy.md#原則2--記憶と真実を分離するtruth-boundary) を
コードに当てはめると、こうなります。

> 想起されたコードは**候補**でしかない。
> それが正しいかを決めるのは、上位にある設計意図とその検証である。

### ReasonScript と同じ構造をしている

このパイプラインは、ReasonScript のコンパイルパイプラインと同型です。

| | ReasonScript | Design_BrainModel |
|---|---|---|
| **入力** | `.rsn` ソース | 開発コンセプト / 設計案 |
| **中間表現** | Surface AST → Semantic AST | システム構造 |
| **出力** | Reason IR → ExecutionPlan | コード |
| **共通点** | 入力から出力へ一足飛びに変換せず、検証可能な中間表現を必ず経由する | |

**入力から出力へ直接飛ばさない。** これが両者に共通する設計判断です。
LLM がプロンプトからコードへ一足飛びに到達してしまうことこそが、
設計意図が失われる原因である——という診断が背景にあります。

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
> — v1 README

**この一文は v1 時点の立ち位置**です。v1 は既存エージェントを補完する制御レイヤーとして設計されました。

ただし目指しているのは、その先です。
既存エージェントの出力を後から検証するのではなく、
**設計意図を起点として、検証可能な構造を経由してコードに到達する経路そのものを作る。**
上表の課題は、監視によってではなく、**生成の経路を変えることによって**解かれるべきだ——
というのが現在の設計方針です。

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

## v1 が直面した限界

**v1 は未完成プロダクトです。** 動作する部分は多くありますが、
中核の推論機構が実用水準に届かず、根本的な作り直しが必要だと判断しました。

具体的には、3つの問題に直面しました。

### 1. 推論爆発（Reasoning Explosion）

構造化推論を実行する際、**Unit 候補の生成が非線形に増加**します。
探索空間が制御を超えて膨張し、**システムフリーズを起こす**に至りました。

**現状は強引に抑え込んでいる状態です。**
フリーズ自体は回避できていますが、その抑制は原理に基づいたものではありません。
したがって、**システムが安定稼働に至ったと判断する材料がない**、というのが現在の評価です。

> **「落ちなくなったこと」と「安定していること」は別である。**

この線引きは、本プロジェクト群の
[原則3「判断しないことを、正当な出力にする」](../design-philosophy.md#原則3--判断しないことを正当な出力にする)を、
自分自身のプロダクト評価に適用した結果です。
根拠が不十分なまま「安定した」と宣言することはできない——だから v1 は未完成プロダクトのままです。

### 2. 記憶機構が学習能力を発揮しなかった

中核の推論 Core であった **MemorySpace** と **HolographicMemory** が、
**期待していた学習能力を発揮できませんでした。**

想起した経験から設計知識が蓄積・改善されていく——という当初の構想が、
実装レベルでは成立しませんでした。

### 3. HolographicMemory の容量が実用に耐えない

**データ容量が想定より大きく**、実用に供するのは難しいと判断しました。

分散表現は本来コンパクトな符号化を期待できる方式ですが、
この実装では容量効率が想定に届きませんでした。

### 判断：パッチではなく、基盤から作り直す

3つの問題に共通していたのは、**推論の実行そのものを制御・検証する仕組みが
アプリケーション層に散在していた**ことです。

個別に対処するのではなく、
**決定論的で、実行が有界で、検証可能な言語基盤を先に作る**——
という判断に至りました。これが [ReasonScript](reasonscript.md) 開発の直接の動機です。

---

## ReasonScript / MRA はこれにどう応えているか

v1 の3つの限界は、そのまま基盤側の設計要件になっています。

| v1 の限界 | 基盤側での対応 |
|---|---|
| **推論爆発** | ReasonScript が探索を **ExecutionPlan として明示化**し、実行を有界に保つ。`1,000 live value` ポリシー、境界のある autograd ライフサイクル管理、ループトレーススナップショットの有界化など、リソース上限が言語ランタイムの責務になっている |
| **推論爆発**（ドメイン側） | [VisionWorldModel](visionworldmodel.md) が **`adaptive_budget.rsn`（推論予算）· `adaptive_oscillation.rsn`（判断の振動検出）· `adaptive_abstention.rsn`（棄権）** を独立モジュールとして持つ。爆発の制御が推論の一級要素になっている |
| **学習能力の不足** | MRA が連想記憶に学習を期待せず、**正規知識は Molecular Memory に明示的に書く**設計に変更（[Truth Boundary](../design-philosophy.md#原則2--記憶と真実を分離するtruth-boundary)）。記憶が勝手に賢くなることを前提にしない |
| **容量の問題** | HolographicMemory を**再構築可能な非正規層**として位置づけ直し、正規データから作り直せるようにした。容量が問題なら破棄・再構築できる |

**特に3つ目が本質的です。** v1 は HolographicMemory に「学習して賢くなること」を期待していました。
MRA はその期待を明示的に捨て、**連想記憶は候補を出すだけ**と定義し直しています。
Truth Boundary は理念的な原則であると同時に、**v1 の失敗から得られた実務的な結論**でもあります。

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
| コードの想起 | 記憶空間 crate 群で自前実装 | MRA の連想活性化機構 |
| 検証 | 独自の評価エンジン | ReasonScript の検証契約 + MRA の Evidence 検証 |

これにより DBM は、**ソフトウェア設計というドメインに固有の推論**に集中できます。
視覚（VisionWorldModel）· 言語（LanguageModel）と同じ基盤の上で、
同じ Molecule 表現を使って設計知識を扱う——という構図です。

**設計案とシステム構造を Molecule として表現できれば、コードの想起は
MRA の連想活性化そのものになります。** v1 で自前に作り込んだ記憶空間の機構を
基盤側に移せるのは、この対応が成立するためです。

v1 で確立した以下の設計は、v2 にも引き継がれる想定です。

- Planner → Executor → Validation → Repair Loop → Convergence Control という推論ループ
- コマンド分類による実行安全制御
- Agent Operational Charter による権限階層と役割分担

→ [設計思想の詳細](../design-philosophy.md) · [開発年表](../timeline.md)
