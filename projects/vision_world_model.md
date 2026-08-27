# VisionWorldModel

## Summary

観測されたものと推論されたものを決して混ぜない世界モデル。**MRA の視覚ドメインモデル**であり、ReasonScript で実際にドメインモデル全体を記述した最初の本格的な実装。根拠が不十分なら判断を保留する `ACCEPT` / `REVISE` / `DEFER` / `ABSTAIN` の4値判定を採用する。

| | |
|---|---|
| **Status** | **VALIDATED**(Phase 1〜3C-1、検証コマンド・テスト・JSON成果物あり)。適応的構造推論以降は **EXPERIMENTAL** |
| **Role** | 設計・実装のすべて(個人開発、AIコーディングエージェント併用) |
| **Languages** | ReasonScript(`src/` の全モデル定義)、Python(約7,700行、検証スクリプト) |
| **Period** | 2026-07 〜 現在 |
| **Repository** | [chigenori053/VisonWorldModel](https://github.com/chigenori053/VisonWorldModel)(リポジトリ名は `Vison` のタイポ。URL維持のためそのまま)。全権利留保 |
| **Evidence** | Phase 1〜3C-1 各段階の検証コマンド・pytest・機械可読成果物(JSON) |
| **Reproducibility** | `reason ci --json` および各 Phase の検証スクリプトで再現可能 |

---

## Problem

AIが世界の状態を推定するとき、観測(実際に見えたもの)と推論(そこから導いた結論)の境界をどう守るか。この境界が曖昧だと、推論結果が事後的に「観測された事実」として扱われてしまう。

同時に、MRA が知識を型付き Atom / Bond からなる Molecule として表現するというアーキテクチャ上の主張を、正解が曖昧さなく決まる実在の分子構造(原子・結合・価数・構造異性)を題材に検証する。

---

## Approach

観測された `VisualAtom` と推論された World 構成要素を、**永続化レベルで分離**する。判断は `ACCEPT`(受理)/ `REVISE`(修正して再検討)/ `DEFER`(保留)/ `ABSTAIN`(棄権)の4値で返し、根拠が不十分であることを明示的な出力として扱う。判断の各結果はハッシュで連結され、後から追跡可能。

構造変更は一貫した操作モデルに従う:

```
  1. 不変な候補(immutable candidate)を構築
  2. グラフ検証(graph validation)
  3. 明示的な状態移行(explicit state migration)
  4. アトミックなコミット(atomic commit)
```

既存構造を直接書き換えず、新しい世代を作って切り替える。構造の世代は `water@generation-1` / `water@generation-2` のように明示的に指定する。

`adaptive_reversibility.rsn`(可逆性評価)と `adaptive_abstention.rsn`(棄権判断)が独立モジュールとして存在し、「戻せるか」「やらないでおくか」を推論の一級要素として扱う。

---

## Architecture

`src/` 配下はすべて `.rsn`(ReasonScript ソース)で構成される。

```
src/
  atom.rsn / bond.rsn
  atom_state.rsn / bond_state.rsn
  bond_formation_request.rsn / bond_dissociation_request.rsn
  adaptive_risk_evaluation.rsn / adaptive_cost_evaluation.rsn
  adaptive_reversibility.rsn / adaptive_abstention.rsn
  adaptive_budget.rsn / adaptive_oscillation.rsn
  adaptive_structural_reasoning_main.rsn ...
```

Phase 構成(各Phaseが検証コマンド・pytest・JSON成果物・レポートをセットで持つ):

| Phase | 内容 |
|---|---|
| Phase 1 | 分子構造の固定表現 |
| Phase 2 | 状態遷移の検証 |
| Phase 3A | 状態→推論へのマッピング(`CLASS_A`/`CLASS_B`/`UNDETERMINED`の三値、フィクスチャ名や入力IDによる参照を一切使わない設計) |
| Phase 3B-1 | 結合状態の遷移(`enabled`/`transmission`/`mode` の検証済みアトミック提案) |
| Phase 3B-2 | 結合型の遷移とバージョニング(`bond_type` を構造的アイデンティティとして扱う) |
| Phase 3B-3 | 結合の形成と解離 |
| Phase 3C-1〜 | 分子の分割と結合、複数分子の相互作用、適応的構造推論 |

---

## My Responsibilities

- **問題設定** — 「観測と推論の境界をどう守るか」という問いの設定、分子構造という検証題材の選定理由(表現形式と対象領域の構造が一致し、正解が曖昧さなく決まる)
- **アーキテクチャ判断** — 4値判定(ACCEPT/REVISE/DEFER/ABSTAIN)、不変候補→検証→移行→コミットという操作モデルの設計
- **仕様策定・受入条件** — 各 Phase の検証コマンド・成果物形式(JSON)の定義
- **不正なショートカットの禁止** — Phase 3A で「判断は状態トレースからの evidence 抽出のみで導出し、フィクスチャ名や入力IDによる参照を一切使わない」という制約を明記し、テストを通すための答えの先読みを構造的に禁止
- **実装** — `.rsn` によるドメインモデル全体の記述(AIコーディングエージェントを実装補助として併用)

---

## Implemented Scope

- 分子構造(原子・結合)の固定表現と状態遷移(Phase 1〜2)
- 状態→推論マッピング(Phase 3A)、結合状態・結合型の遷移(Phase 3B-1〜2)、結合の形成/解離(Phase 3B-3)
- 適応的構造推論モジュール群(リスク評価・コスト評価・可逆性評価・推論予算・振動検出・棄権判断)
- VisualAtom/World 構成要素の分離を持つ視覚テスト(画像→世界モデル変換の可視化、任意PNG/JPEGの解析)

---

## Validation Evidence

| 観点 | 内容 | 証拠 |
|---|---|---|
| Atomic state transitions | 不変候補→グラフ検証→状態移行→アトミックコミット | Phase 3B-1〜3B-3 の検証スクリプト + JSON成果物 |
| Schema validation / Invalid-input rejection | 分子構造の固定表現に対する検証 | `scripts/molecular_validation.py suite` → `artifacts/validation_summary.json` |
| 不正なショートカット禁止の検証 | フィクスチャ名参照を使わない evidence 抽出 | Phase 3A 仕様記述 + `scripts/reasoning_mapping_validation.py suite` |
| Cross-runtime compatibility | ReasonScript 公開CLIのみを利用し基盤を改変しない | `reason.toml`(`reason_version = ">=0.5.1"`) |
| Regression prevention | プロジェクト全体の検証 | `reason ci --json` |

```bash
python3 scripts/vision_world_model_validation.py suite
python3 -m pytest -q tests/test_vision_world_model_validation.py
```

---

## Reproduction

```bash
git clone https://github.com/chigenori053/VisonWorldModel.git
cd VisonWorldModel
# ReasonScript >=0.5.1 がインストール済みであること
reason ci --json

python3 scripts/molecular_validation.py suite
python3 scripts/state_transition_validation.py suite
python3 scripts/vision_visual_test.py demo
```

`artifacts/vision_world_model/visual_tests/index.html` で VisualAtom・VisualBond・スコア・Decision のオーバーレイを比較できる。

---

## Results

- ReasonScript でドメインモデル全体を記述し、自作言語の実用性を検証した
- 観測(VisualAtom)と推論(World構成要素)を永続化レベルで分離した
- ACCEPT/REVISE/DEFER/ABSTAIN の4値判定を実装し、判断の保留・棄権を正常系として設計した
- 不変候補構築→グラフ検証→状態移行→アトミックコミットという構造変更モデルを確立した
- Phase 1〜3C-1の各段階で検証コマンド・テスト・機械可読成果物を整備した

---

## Limitations

- 適応的構造推論(複数分子の相互作用など Phase 3C-1 以降)は EXPERIMENTAL であり、Phase 1〜3C-1 ほどの検証密度に達していない
- リポジトリ名に `Vison`(iの欠落)というタイポがあり、URL維持のため訂正していない
- 視覚ドメイン以外(言語・ソフトウェア設計)との統合検証はまだ行われていない(MRA 全体としての妥当性はこの1ドメインだけでは確認できない)

---

## Current Status

Phase 3C-1 まで検証完了(VALIDATED)。適応的構造推論に着手中(EXPERIMENTAL)。

---

## Next Step

- 複数分子の相互作用のさらなる検証
- 適応的構造推論の Phase を刻んだ検証の継続
- LanguageModel・Design_BrainModel v2 との Molecule 表現の整合確認

---

## Repository and Documents

- **リポジトリ**: [chigenori053/VisonWorldModel](https://github.com/chigenori053/VisonWorldModel)
- **主要ドキュメント**: `docs/VisionWorldModel_Phase2_Guide_ja.md` ほかドキュメント37本
- **関連ページ**: [MRA](mra.md) · [LanguageModel](language_model.md) · [決定論的推論ケーススタディ](../case-studies/deterministic_reasoning.md) · [研究系譜](../history/research_lineage.md)
