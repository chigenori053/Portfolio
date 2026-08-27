# ケーススタディ: ReasonScript Runtime — Hybrid DSLの設計と決定論の維持

## 概要

ReasonScript は、**Python を実行系、Rust をランタイム**とする Hybrid DSL である。単一言語で処理系を実装するのではなく、言語処理系の反復開発に強い Python と、実行時の安全性・決定性に強い Rust を分離し、両者を跨いでも決定論が壊れないコンパイルパイプラインを設計した。

## 課題

言語処理系を作る際、次の対立がある。

- **開発速度**: パーサ・検証ロジック・ツールチェーンは頻繁に変更されるため、反復開発に強い言語が望ましい
- **実行時保証**: 実行エンジン・テンソル演算・可視化ランタイムは、型による安全性と決定性が必要
- 単一言語ですべてを賄うと、どちらかが妥協になる

## Surface AST から ExecutionPlan までの変換

ソースコードは4段階の中間表現を経て、決定論的な実行結果になる。

```
  .rsn ソース
      │
      ▼
  Surface AST        ← 構文の忠実な表現(Python側)
      │
      ▼
  Semantic AST       ← 名前解決・型・スコープの確定(Python側)
      │
      ▼
  Reason IR          ← 推論の正規中間表現、JSON Schemaで検証(Python側)
      │
      ▼
  ExecutionPlan      ← 決定論的な実行計画(Rustランタイムへ引き渡し)
      │
      ▼
  InferenceResult    ← 検証済みの実行結果
```

同一入力からは必ず同一の `ExecutionPlan` と `InferenceResult` が生成される。この保証が、Python(コンパイル)と Rust(実行)という異なる言語・プロセスをまたいでも成立する必要がある。

## Python / Rust Runtime の役割分担

| 層 | 言語 | 担当 | 規模 |
|---|---|---|---|
| 実行系/ツールチェーン | Python | コンパイルパイプライン、`reason` CLI、検証、CI、アーティファクト管理、SDK、Conformance | 543ファイル/82,244行 |
| ランタイム | Rust | 実行エンジン、Reason IR検証、トランザクション、テンソル演算、可視化 | 197ファイル/45,134行 |

Python 側(`toolchain/native_runtime.py`)が、配布物に同梱されたネイティブ Rust 実行ファイル(`reasonunit-runtime-native`)を解決して呼び出す構造を取る。プロセス境界を越える呼び出しであるため、中間表現(Reason IR / ExecutionPlan)が両言語間の**規範的な契約**として機能する必要があり、これを JSON Schema で機械検証している。

## Unified Execution Runtime Architecture — 目的別の複数ランタイム

単一のランタイムではなく、目的別に7種類のランタイムを持つ構成を取っている。

| ランタイム | 役割 |
|---|---|
| `RuntimeReal` | 標準の実行ランタイム |
| `HybridRuntime` | Rust実装のハイブリッドランタイム。Reason IRバリデータを含む |
| `NativeReasonUnitRuntime` | ReasonUnitのネイティブ実行 |
| `ClusterRuntime` | Dynamic ReasonUnitクラスタ実行 |
| `VisionRuntime` | 視覚処理向けランタイム |
| `VisualizationRuntime` | **Safe-Rust** による意味構造の可視化ランタイム。実行時に構造が壊れないことを型で保証 |
| `RuntimeComplex` | 複素数演算ランタイム |

複数ランタイムに分けている理由は、用途ごとに要求される保証の水準が異なるため。特に `VisualizationRuntime` は可視化という「壊れても実害が小さい」ように見える領域であっても、Safe-Rust を選んで型による安全性を維持している。

## Local / Cluster の責務

`ClusterRuntime` は Dynamic ReasonUnit のクラスタ実行(計画・実行・シミュレーション・検証・比較)を担う。`reason cluster` CLI サブコマンドから、単一プロセス内の `RuntimeReal` と、複数 ReasonUnit にまたがる `ClusterRuntime` を使い分ける構成になっている。ローカル実行での決定論保証(同一入力→同一ExecutionPlan)が、クラスタ実行でも維持されることが前提となる。

## 決定論維持の検証

| 検証内容 | 方法 | 結果 |
|---|---|---|
| コンパイルパイプライン全体 | `./reason ci --json` | 全ステージ PASS・1,116件テスト通過(2026-08-12, commit `0efb2ab`, Python 3.14.0で実測) |
| Reason IRのスキーマ適合 | `schemas/reason_ir.schema.json` による検証 | CI内で自動検証 |
| Goldenコーパス比較 | ソース→中間表現の期待値比較 | `golden/` ディレクトリ、CI内で自動検証 |
| ロールバック安全性 | `Proof` に `invalid` を含む場合の自動ロールバック | 言語意味論として実装(状態を書き込む操作を`apply`/`rollback`の2つに限定) |

## 得られた知見

- **言語境界を跨ぐ決定論の保証には、中間表現をスキーマで固定することが不可欠。** Python側とRust側で異なる実装であっても、Reason IRという規範的な契約さえ守れば、両者は独立に検証・置き換え可能になる
- **単一のランタイムに全ての用途を担わせるより、目的別に分けたほうが、各ランタイムに要求する保証水準を明確にできる。** 特にSafe-Rustのような型システムの恩恵は、「壊れても実害が小さそうな」領域にこそ効果的に適用できる
- Hybrid構成は開発初期に分散ランタイム(Elixir)を検討した経緯もあるが、設計収束の結果、現行のPython/Rust構成に落ち着いた(`Legacy/elixir_runtime/`として履歴のみ残存)

→ [ReasonScriptプロジェクトページ](../projects/reasonscript.md) · [決定論的推論ケーススタディ](deterministic_reasoning.md)
