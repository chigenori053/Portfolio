# Project Index

全7プロジェクトの一覧。Status定義は[Statusモデル](#status定義)を参照。

## 基盤

| プロジェクト | 概要 | 主言語 | 規模 | Status | ライセンス |
|---|---|---|---|---|---|
| **[ReasonScript](reasonscript.md)** | 推論を記述する状態遷移記述言語。決定論的実行とロールバック安全性を言語仕様で保証 | Hybrid DSL(Python実行系/Rustランタイム) | 約133,600行・CI 1,116件パス | **VALIDATED** | Apache-2.0 |

## MRA — Molecular Reasoning Architecture

| プロジェクト | 概要 | 規模 | Status | ライセンス |
|---|---|---|---|---|
| **[MRA(アーキテクチャ全体)](mra.md)** | 知識をMoleculeとして表現し、視覚・言語・ソフトウェア設計へ展開する推論アーキテクチャ | 仕様書中心 | **EXPERIMENTAL** | 全権利留保 |
| **[VisionWorldModel](vision_world_model.md)** | 視覚ドメインモデル。観測と推論を分離し、判断を保留できる世界モデル | 約7,700行 | **VALIDATED**(Phase 3C-1まで) | 全権利留保 |
| **[LanguageModel](language_model.md)** | 言語ドメインモデル。連想記憶と正規知識を分離 | 仕様書中心・実装約450行 | **PROPOSED**(Phase 0まで) | 全権利留保 |
| **[Design_BrainModel](design_brainmodel.md)** | ソフトウェア設計ドメインモデル(計画)。v1は設計案からコードを想起するコーディングエージェント | 約57,000行・60+ crate | **PAUSED**(v1)/PROPOSED(v2) | 全権利留保 |

## 探索期

| プロジェクト | 概要 | 主言語 | 規模 | Status | ライセンス |
|---|---|---|---|---|---|
| **[COHERENT](coherent.md)** | 理論検証プロジェクト(推論モデル名: BrainModel)。非Transformer推論の成立可能性を検証 | Python | 約43,000行 | **EXPERIMENTAL** | 全権利留保 |
| **[mathlang](mathlang.md)** | 数学学習支援言語。過程を第一級のデータにするという発想の出発点 | Python | 約9,200行 | **PAUSED** | Apache-2.0 |

---

## Status定義

| Status | 定義 |
|---|---|
| **VALIDATED** | 実装、実行、測定、受入条件の通過を確認済み(対象範囲を併記) |
| **IMPLEMENTED** | 実装済みだが、十分な比較評価または外部検証前 |
| **EXPERIMENTAL** | 仮説検証中であり、結果が確定していない |
| **PROPOSED** | 設計または構想段階 |
| **PAUSED** | 意図的に開発を停止し、必要時のみ保守する |
| **ARCHIVED** | 現行設計では使用しない過去成果 |

詳細は[Evidence検証マトリクス](../evidence/verification_matrix.md)を参照。

→ [ポートフォリオ トップ(日本語)](../README_ja.md) · [研究系譜](../history/research_lineage.md)
