# MRA — Molecular Reasoning Architecture

## Summary

知識を**型付き Atom と Bond からなる Molecule**として表現し、Evidence と Provenance を伴う正規データとして扱う推論アーキテクチャ。[ReasonScript](reasonscript.md) を実装手段とし、視覚([VisionWorldModel](vision_world_model.md))・言語([LanguageModel](language_model.md))・ソフトウェア設計([Design_BrainModel v2](design_brainmodel.md#next-step--v2-への再設計))という3つの異なるドメインへ、同一の表現形式と推論契約で展開することを狙う長期研究テーマ。

| | |
|---|---|
| **Status** | **EXPERIMENTAL**(データモデルと Truth Boundary は仕様として確立。3ドメインへの統一的な展開はまだ検証されていない) |
| **Role** | アーキテクチャ設計・仕様策定(個人研究、AIコーディングエージェント併用) |
| **Languages** | 仕様書中心(ReasonScript / Python で各ドメインモデルが実装) |
| **Period** | 2026-08 〜 現在(構想は2026-04の ReasonScript 開発時点から) |
| **Repository** | 単独リポジトリはなし。[VisionWorldModel](https://github.com/chigenori053/VisonWorldModel) / [LanguageModel](https://github.com/chigenori053/LanguageModel) / [Design_BrainModel](https://github.com/chigenori053/Design_BrainModel) の各リポジトリに分散実装 |
| **Evidence** | Truth Boundary の定式化(LanguageModel 仕様書 §2.1)、VisionWorldModel Phase 1〜3C-1 の検証済み実装 |
| **Reproducibility** | 各ドメインモデルのリポジトリ単位で再現手順あり(下記プロジェクトページ参照) |

---

## Problem

単一のニューラルモデルに言語能力・知識・推論のすべてを担わせると、知識の更新が難しく、根拠が示せず、幻覚が原理的に排除できない。同様に、連想記憶(ベクトル類似度検索・ホログラフィック記憶)をそのまま知識として扱う設計は、「意味的に近い」ことと「その関係が成立する」ことを混同する。

**MRA が解こうとしている問題**: 連想記憶を「候補を出す層」に限定し、事実の確定を正規データと Evidence 検証に委ねる構造を、ドメインに依存しない共通の表現形式として確立できるか。

---

## Approach

知識を **Molecule**(型付き Atom と Bond からなる検証可能な単位)として表現する。Molecule には **Evidence**(主張を支持または反証する参照可能な根拠)と **Provenance**(入力・抽出器・モデル・時刻・変換履歴などの来歴)が付随する。

### Truth Boundary — 中心原理

> HolographicMemory は Semantic Activation Field(意味活性化の場)であり、真実の保存先ではない。ベクトル類似度、復号結果、ニューラルモデルの確信度だけを根拠として Relation を断定してはならない。
> — *MRA Holographic Semantic Language Model 仕様書 v0.1, §2.1*

表現の三層分離:

| 層 | 表現 | 正規性 |
|---|---|---|
| Semantic Space | dense vector / semantic frame | 非正規 |
| HolographicMemory | bound / superposed vector | 非正規・再構築可能 |
| MRA Molecular Memory | typed molecule graph | **正規** |

「HolographicMemory の削除・再構築によって正規知識が失われてはならない」— 連想記憶は捨てて作り直せるキャッシュとして位置づけられている。この原則は [COHERENT](coherent.md) の想起→検証(Accept/Review/Reject)、[VisionWorldModel](vision_world_model.md) の観測→推論の分離にも共通する系譜上の到達点である。

### 3ドメインへの展開という検証設計

応用が1つしかない基盤は「その応用のために作ったもの」にしか見えない。**視覚・言語・ソフトウェア設計という無関係な3ドメインが、同一の表現形式と推論契約を共有できるかどうかが、MRA の妥当性そのものの検証になる**、というのが設計上の立場である。

各ドメインモデルは基盤(ReasonScript)を改変せず、公開 CLI と決定論的契約のみを利用する。LanguageModel は基盤をコミットハッシュ単位(`7f29c1c`)で固定している。

---

## Architecture

```
            ★ MRA — Molecular Reasoning Architecture
              知識を型付き Atom / Bond からなる Molecule として表現
                 │
     ┌───────────┼────────────────────────┐
     ▼           ▼                        ▼
 VisionWorldModel   LanguageModel      Design_BrainModel v2
  視覚ドメイン       言語ドメイン         ソフトウェア設計ドメイン
  VALIDATED          PROPOSED            PROPOSED
  (Phase 3C-1まで)    (Phase 0まで)        (再設計計画段階)
```

### データモデル

| 種別 | 表すもの |
|---|---|
| Concept Molecule | 人・物・抽象概念などの同一性と属性 |
| Relation Molecule | subject / predicate / object 等の関係 |
| Event Molecule | agent / patient / recipient / 時刻を伴う出来事 |
| Experience Molecule | 状況・行為・結果・観測を関連付けた経験記録 |

HRR(Holographic Reduced Representation)/ VSA(Vector Symbolic Architecture)に基づく Binding / Superposition / Cleanup を用いた分散表現を採用する(詳細は [LanguageModel](language_model.md) 参照)。

---

## My Responsibilities

- **アーキテクチャ判断** — Molecule/Atom/Bond による表現形式の設計、Truth Boundary の定式化
- **仕様策定** — 三層分離の MUST/MUST NOT 規範、Evidence/Provenance のデータモデル
- **ドメイン展開の設計** — 視覚・言語・ソフトウェア設計という3ドメインへの展開方針と、各ドメインモデルが基盤を改変しないための固定(lock)戦略
- **判断根拠** — Design_BrainModel v1 が推論爆発・記憶の学習不足・容量非現実性という3つの限界に直面した経験から、「連想記憶に学習を期待しない」という設計へ転換した判断

MRA 自体は現時点でコード実装を持たず、仕様策定とドメインモデルへの展開設計が中心。各ドメインモデルの実装範囲・責任分界は [VisionWorldModel](vision_world_model.md)・[LanguageModel](language_model.md)・[Design_BrainModel](design_brainmodel.md) の各ページを参照。

---

## Implemented Scope

- Truth Boundary の仕様化(LanguageModel 仕様書 §2.1 として文書化)
- Molecule / Evidence / Provenance のデータモデル定義(スキーマ: `molecule.schema.json`、`semantic_frame.schema.json`)
- VisionWorldModel における「不変な候補の構築→グラフ検証→明示的な状態移行→アトミックなコミット」という構造変更モデルの実装(Phase 1〜3C-1)
- LanguageModel における `foundation.lock.json` による基盤固定戦略の実装

MRA アーキテクチャそのものを統合するランタイム・共通ライブラリはまだ存在しない。各ドメインモデルは個別のリポジトリで、共通の設計原則に従いながら独立に実装されている。

---

## Validation Evidence

| 観点 | 内容 | 証拠 |
|---|---|---|
| Truth Boundary の実装 | 想起は候補、確定は検証という分離 | COHERENT の Accept/Review/Reject + DecisionLog、VisionWorldModel の観測/推論分離 |
| Atomic state transitions | 不変な候補→グラフ検証→状態移行→アトミックコミット | VisionWorldModel Phase 3B-1〜3B-3(検証コマンド・テスト・JSON成果物あり) |
| Provenance | 来歴の記録 | LanguageModel Audit Logger 設計(棄却理由まで記録) |
| 基盤固定によるバージョン管理 | コミットハッシュ単位の依存固定 | LanguageModel `foundation.lock.json`(`7f29c1c31a06ad70abc4024bc4655873c43797b3`) |

**未検証の目標**: 3ドメイン間で同一の Molecule 表現と推論契約が実際に機能するかは、LanguageModel の実装がこれからのため未確認。VisionWorldModel 単体では構造変更モデルが機能することを確認済みだが、これは MRA 全体の妥当性を意味しない。

---

## Reproduction

MRA 自体の統一的な再現手順はない。各ドメインモデルのリポジトリで個別に再現する。

```bash
# VisionWorldModel(最も検証が進んでいるドメインモデル)
git clone https://github.com/chigenori053/VisonWorldModel.git
cd VisonWorldModel
reason ci --json
```

---

## Results

- Truth Boundary を、ドメインを問わず適用できる規範として定式化した
- Molecule / Evidence / Provenance のデータモデルをスキーマとして確立した
- VisionWorldModel でこのモデルに基づく構造変更を実装し、Phase 3C-1 まで検証を完了した
- LanguageModel で基盤固定(lock)という再現性確保の方法を確立した

---

## Limitations

- **MRA 全体としての実装はまだ存在しない。** 各ドメインモデルは共通原則に従うが、共通ランタイム・共通ライブラリはない
- 3ドメイン展開という検証設計そのものが未完了(LanguageModel は Phase 0、Design_BrainModel v2 は計画段階)
- Design_BrainModel v1 の記憶機構(MemorySpace/HolographicMemory)が実運用規模で学習能力・容量の面で機能しなかった経緯があり、MRA の Molecular Memory がその代替として同水準の規模で機能するかは未実証

---

## Current Status

EXPERIMENTAL。データモデルと Truth Boundary は仕様として確立(VALIDATED相当の設計確認)。3ドメイン展開はドメインごとに進捗が異なり、VisionWorldModel が最も先行している。

---

## Next Step

- LanguageModel の Holographic Core 実装(Phase 1)着手
- Design_BrainModel v2 の設計着手(v1 の crate 群を MRA Base + ReasonScript へ委譲)
- 3ドメインが実際に同一の Molecule 表現を共有できるかの統合検証

---

## Repository and Documents

- 単独リポジトリなし。ドメインモデル: [VisionWorldModel](https://github.com/chigenori053/VisonWorldModel) · [LanguageModel](https://github.com/chigenori053/LanguageModel) · [Design_BrainModel](https://github.com/chigenori053/Design_BrainModel)
- **中核仕様書**: `MRA_Holographic_Semantic_Language_Model_Specification_v0.1.md`(LanguageModel リポジトリ)
- **関連ページ**: [VisionWorldModel](vision_world_model.md) · [LanguageModel](language_model.md) · [Design_BrainModel](design_brainmodel.md) · [決定論的推論ケーススタディ](../case-studies/deterministic_reasoning.md) · [研究系譜](../history/research_lineage.md)
