# LanguageModel — MRA Holographic Semantic Memory

## Summary

モデルパラメータだけに言語能力・知識容量・推論能力のすべてを担わせず、**連想記憶と正規知識を厳密に分離**した言語モデル基盤の研究。**MRA の言語ドメインモデル**であり、その仕様書は MRA 全体で共有される中核概念(Molecule・Evidence・Provenance・Truth Boundary)を最初に定式化した文書でもある。

| | |
|---|---|
| **Status** | **PROPOSED**(Phase 0 = 基盤固定・仕様策定が完了。実装はこれから) |
| **Role** | 仕様策定(個人研究、AIコーディングエージェント併用) |
| **Languages** | Python、ReasonScript(予定) |
| **Period** | 2026-08 〜 現在 |
| **Repository** | [chigenori053/LanguageModel](https://github.com/chigenori053/LanguageModel)。全権利留保 |
| **Evidence** | `foundation.lock.json` によるコミットハッシュ単位の基盤固定、スキーマ定義済み(`semantic_frame.schema.json`、`molecule.schema.json`) |
| **Reproducibility** | 仕様検証コマンドで再現可能(実装コードは約450行のみ) |

---

## Problem

現在の言語モデルは、言語能力・知識・推論のすべてを単一のパラメータ集合に押し込んでいる。その結果、知識の更新が難しく、根拠が示せず、幻覚が原理的に排除できない。

**検証しようとしている仮説**: 自然言語文から得た意味構造を分散複合表現へ符号化し、容量増加や干渉のある条件でも、関連する正規 Molecule を高再現率で候補化できるか。また、候補を Evidence と Provenance により検証し、元の Relation Structure を正確かつ再現可能に復元できるか。

---

## Approach

「言語モデルにすべてをやらせる」のではなく、4つの担い手に責務を分ける。

| 担い手 | 責務 |
|---|---|
| Neural Model | 言語知覚、意味写像、自然言語生成 |
| HolographicMemory | 分散的な連想、文脈活性化、パターン補完 |
| MRA Molecular Memory | 明示的知識・状態・経験・根拠・来歴の**正規**保存 |
| ReasonScript / MRA Runtime | 検証、決定論的操作、推論規則の実行 |

### Truth Boundary(仕様書 §2.1 より)

1. HolographicMemory の出力は常に候補とスコアであり、事実判定ではない
2. ユーザーへの事実応答は、正規 Molecule の取得と Evidence 検証を通過しなければならない
3. ベクトル類似度・復号結果・ニューラルモデルの確信度だけを根拠として Relation を断定してはならない
4. HolographicMemory の削除・再構築によって正規知識が失われてはならない
5. 正規知識の変更は Molecular Memory の書き込み経路だけで行う

---

## Architecture

```
Language Input → Input Normalizer → Semantic Encoder → Typed Semantic Frame
  → Holographic Encoder → HolographicMemory Index
  → Candidate Molecule IDs + Activation Scores     ← ここまでは「候補」でしかない
  → Molecular Memory Retrieval
  → Schema / Evidence / State Validation           ← ここで初めて事実になる
  → Model A/B/C Reasoning Interface → Response Molecule → Language Decoder
```

`Audit Logger` が入出力・版・閾値に加え**棄却理由**まで記録する。「答えなかった」ことも監査対象になる。

### 基盤の固定(foundation.lock.json)

```json
{
  "reason_script": {
    "version": "0.5.4.5",
    "commit": "7f29c1c31a06ad70abc4024bc4655873c43797b3"
  }
}
```

> `LanguageModel` は基盤を変更せず、その公開 CLI と決定論的な Reasoning 契約を利用する独立プロジェクトである。

---

## My Responsibilities

- **仮説形成** — Relation Recovery(意味構造の高再現率な復元)を検証可能な最小システムとして切り出す判断
- **アーキテクチャ判断** — 4者(Neural Model / HolographicMemory / Molecular Memory / Runtime)への責務分割、Truth Boundary の MUST/MUST NOT 規範化
- **スコープ確定** — v0.1 の対象・非対象を明示的に規定(大規模事前学習モデルの新規学習、自律エージェント、HolographicMemory からの直接応答などを非対象として除外)
- **再現性の要件定義** — GPU等による非決定性が残る場合は「その範囲と許容差を記録する」という現実的な基準の策定
- **基盤固定の判断** — 自作言語(ReasonScript)を自分で使う際に基盤側を都合よく書き換えないという規律を、コミットハッシュ単位の lock として明文化

---

## Implemented Scope

- 仕様書(`MRA_Holographic_Semantic_Language_Model_Specification_v0.1.md`)
- `foundation.lock.json` による基盤固定
- スキーマ定義: `semantic_frame.schema.json`、`molecule.schema.json`
- Phase 0 設計文書: `PHASE0_FOUNDATION.md`、`PHASE1_HOLOGRAPHIC_CORE.md`、`REASONSCRIPT_PYTHON_BOUNDARY.md`

実装コードは約450行のみで、**現時点での主成果物は仕様書**である。Concept/Relation/Event/Experience Molecule の実装、Holographic Encoder、Cleanup 処理などはこれから。

---

## Validation Evidence

| 観点 | 内容 | 証拠 |
|---|---|---|
| Schema validation | Semantic Frame / Molecule の構造検証 | `schemas/semantic_frame.schema.json`、`schemas/molecule.schema.json` |
| Provenance | 来歴の記録設計 | Audit Logger 仕様(棄却理由まで記録) |
| バージョン固定による再現性 | 基盤をコミットハッシュ単位で固定 | `foundation.lock.json` |
| 検証コマンド | プロジェクト検証 | `reason project-validate . --json`、`reason check`、`python3 -m unittest tests/test_holographic.py -v` |

**現時点では実装が仕様に追従していないため、Relation Recovery の再現率など実証的な検証結果はまだ存在しない。**

---

## Reproduction

```bash
git clone https://github.com/chigenori053/LanguageModel.git
cd LanguageModel
../ReasonScript-v0.5.4.5/reason project-validate . --json
../ReasonScript-v0.5.4.5/reason check
python3 -m unittest tests/test_holographic.py -v
```

ReasonScript v0.5.4.5(指定コミット)が別ディレクトリに必要。

---

## Results

- Truth Boundary を MRA 全体で共有される中核原則として定式化した
- Molecule / Evidence / Provenance のデータモデルをスキーマとして確立した
- 基盤をコミットハッシュ単位で固定するという再現性確保の方法を確立した
- v0.1 のスコープ(対象・非対象)を明示し、スコープクリープを設計段階で防いだ

---

## Limitations

- 実装はほぼ存在せず(約450行)、Relation Recovery の再現率など実証的な結果はまだ得られていない
- Model A/B/C(学習・世界モデリング・推論)は現時点で interface stub のみ
- 汎用対話モデルとしての評価ではなく、最小システム(MHSM v0.1)の検証に限定されている

---

## Current Status

PROPOSED(Phase 0 = 基盤固定・仕様策定が完了)。Holographic Core の実装はこれから。

---

## Next Step

- Phase 1: Holographic Core(Semantic Encoder、Holographic Encoder、Cleanup)の実装
- Concept/Relation/Event/Experience Molecule の実装
- Relation Recovery の再現率を測定する評価データセットの整備

---

## Repository and Documents

- **リポジトリ**: [chigenori053/LanguageModel](https://github.com/chigenori053/LanguageModel)
- **中核仕様書**: `MRA_Holographic_Semantic_Language_Model_Specification_v0.1.md`
- **関連ページ**: [MRA](mra.md) · [COHERENT](coherent.md)(想起→検証の原型) · [VisionWorldModel](vision_world_model.md) · [研究系譜](../history/research_lineage.md)
