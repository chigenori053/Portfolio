# Design_BrainModel (DBM)

## Summary

設計意図からコードを「想起」するコーディングエージェント。人間と合意した設計案からシステム構造を生成し、その構造からコードを想起する三段階パイプラインを目指す。**v1 は未完成プロダクト**であり、直面した3つの限界が [ReasonScript](reasonscript.md) 開発の直接の動機になった。v2 は MRA のソフトウェア設計ドメインモデルとして再設計を計画している。

| | |
|---|---|
| **Status** | **PAUSED**(v1 は開発停止。既知の限界を解決しないまま実用水準に達しておらず、ReasonScript + MRA Base による v2 再設計を計画中) |
| **Role** | 設計・実装のすべて(個人開発、AIコーディングエージェント併用) |
| **Languages** | Rust(約49,900行 / 539ファイル)、Python(約7,200行) |
| **Period** | 2026-01 〜(v1 開発停止、v2 未着手) |
| **Repository** | [chigenori053/Design_BrainModel](https://github.com/chigenori053/Design_BrainModel)。全権利留保(再設計中のため) |
| **Evidence** | Phase 6(大規模)・Phase 7(実リポジトリ)・Phase 8(人間評価)の機械可読レポート(JSON) |
| **Reproducibility** | `cargo build -p design_cli --bin dbm` でビルド・実行可能。ただし推論爆発の抑制は根拠が不十分(下記 Limitations 参照) |

---

## Problem

フロンティアのコーディングエージェントはコード生成には強いが、以下の課題を抱える。

| 課題 | DBM のアプローチ |
|---|---|
| エージェントは設計意図を保持しない | `design.md` → `DesignUnit` による意図の永続化 |
| LLM出力は非決定的 | 構造化された実行・検証フローによる制御 |
| AI生成の変更がアーキテクチャを壊しうる | 設計・コード・実行を横断した推論 |
| 修正ループがアドホックになりやすい | Repair を推論プロセスの正式な一部として扱う |
| 安全でない実行がプロジェクトを破壊しうる | コマンド分類による実行安全制御 |

> *"DBM doesn't compete with Claude Code — it provides the structure and safety that Claude Code tends to lack."* — v1 README

---

## Approach

三段階パイプライン: 開発コンセプト/設計案(人間との合意形成)→ システム構造(検証可能な中間層)→ コード。三段目の動詞が「生成」ではなく**「想起」**であることが中心的な主張であり、[COHERENT](coherent.md) の Resonance Recall と同じ機構をコード生成ドメインに適用したもの。

このパイプラインは ReasonScript のコンパイルパイプラインと同型である — 入力から出力へ一足飛びに変換せず、検証可能な中間表現を必ず経由する。

### 決定論ゲート — 仕様で凍結された再現性

再現性の実現手段を仕様レベルで凍結している。

| 項目 | 固定値 |
|---|---|
| ハッシュアルゴリズム | FNV-1a 64bit |
| シード | `0xcbf29ce484222325` |
| 浮動小数点の整形 | `{:.6}` の固定精度文字列 |
| リスト型フィールド | ハッシュ前にソート |
| `Option` の符号化 | `null` または `some:<value>` |
| 空文字列と `None` | 区別する |
| テンプレート選択の曖昧性閾値 | `TEMPLATE_SELECTION_EPSILON = 1e-6` |

決定論ゲートは、同一入力に対して `snapshot_v2` ハッシュ・テンプレート選択・説明テキストが完全一致することを要求する。

### Agent Operational Charter

`AGENT.md` により、権限階層(`specs/ > Rule.md > TASK_STATE.yaml > 実装コード`)と役割分担を仕様化している。

| 役割 | 担当 | できること | できないこと |
|---|---|---|---|
| Architect | 人間 | 仕様承認、対立の解決、最終決定 | — |
| ResearchAgent | Gemini CLI | 仕様案生成、代替設計 | 本番コードを書かない |
| CodingAgent | Codex CLI | 承認済み仕様の実装、テスト生成 | アーキテクチャを再定義しない |
| ValidationAgent | — | 仕様と実装の整合検証、回帰チェック | — |

---

## Architecture

```
┌─────────────────────────────────────┐
│          Design Intent              │
│      design.md / DesignUnit         │   ← 設計意図を構造化して永続化
└──────────────────┬──────────────────┘
                   │ anchored to
┌──────────────────▼──────────────────┐
│        DesignBrainModel (DBM)       │
│  Planner → Executor → Validation    │
│  Repair Loop → Convergence Control  │
│  Execution Safety                   │
└──────────────────┬──────────────────┘
                   │ operates on
┌──────────────────▼──────────────────┐
│       Development Environment       │
└─────────────────────────────────────┘
```

60を超える crate が責務ごとに分離されている(アーキテクチャ推論・記憶空間・意味層・実行・設計推論・制御・モデルの各系統)。アプリケーション層は `apps/`(cli・desktop・gui・lsp・server)。

---

## My Responsibilities

- **問題設定** — 「コーディングエージェントが生成したコードが設計から逸脱するのをどう防ぐか」という問いの設定
- **アーキテクチャ判断** — 三段階パイプライン、決定論ゲートの仕様凍結範囲、60+ crate への責務分割
- **AI協働の統制設計** — Agent Operational Charter による権限階層と役割分担(提案する者と実装する者を権限で分離)
- **実装** — Rust ワークスペース全体の設計と実装(AIコーディングエージェントを実装補助として併用)
- **失敗の抽象化レベル判定と再設計判断** — 推論爆発・記憶機構の学習不足・容量非現実性という3つの症状を、個別バグではなく「推論の実行を制御・検証する仕組みがアプリケーション層に散在している」という構造的問題と診断し、パッチではなく基盤からの作り直しを決定
- **自己評価の基準適用** — 「落ちなくなったこと」と「安定していること」を区別し、根拠がない限り後者を主張しないという基準を自身のプロダクト評価に適用(下記 Limitations 参照)

---

## Implemented Scope

- 自律実行ループ(Planner → Executor → Validation → Repair Loop → Convergence Control)
- REPL ベースの対話、コマンド分類による実行安全制御
- 決定論ゲート(FNV-1a ハッシュ、`snapshot_v2` 比較)
- Agent Operational Charter(`AGENT.md`)
- API バージョニング・非推奨化(`since`/`note` 属性による V1→V2 移行設計)

進行中: Git 読み取り専用統合、制限付き git add/commit、設計意図統合(DesignUnit)。計画中: アーキテクチャ認識リファクタ、GitHub/PR統合、UI/ダッシュボード。

---

## Validation Evidence

| 観点 | 内容 | 証拠 |
|---|---|---|
| Byte-identical artifacts | 同一入力で `snapshot_v2` ハッシュ・テンプレート選択・説明テキストが完全一致 | 決定論ゲートの仕様 + `DESIGN.md` |
| Canonical serialization | ハッシュ前のソート、`Option` の符号化規則、空文字列と`None`の区別 | `DESIGN.md` 正規化規則 |
| 大規模検証 | Phase 6(大規模)・Phase 7(実リポジトリ)・Phase 8(人間評価) | `phase6_large_scale_report.json`、`phase7_real_repository_report.json`、`phase8_human_evaluation_report.json` |
| Regression prevention | アーキテクチャ・メモリ空間・世界モデルの検証レポート | `architecture_benchmark_report.json`、`memoryspace_verification_report.json`、`worldmodel_verification_report.json` |

---

## Reproduction

```bash
git clone https://github.com/chigenori053/Design_BrainModel.git
cd Design_BrainModel
cargo build -p design_cli --bin dbm
cargo test -p design_cli

# REPL
cargo run -p design_cli -- --repl
```

対象環境: macOS(Apple Silicon 最適化)・Cargo・zsh。

---

## Results

- 60を超える crate による Rust ワークスペースを責務ごとに分離して設計した
- 決定論の実現手段(ハッシュアルゴリズム、シード、浮動小数点整形精度、曖昧性閾値)を仕様で凍結した
- Planner → Executor → Validation → Repair Loop → Convergence Control という自律実行ループを実装した
- Agent Operational Charter として複数AIエージェントの権限階層と役割分担を仕様化した
- Phase 6(大規模)・Phase 7(実リポジトリ)・Phase 8(人間評価)まで検証を実施した

---

## Limitations

**v1 は未完成プロダクトである。** 実用水準に届かなかった原因を3つに切り分けている。

1. **推論爆発** — 構造化推論の実行時に Unit 候補の生成が非線形に増加し、探索空間が制御を超えて膨張してシステムフリーズに至った。**現状は強引に抑え込んでいる状態であり、安定稼働に至ったと判断する材料がない。**「落ちなくなったこと」と「安定していること」は別である、という基準をここに適用している。
2. **学習能力の不足** — 中核の推論 Core であった MemorySpace と HolographicMemory が、期待していた学習能力を発揮しなかった。想起した経験から設計知識が蓄積・改善される、という構想が実装レベルでは成立しなかった。
3. **容量の非現実性** — HolographicMemory のデータ容量が想定より大きく、実用に供するのは難しいと判断した。

これら3つは個別のバグではなく、「推論の実行そのものを制御・検証する仕組みがアプリケーション層に散在している」という構造的な問題と診断した。

---

## Current Status

PAUSED。v1 は開発停止し、既知の限界(推論爆発・学習能力不足・容量非現実性)を解決しないまま実用水準に届いていない。ReasonScript + MRA Base による v2 再設計を計画している。

---

## Next Step — v2 への再設計

| 関心事 | v1 | v2(計画) |
|---|---|---|
| 決定論的実行 | `hybrid_vm` 等で自前実装 | ReasonScript の ExecutionPlan |
| 知識・記憶表現 | `memory_space_*`・`chm`/`dhm`/`shm` を自前実装 | MRA Base の Molecule/Evidence/Provenance |
| 設計意図の永続化 | `DesignUnit`(独自形式) | MRA の型付き Molecule として表現 |
| コードの想起 | 記憶空間 crate 群で自前実装 | MRA の連想活性化機構 |

v1 で確立した Planner→Executor→Validation→Repair Loop、コマンド分類による実行安全制御、Agent Operational Charter は v2 にも引き継ぐ想定。v1 は破棄されるものではなく、**MRA と ReasonScript が満たすべき要件を洗い出した先行実装**として位置づけている。

---

## Repository and Documents

- **リポジトリ**: [chigenori053/Design_BrainModel](https://github.com/chigenori053/Design_BrainModel)
- **主要文書**: `AGENT.md`(Agent Operational Charter)、`DESIGN.md`(決定論ゲート仕様)
- **関連ページ**: [ReasonScript](reasonscript.md)(v1の限界が動機となった基盤) · [MRA](mra.md)(v2の位置づけ) · [COHERENT](coherent.md)(記憶機構の原型) · [研究系譜](../history/research_lineage.md)
