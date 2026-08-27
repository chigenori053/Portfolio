# COHERENT

## Summary

**Transformer に依らない推論は成立するか**を検証する理論検証プロジェクト。中核の推論モデル名は **BrainModel**。物理現象である光学干渉を数理的にシミュレートした HolographicMemory と、想起優先(Recall-First)の推論アーキテクチャによって、「情報の密度を上げることでニューラルネットワークに頼らない推論が可能になるか」という仮説を検証する。

| | |
|---|---|
| **Status** | **EXPERIMENTAL**(項目により証拠の強さが大きく異なる。詳細は Validation Evidence 参照) |
| **Role** | 仮説設計・実装・検証のすべて(個人研究、AIコーディングエージェント併用) |
| **Languages** | Python 3.12(約43,000行 / 327ファイル)、Jupyter Notebook |
| **Period** | 2025-11 〜 現在 |
| **Repository** | [chigenori053/COHERENT](https://github.com/chigenori053/COHERENT)(全権利留保) |
| **Evidence** | `p1_word_gen_baseline.csv` ほか実測共鳴値つきCSV、`PHASE_A_REPORT.md` 等の実行結果レポート |
| **Reproducibility** | 単語・多言語想起と数式判定は再現可能。文字生成(カタカナ・漢字)は生成スクリプト未収録のため再現不可 |

---

## Problem

Transformer(自己注意機構によるニューラルネットワーク)以外の推論モデルで、LLM と同様または近い推論を実現できるか。製品開発ではなく理論検証が目的。

中核仮説: **情報の密度を上げることで、ニューラルネットワークに頼らない推論が可能になるか。**

巨大なパラメータ集合による近似ではなく、表現そのものの情報密度によって推論を成立させられないか、という問い。

---

## Approach

2つの技術で仮説を検証する。

**① HolographicMemory(光学記憶)** — 光学干渉を数理的にシミュレーションし、正しい/あいまい/間違いの三値判定と、言語生成(想起)ができるかを検証する。Dynamic / Static / Causal の3層構成で、`MemoryOrchestrator` が層間の昇格・因果リンク誘導を制御する。

**② MemorySpace(永続記憶+計算空間)** — 記憶があれば想起(Recall)、なければ推論を実行(Compute)し、結果を永続化する運用が、計算資源を高効率に運用できるかを検証する。

```
  推論要求 → 記憶を検索 ─該当あり→ 想起(Recall、計算コストほぼゼロ)
                    └─該当なし→ 推論を実行(Compute) → 結果を永続化
```

判定結果は `ProcessingResult` として、`Action`(判定)と `DecisionLog`(根拠)を必ず伴って返される。この「判定に必ず根拠ログを添える」設計が、後の MRA における Evidence / Provenance につながっている。

---

## Architecture

**Recall-First** アーキテクチャを採用し、System 1(直感的想起)と System 2(論理的推論)のハイブリッドという構図を取る。

| レイヤー | 責務 |
|---|---|
| Interface(Layer A) | Semantic Parser — 自然言語を Semantic IR に変換 |
| Core(Layer B) | Action Executor(実行・状態更新)、Tracer(実行ログ記録) |
| Physics(Layer C) | Optical Engine — 複素数演算による記憶の想起と干渉シミュレーション |

`MemorySpace` は `AcceptStore` / `ReviewStore` / `RejectStore` の3領域と `MemoryRouter` で構成される。テキスト・画像・音声を複素数テンソル(Holographic Tensor)に統一符号化し、同一空間で類似度ではなく**干渉**として想起を定式化する。

---

## My Responsibilities

- **仮説形成** — 「情報密度がニューラルネットワークの代替になり得るか」という中核仮説の設定
- **アーキテクチャ判断** — Recall-First 構成、3層 HolographicMemory、Accept/Review/Reject の三値判定設計
- **実験設計** — Recall Boundary Sweep(θ掃引)、単語・多言語想起(60語、複数手がかり条件)、文字生成(カタカナ・漢字)、数式正誤判定の各実験を設計
- **結果解釈と証拠強度の切り分け** — 実測データに基づく成果(単語想起)と、設計確認に留まる成果(計算削減効果)を明確に区別
- **失敗の抽象化レベル判定** — 漢字生成が60〜80%に留まった際、次元拡大(1024→2048)で正解率が変化しないことを確認し、原因を「記憶容量の不足」ではなく「属性定義の重複」と特定
- **過大主張の訂正** — 当初「計算削減80%を実証」としていた記述を、一次データ(`PHASE_B_LOG.json`)を検めた結果、共鳴スコアがテストコード内の定数であり性能としては未測定であることを確認し、訂正した(詳細は[AI支援開発の責任分界](../methodology/ai_assisted_development.md)を参照)

---

## Implemented Scope

- HolographicMemory(Dynamic / Static / Causal の3層)とオーケストレータ
- MemorySpace(Accept/Review/Reject の三値判定 + DecisionLog)
- 属性ホログラムからの動的な記号生成(記号そのものは保存しない設計)
- SymbolicEngine(SymPy 統合)による数式の正誤・同値判定
- 自然言語の条件文理解、意味構造からの文生成と逆解析による意味一致検証
- Streamlit による記憶干渉・推論過程のリアルタイム可視化 UI

---

## Validation Evidence

| 検証項目 | 結果 | 証拠の強さ | 一次データ |
|---|---|---|---|
| 単語・多言語の想起 | 60語で100%、混在環境での劣化率0.00% | **実測データあり** | `p1_word_gen_baseline.csv`(実測共鳴値つき60行) |
| 数式の正誤・同値判定 | 成功(交換法則・結合法則・簡約含む) | **実行結果あり** | `PHASE0_REPORT.md` / `PHASE_A_REPORT.md`(10ケース) |
| 三値判定(Accept/Review/Reject)の成立 | 実装・運用されている | **コードで確認可能** | `coherent/core/memory/space/` |
| 自然言語の条件文理解 | 18/18 | 実行結果あり | `PHASE_C_REPORT.md` |
| 文字生成(カタカナ・漢字) | カタカナ100%・漢字60〜80% | ⚠️ **再現不可** | レポートのみ。生成スクリプト未収録 |
| 記憶再利用による計算削減 | θ掃引で意図どおりの挙動 | ❌ **未測定**(性能としては) | 5件の固定シナリオによる設計確認のみ。共鳴スコアはテスト内の定数 |
| LLM 同等の汎用推論 | 未到達 | — | — |

**計算削減効果についての訂正**: `report/PHASE_B_REPORT.md` は想起しきい値 θ を振ったときに Recall/Compute の切り替えが意図どおり動くかを確認するテストであり、サンプル数5件・共鳴スコアはテスト内定数という制約を持つ。「計算量を80%削減した」という当初の記述は誇張であり、実ワークロードでの計算削減率は未測定である。

---

## Reproduction

```bash
git clone https://github.com/chigenori053/COHERENT.git
cd COHERENT
uv init && uv python install 3.12 && uv sync

# 単語・多言語想起の再現
python3 -m pytest report/ -k word_gen

# Streamlit 可視化UI
uv run streamlit run coherent/tools/memory_simulator/app.py
```

Python 3.12 以上が必要。単語想起・数式判定・自然言語理解は再現可能。文字生成(カタカナ・漢字)は生成スクリプトが未収録のため**再現不可**であることを明示する。

---

## Results

- 日英60語の想起で100%・混在劣化率0.00%を実測データつきで達成
- 数式の正誤・同値判定(交換法則・結合法則・簡約・項順序差分を含む)を実証
- Accept/Review/Reject の三値判定と根拠ログの設計を確立し、実装で運用
- 属性ホログラムから記号を動的生成する言語生成を部分的に達成(カタカナ100%、漢字60〜80%)
- 漢字生成の限界原因を、記憶容量ではなく属性定義の重複と特定(反証ではなく符号化設計の課題として切り分け)

---

## Limitations

- 「記憶があれば計算しない」という中核の効率化仮説は、性能としては未測定(5件の固定シナリオによる設計確認のみ)
- カタカナ・漢字生成の結果は生成スクリプトが現在のリポジトリに含まれておらず、第三者による再現ができない
- 漢字生成の正解率が60〜80%に留まっており、実用的な文字生成には至っていない
- LLM と同等の汎用推論には到達していない

---

## Current Status

EXPERIMENTAL。仮説の全面的な立証には至っていないが、否定もされていない。単語・多言語想起という最も裏付けの強い成果と、記憶再利用による効率化という未測定の中核仮説が併存している。

---

## Next Step

- 記憶再利用による計算削減効果の実測(代表的な問題集合・ベースライン・共鳴スコアの実測が必要)
- 文字生成(特に漢字)の生成スクリプトの再整備と、より詳細な属性定義(第2部首・画数)による識別精度の改善
- 実ワークロードでの Recall/Compute 切り替えの性能測定

---

## Repository and Documents

- **リポジトリ**: [chigenori053/COHERENT](https://github.com/chigenori053/COHERENT)
- **主要ドキュメント**: `COHERENT_System_Architecture.md`、`COHERENT_LM_Architecture.md`、`CausalScript_DSL_Specification.md`、`CRS_Memory_Library_Spec_v0_1.md`
- **関連ページ**: [MRA](mra.md)(Truth Boundary の起点) · [Design_BrainModel](design_brainmodel.md)(記憶機構が実運用規模で直面した限界) · [研究系譜](../history/research_lineage.md)
