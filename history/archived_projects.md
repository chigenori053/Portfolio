# Archived Projects & Superseded Designs

## 現状

本ポートフォリオが対象とする7プロジェクトのうち、ARCHIVED(現行設計では使用しない過去成果)に分類されるプロジェクトは**現時点で存在しない**。mathlangは開発停止(PAUSED)だが、そのDSL設計は現行プロジェクト群に発想として継承されており、Design_BrainModel v1も同様にPAUSEDだが、決定論ゲートの設計はReasonScriptに直接引き継がれている。詳細はそれぞれの[project_index.md](../projects/project_index.md)のStatus定義を参照。

## 内部で置き換えられた・不採用になった設計

プロジェクト全体ではなく、個別コンポーネントとしてARCHIVED相当となった設計を記録する。

### ReasonScript: Elixir分散ランタイム(不採用)

開発初期に、分散ランタイムとしてElixirの導入が計画されていた。その後の設計収束によって不要となり、現行のPython(実行系)/ Rust(ランタイム)のHybrid構成に一本化された。実装は `Legacy/elixir_runtime/` にのみ残存し、現行の実装本体には含まれない。

### Design_BrainModel: V1 API(非推奨化・移行済み)

PhaseA-Finalの時点で、以下のAPIがV2へ移行し、V1は非推奨化された(`since = "PhaseA-Final"`、`note = "Will be removed in PhaseC"` を付与)。

```
HybridVM::snapshot            → HybridVM::snapshot_v2
HybridVM::compare_snapshots   → HybridVM::compare_snapshots_v2
HybridVM::explain_design      → HybridVM::explain_design_v2
HybridVM::rebuild_l2_from_l1  → HybridVM::rebuild_l2_from_l1_v2
```

### COHERENT: 「計算削減80%実証」という記述(訂正・撤回)

2026-08-12、外部レビューを受けた検証で、README・年表・設計思想・技術経歴書の4箇所に記載していた「計算削減80.0%・誤想起率0%」という記述が、実ワークロードの性能測定ではなく5件の固定シナリオによる設計確認に過ぎないことが判明し、撤回・訂正した。経緯の詳細は[AI支援開発の責任分界](../methodology/ai_assisted_development.md#事例-過大主張の検出と訂正)を参照。

## Design_BrainModel v1 — 過去成果ではなく先行実装として位置づけ

Design_BrainModel v1は開発停止(PAUSED)だが、ARCHIVEDには分類していない。理由は、v1が独立したRustプロジェクトとして推論・記憶・アーキテクチャ評価の機構をすべて自前で構築したことで、**MRAとReasonScriptが満たすべき要件を洗い出した先行実装**という価値を持ち続けているため。決定論ゲート(FNV-1a、`{:.6}`固定精度、`1e-6`閾値)の設計は、そのままReasonScriptの決定論的コンパイルパイプラインに引き継がれている。詳細は[Design_BrainModelプロジェクトページ](../projects/design_brainmodel.md)を参照。

→ [研究系譜](research_lineage.md) · [プロジェクト一覧](../projects/project_index.md)
