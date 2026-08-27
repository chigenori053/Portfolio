# mathlang

## Summary

数学学習支援言語。Python をベースにした DSL として、中学生以上の学生のプログラミング学習と数学学習の双方を支援することを目的に開発。研究のための実験言語ではなく、**学習支援プロダクト**として始まった、系譜全体の起点。

| | |
|---|---|
| **Status** | **PAUSED**(2025-11で更新停止。意図的な停止であり、発想は後続プロジェクトに継承済み) |
| **Role** | 企画・設計・実装のすべて(個人開発、AIコーディングエージェント併用) |
| **Languages** | Python 3.12(実装 約9,200行)、Jupyter Notebook |
| **Period** | 2025-11(開始・停止とも同月) |
| **Repository** | [chigenori053/mathlang](https://github.com/chigenori053/mathlang)(Apache-2.0) |
| **Evidence** | pytest によるパーサ・評価器の意味論テスト(30ファイル) |
| **Reproducibility** | `python main.py` 系のCLIで即座に再現可能 |

---

## Problem

数学教育において、最終解答だけでなく「なぜその変形をしたのか」という過程が失われがちである。プログラミング学習と数学学習を同時に支援するには、学習者が書く数式表記(教科書どおりの記法)をそのまま受け取りながら、正誤だけでなく「過程」を検証できるデータとして扱う必要があった。

---

## Approach

`step` を `before` / `after` / `note` の三点セットとして構文レベルで強制する。「何を、何に変え、なぜそうしたか」を書かないとプログラムにならない設計。

```text
step:
    before: (3 + 5) * 4
    after: 8 * 4
    note: simplify addition
```

### Python の資産を借り、学習者からは隠す

`MathLangInputParser` が教科書どおりの記法(`x^2` → `x**2`、`2xy` → `2*x*y`、`√x` → `sqrt(x)`、`(x-1)(x+1)` → `(x-1)*(x+1)`)を内部形式へ正規化する。仕様書はこの狙いを *"hides Python/SymPy-like syntax from educational users"* と明記している。

`SymbolicEngine` が SymPy をランタイムに統合し、`ValidationEngine` が学習者の解答を3モード(`symbolic_equiv`/`exact_form`/`canonical_form`)で判定する。**形の異なる複数の解答を正解として受理**しつつ、「数学的には正しいが求められた形ではない」を区別できる。

---

## Architecture

| レイヤー | 主要モジュール | 役割 |
|---|---|---|
| DSL Core | `core/parser.py`、`core/ast_nodes.py` | MathLang構文をASTに解析 |
| Execution | `core/evaluator.py` | 推論ステップを再生し注釈付き出力を生成 |
| SymbolicAI | `core/symbolic_engine.py` | SymPyと独自ロジックによる式の簡約・説明生成 |
| Causal | `core/causal/` | 誤りの原因推定、修正候補(最大3件)の提示 |
| Logging | `core/learning_logger.py` | 推論の全ステップを構造化JSONで記録 |

3層CLI構成(Edu / Pro / Demo)を持ち、`scenarios/config.json` でシナリオを定義してCLIフラグを変更せずに新しい `.mlang` プログラムを追加できる。

---

## My Responsibilities

- **企画** — 学習支援プロダクトとしての対象(中学生以上)と方針(過程を第一級のデータにする)の設定
- **DSL設計** — `before`/`after`/`note` を構文として強制する文法設計、`counterfactual`(反実仮想)節の設計
- **実装** — パーサ、評価器、SymbolicEngine統合、CausalEngine、LearningLoggerを含む全体実装(AIコーディングエージェントを実装補助として併用)
- **判断** — 研究目的の実験言語ではなく学習支援ツールとして開発方針を確定し、2025-11時点で開発を意図的に停止する判断

---

## Implemented Scope

- `MathLangInputParser`(数式表記の正規化)
- `SymbolicEngine`(SymPy統合 + 数値サンプリングによるフォールバック)
- `ValidationEngine`(3モードの正誤判定)
- `CausalEngine`(誤り原因推定、修正候補提示)
- `LearningLogger`(推論過程の構造化JSON記録)
- Edu/Pro/Demo の3層CLIとシナリオ機構

---

## Validation Evidence

| 観点 | 内容 | 証拠 |
|---|---|---|
| Invalid-input rejection / 正誤判定 | `symbolic_equiv`/`exact_form`/`canonical_form` の3モード判定 | `core/validation_engine.py` + pytest |
| Regression prevention | パーサ・評価器の意味論保護 | `tests/`(pytest、30ファイル) |
| Reproducibility | 同一手順から同一実行トレースが再生される | `LearningLogger` の構造化JSON出力 |

---

## Reproduction

```bash
git clone https://github.com/chigenori053/mathlang.git
cd mathlang

python main.py --file edu/examples/pythagorean.mlang
python main.py -c "problem: 1 + 1\nend: 2"
python -m edu.cli.main --scenario arithmetic
```

Python 3.12 環境で即座に実行可能。追加の外部依存は SymPy のみ。

---

## Results

- 数式の正誤判定を実装し、形の異なる複数の解答を正解として受理しつつ「求められた形ではない」を区別できる判定を実現した
- 誤りの原因を推定する因果解析エンジンを実装し、修正候補を複数(最大3件)提示できるようにした
- 変形の前後と根拠を構文として強制するDSLを設計し、反実仮想を言語機能として実装した
- 推論ステップを構造化JSONとして記録するLearningLoggerを実装した

---

## Limitations

- 2025-11以降、更新が停止しており、その後のPython/SymPyバージョンでの動作確認はしていない
- 学習効果や教育現場での利用実績についての外部評価は行っていない
- 対応する数学範囲は `docs/Supported_Math_Knowledge.md` に記載の範囲に限定される

---

## Current Status

PAUSED(2025-11で更新停止)。学習支援のために解いた問題(過程の記述・再生・同値判定)が後続すべてのプロジェクトの土台になっている。

---

## Next Step

現時点で再開の計画はない。ここで得られた設計(before/after/noteの構造化、同値だが形が違う解答の区別、誤り原因の複数候補提示)は [ReasonScript](reasonscript.md) の Reason IR、[VisionWorldModel](vision_world_model.md) の状態遷移検証、[Design_BrainModel](design_brainmodel.md) の Repair ループへと引き継がれている。

---

## Repository and Documents

- **リポジトリ**: [chigenori053/mathlang](https://github.com/chigenori053/mathlang)(Apache-2.0)
- **主要ドキュメント**: `docs/MathLangInputParser_Spec.md`、`docs/Supported_Math_Knowledge.md`、`docs/Math_Curriculum_Test_Items.md`
- **関連ページ**: [ReasonScript](reasonscript.md)(Reason IRへの発展) · [研究系譜](../history/research_lineage.md)
