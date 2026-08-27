# ReasonScript

## Summary

推論を状態遷移として記述するための、決定論的な言語処理系。**Python を実行系、Rust をランタイムとする Hybrid DSL** として、ゼロから設計・実装した。ポートフォリオ全体の基盤であり、MRA の3つのドメインモデルはすべてこの上に構築される。

| | |
|---|---|
| **Status** | **VALIDATED**（対象範囲: コンパイルパイプラインとCI検証済みの言語コア。ReasonGraph/World ビューア・パッケージレジストリ・SDK公開APIマニフェストは未実装） |
| **Role** | 設計・実装のすべて(個人開発、AIコーディングエージェント併用) |
| **Languages** | Python(実行系・ツールチェーン)、Rust(ランタイム)、TypeScript/Go/Java(DTOバインディングのみ) |
| **Period** | 2026-04 〜 現在(v0.5.4.5, 2026-08-07) |
| **Repository** | [chigenori053/ReasonScript](https://github.com/chigenori053/ReasonScript)(Apache-2.0) |
| **Evidence** | `./reason ci --json` 実行、全ステージ PASS・1,116件のテスト通過を確認(2026-08-12, commit `0efb2ab`, Python 3.14.0) |
| **Reproducibility** | `pip install -e .` 後 `./reason ci --json` で第三者が再現可能 |

---

## Problem

LLM を使った推論ワークフローには構造的な弱点がある。

- 同じ入力でも実行結果が変わり、再現性がなく検証もデバッグもできない
- なぜその結論に至ったかが残らず、事後に監査できない
- 失敗したときに安全に戻せる単位が定義されていない

これらを、ライブラリやプロンプト技法ではなく**言語仕様のレベルで**解決することを目指した。

### 直接の開発動機

先行プロジェクト [Design_BrainModel v1](design_brainmodel.md) が、推論爆発・記憶機構の学習不足・容量の非現実性という3つの限界で実用水準に届かなかったことが直接の契機である。「推論の実行を制御・検証する仕組みがアプリケーション層に散在している」という診断から、決定論的で実行が有界、検証可能な言語基盤を先に作るという判断に至った。

---

## Approach

推論を**知識を書く言語ではなく、状態遷移を書く言語**として設計した。

> *"The Semantic Language is **not** a knowledge representation language. It is a **semantic reasoning state-transition language**."*
> — `ReasonScript_Semantic_Language_Core_v0.2.md`(2026-06-15 凍結)

中核となる4原則:

1. Knowledge is not primitive. Knowledge is generated.
2. Reasoning precedes Knowledge.
3. Every Knowledge object contains complete evidence.
4. Semantic reasoning is deterministic.

言語の意味論は6つの状態遷移プリミティブで定義される。

```
  goal      → 望ましい状態を宣言する
  derive    → 候補となる推論を生成する
  prove     → 導出を検証する
  apply     → 検証済みの変更をコミットする(状態を書き込む唯一の操作)
  converge  → 状態を安定させる
  rollback  → 直前の安全な状態へ戻す(状態を復元する唯一の操作)
```

`Proof` の内部文字列に `invalid` が含まれる場合、決定論的な証明失敗として扱われ、直前の安全な `State` チェックポイントへの自動ロールバックが起動する。状態を書き込む操作を `apply` と `rollback` の2つに限定することで、ロールバック安全性を言語レベルで成立させている。

---

## Architecture

決定論的コンパイルパイプライン(4段階の中間表現):

```
  .rsn ソース
      │
      ▼
  Surface AST        ← 構文の忠実な表現
      │
      ▼
  Semantic AST       ← 名前解決・型・スコープの確定
      │
      ▼
  Reason IR          ← 推論の正規中間表現(JSON Schema で検証)
      │
      ▼
  ExecutionPlan      ← 決定論的な実行計画
      │
      ▼
  InferenceResult    ← 検証済みの実行結果
```

同一入力からは必ず同一の `ExecutionPlan` と `InferenceResult` が生成される。Reason IR は `schemas/reason_ir.schema.json` によって機械的に検証される。

### Hybrid DSL 構成

| 層 | 言語 | 担当 | 規模 |
|---|---|---|---|
| 実行系 / ツールチェーン | **Python** | コンパイルパイプライン、`reason` CLI、検証、CI、アーティファクト管理、SDK、Conformance | 543ファイル / 82,244行 |
| ランタイム | **Rust** | 実行エンジン、Reason IR 検証、トランザクション、テンソル演算、可視化 | 197ファイル / 45,134行 |
| DTO バインディング | TypeScript / Go / Java | 型定義の共有のみ | 計 約6,200行 |

Python 側(`toolchain/native_runtime.py`)が、配布物に同梱されたネイティブ Rust 実行ファイル(`reasonunit-runtime-native`)を解決して呼び出す。目的別に7つのランタイム(`RuntimeReal` / `HybridRuntime` / `NativeReasonUnitRuntime` / `ClusterRuntime` / `VisionRuntime` / `VisualizationRuntime` / `RuntimeComplex`)を持つ。詳細は[ReasonScript Runtime ケーススタディ](../case-studies/reasonscript_runtime.md)を参照。

`Rust / Python / TypeScript / Go / Java` の5言語が単一の規範 DTO 契約を共有する(`Common_DTO_Specification_v0.1.md`)。Go と Java は DTO バインディングのみで、処理系の実装言語ではない。

---

## My Responsibilities

- **問題設定・仮説形成** — Design_BrainModel v1 の3つの限界を、パッチではなく言語基盤の再設計が必要な構造的問題と診断
- **仕様策定** — 状態遷移意味論(6プリミティブ)、型仕様、名前空間解決、操作的意味論、ABI仕様など40本以上の仕様書を実装に先立って執筆
- **アーキテクチャ判断** — Python(実行系)/ Rust(ランタイム)の Hybrid 構成、4段階中間表現パイプライン、7ランタイムの使い分けを決定
- **実装** — 言語処理系・ツールチェーン・CI・Conformance フレームワークの設計と実装のすべて(AIコーディングエージェントを実装補助として併用)
- **受入条件・検証設計** — CI パイプラインの段階構成、Golden コーパス、決定論ゲートの設計
- **最終的な技術判断** — 仕様の凍結タイミング(`Semantic Language Core v0.2` を2026-06-15に凍結)、非推奨化の方針

AI コーディングエージェントには実装の生成・定型的なテストコード作成を委任し、仕様の妥当性・型システムの整合性・決定論保証の成立可否は本人が検証している。詳細は[AI支援開発の責任分界](../methodology/ai_assisted_development.md)を参照。

---

## Implemented Scope

- 状態遷移意味論、4段階中間表現パイプライン、7種のランタイム
- `reason` CLI(ビルド・実行・検証・CI・アーティファクト管理)、`reason view`(CodeViewer)、`reason cluster`
- IDE(`apps/reasonscript-ide`)、VS Code 拡張(MIT)、ブラウザ Playground、LSP サーバ(Phase 1)
- Tensor Training Foundation v0.2(NCHW Conv2d/MaxPool2d/AvgPool2d、リバースモード自動微分、`.rstensor` ファイルプロファイル)
- Conformance フレームワーク(`conformance/run_conformance.py`)

未実装(README に明記): ReasonGraph/World ビューアの完全版、パッケージレジストリ、LSP シンボルインデックスのコンパイラソーススパンへの移行、SDK 公開APIマニフェスト。

---

## Validation Evidence

| 観点 | 内容 | 証拠 |
|---|---|---|
| CI パイプライン | チェックアウト→ワークスペース検証→診断→アーティファクト→Golden コーパス→エージェントプロトコル→DTO互換性→テストスイート | `./reason ci --json` 実行、**全ステージ PASS・1,116件通過**(2026-08-12, commit `0efb2ab`, Python 3.14.0で実測) |
| Schema validation | Reason IR の JSON Schema 検証 | `schemas/reason_ir.schema.json` |
| Golden test | ソース→中間表現の期待値比較 | `golden/` コーパス |
| Deterministic planning | 同一入力から同一 ExecutionPlan/InferenceResult | コンパイルパイプライン設計 + CI 通過 |
| Rollback | `Proof` に `invalid` を含む場合の自動ロールバック | 言語意味論として実装(`apply`/`rollback` の2操作限定) |
| Artifact integrity | `.rstensor` のチェックサム検証・capability チェック | `reason tensor import\|inspect\|verify` |
| Cross-language contract | 5言語が単一の DTO 契約を共有 | `Common_DTO_Specification_v0.1.md` + `dto/` バインディング |

README は v0.5.4.5 時点でテスト数を1,085件と記載しているが、本ポートフォリオ作成時に実際に実行したところ全ステージ PASS・1,116件の通過を確認した(差分はテスト追加による)。

---

## Reproduction

```bash
git clone git@github.com:chigenori053/ReasonScript.git
cd ReasonScript
pip install -e .

./reason ci --json
./reason reasoning-runtime run examples/v0_8/reasoning_runtime/animal_isa.rsn --json
```

プラットフォーム別インストーラは `docs/installation/`(Linux / macOS / Windows)。想定実行時間: CI フル実行で数分程度(規模非公開)。

---

## Results

- 状態遷移意味論を設計し、証明失敗時の自動ロールバックを言語意味論に組み込んだ
- Python 実行系と Rust ランタイムを分離した Hybrid 構成を構築し、ネイティブ実行ファイル解決を実装
- 4段階の中間表現による決定論的コンパイルパイプラインを設計・実装
- 5言語が単一の規範 DTO 契約を共有するクロス言語バインディングを構築
- `./reason ci` の実行により全ステージ PASS・1,116件のテスト通過を実測確認

---

## Limitations

- ReasonGraph/World ビューア、パッケージレジストリ、SDK 公開APIマニフェストは未実装
- `Legacy/elixir_runtime/` に初期検討していた分散ランタイム実装が残っているが、設計収束により現行構成からは外れている(実装本体には含まれない)
- クロス言語 DTO 契約は「単一契約を共有する設計になっている」ことの確認であり、5言語間の相互運用を統合的に検証する自動テストは未整備

---

## Current Status

v0.5.4.5 リリース済み。CI 全ステージ PASS・1,116テストを実行確認済み(VALIDATED)。周辺ツール(ビューア・レジストリ)は未実装。

---

## Next Step

- ReasonGraph/World ビューアの完成
- パッケージレジストリの設計
- SDK 公開APIマニフェストの整備
- LanguageModel・VisionWorldModel など応用ドメインからのフィードバックに基づく言語仕様の拡張

---

## Repository and Documents

- **リポジトリ**: [chigenori053/ReasonScript](https://github.com/chigenori053/ReasonScript)
- **主要仕様書**: `ReasonScript_Language_Specification_v0.1.md`、`ReasonScript_Semantic_Language_Core_v0.2.md`、`ReasonScript_Operational_Semantics_v0.1.md`、`Common_DTO_Specification_v0.1.md`、`Conformance_Framework_Specification_v0.1.md` ほか(`docs/specifications/` に40本以上)
- **関連ページ**: [ReasonScript Runtime ケーススタディ](../case-studies/reasonscript_runtime.md) · [決定論的推論ケーススタディ](../case-studies/deterministic_reasoning.md) · [MRA](mra.md) · [研究系譜](../history/research_lineage.md)
