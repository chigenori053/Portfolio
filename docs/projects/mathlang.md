# MathLang

> **数学学習支援言語。** Python をベースにした DSL として、
> 中学生以上の学生のプログラミング学習と数学学習の双方を支援することを目的に開発。

| | |
|---|---|
| **リポジトリ** | https://github.com/chigenori053/mathlang |
| **開始** | 2025-11（2025-11 で更新停止） |
| **目的** | **数学学習支援プロダクト**（対象: 中学生以上の学生） |
| **主要言語** | Python 3.12（実装 約9,200行）· Jupyter Notebook |
| **規模** | `core/` に23モジュール · テスト30ファイル · ドキュメント29本 |
| **ライセンス** | Apache-2.0 |
| **位置づけ** | 系譜の起点。「推論過程を第一級のデータにする」という発想の出発点 |

---

## 何のための言語か

MathLang は、研究のための実験言語として始まったものではありません。
**数学学習を支援するプロダクト**として開発されました。

リポジトリの構成にもそれが表れています。`edu/`（教育向け）配下には
`lessons/` · `examples/` · `notebooks/` · 専用CLI・UI が置かれ、
仕様書は対象ユーザーを **students / teachers** と明記しています。
`ExerciseSpec` は *"validating student answers"* のためのデータ構造として定義されています。

狙いは**プログラミング学習と数学学習を同時に支援する**ことです。
学習者は数学の問題を解く過程を「プログラムとして書く」ことになり、
その過程がそのまま検証・再生の対象になります。

### 設計方針

- **Process-first** — 最終解答よりも、途中の作業ステップを重視する
- **AI-assisted** — 人間の推論と SymbolicAI（SymPy）を組み合わせ、式の検証・説明・変換を行う
- **Reproducible** — 同一入力からは常に同一の実行トレースが再生される
- **Lightweight** — Python 3.12 のモジュール構成

「なぜその変形をしたのか」が消えてしまう、という数学教育の課題に対する、言語設計からの回答です。

---

## Python の資産を活用する設計

MathLang は Python をベースにした DSL です。
**Python の処理系と SymPy を土台に据えたうえで、その存在を学習者から隠す**という構造を取っています。

### ① 人間が書く数式を、Python が理解できる形に翻訳する

`MathLangInputParser` が、学習者が自然に書く数式表記を内部形式へ正規化します。

| 学習者の入力 | 内部表現（Python / SymPy 形式） |
|---|---|
| `(x - y)^2` | `(x - y)**2` |
| `2xy + 3x` | `2*x*y + 3*x` |
| `√x + 2y` | `sqrt(x) + 2*y` |
| `(x - 1)(x + 1)` | `(x - 1)*(x + 1)` |

> *"This parser hides Python/SymPy-like syntax from educational users
> while producing valid internal expressions."*
> — `docs/MathLangInputParser_Spec.md`

累乗記号 `^`、暗黙の乗算 `2xy`、平方根 `√`、隣接括弧による積——
**数学の教科書で使う記法をそのまま受け取る**ことが、学習支援ツールとしての前提条件でした。
`**` や `*` を明示的に書かせることは、数学の学習にとって本質的な負荷ではないからです。

正規化された式は、DSL パーサ · SymbolicEngine · Evaluator · KnowledgeRegistry の
すべてが共通で扱える形になります。

### ② SymbolicAI をランタイムに組み込む

`SymbolicEngine` が SymPy をランタイムに統合しています。

| メソッド | 役割 |
|---|---|
| `to_internal` | 内部表現への変換 |
| `is_equiv` | 2式が数学的に同値かを判定 |
| `simplify` | 式の簡約 |
| `evaluate` / `evaluate_numeric` | 値の評価 |
| `explain` | 変形（before → after）の説明を生成 |

SymPy が利用できない環境向けに `_fallback_is_equiv` と
**`_numeric_sampling_equiv`（数値サンプリングによる同値判定）** のフォールバックも実装されています。
式評価そのものは Python の `ast` モジュールを使った独自の評価器で行われます。

---

## 数式の正誤判定

学習支援プロダクトとしての中核機能です。
`ValidationEngine.check_answer()` が、学習者の解答を `ExerciseSpec`（問題定義）に対して検証します。

```python
ValidationResult(
    is_correct = bool,   # 正誤
    message    = str,    # 学習者へのフィードバック
)
```

### 3つの検証モード

| モード | 判定内容 |
|---|---|
| `symbolic_equiv` | **数学的に同値であればよい** — 表記の違いを許容する |
| `exact_form` | 正規化後に完全一致すること |
| `canonical_form` | 指定された標準形に一致すること |

**`symbolic_equiv` が学習支援として重要です。** 一つの問題に対して
**形の異なる複数の解答を、すべて正解として受理できます。**
`x^2 - 1` と `(x-1)(x+1)` は同値なので、どちらで答えても正解になる。

同時に `canonical_form` を使えば「同値だが、求められた形ではない」を区別できます。
実際のフィードバック文にもその区別が現れます。

> *"Your answer is mathematically correct, but not in the required form."*

**「数学的には合っているが、いま練習しているのはその形ではない」**——
これは学習支援ツールとして踏み込んだ判定です。単なる正誤の二値では表現できません。

### 誤りの原因推定と、複数の修正候補

`CausalEngine` が `LearningLogger` の記録を取り込み、誤りの原因を推定します。

| メソッド | 役割 |
|---|---|
| `why_error` | 直近の無効ステップを優先して原因を推定 |
| `suggest_fix_candidates(error_node_id, limit=3)` | **修正候補を複数（既定で最大3件）返す** |
| `counterfactual_result` | 前提を変えた場合の結果を算出（複数の介入に対応） |

エラー時には `== Causal Analysis ==` セクションとして、
推定原因と修正候補ステップが自動的に表示されます。

**「どこで間違えたか」だけでなく「なぜ間違えたか」と「どう直せるか」を複数案で返す**——
学習支援として、答え合わせの一歩先に踏み込んだ設計です。

---

## DSL の構文

```text
# MathLang DSL v2.5
meta:
    id: lesson_01
    topic: arithmetic
config:
    causal: true
    fuzzy-threshold: 0.6
problem: (3 + 5) * 4
prepare:
    - a = 3 + 5
    - b = a * 4
step:
    before: (3 + 5) * 4
    after: 8 * 4
    note: simplify addition
step:
    before: 8 * 4
    after: 32
end: 32
counterfactual:
    assume:
        a: 10
    expect: a * 3 + 2
```

実行結果：

```
Config: {'causal': True, 'fuzzy-threshold': 0.6}
Problem: (3 + 5) * 4
Prepare: a = 3 + 5
Prepare: b = a * 4
Step (unnamed): 8 * 4
Step (unnamed): 32
End: 32
```

### 構文要素

| 節 | 役割 |
|---|---|
| `meta` | 問題ID・トピックなどのメタデータ |
| `config` | 因果推論の有効化、ファジィ閾値などの設定 |
| `problem` | 解くべき問題 |
| `prepare` | 準備段階の代入・定義 |
| `step` | **`before` → `after` の変形と、その理由（`note`）** |
| `end` | 最終解答 |
| `counterfactual` | **反実仮想** — 「もし a が 10 だったら？」 |

`step` が `before` / `after` / `note` の三点セットになっているのが核心です。
変形の前後と**その根拠**が、構文として強制されます。

### 反実仮想（Counterfactual）

`counterfactual` 節により、「前提を変えたらどうなるか」を言語レベルで表現できます。
これは単なる再計算ではなく、**因果構造を持った思考実験**として扱われます。

---

## アーキテクチャ

| レイヤー | 主要モジュール | 役割 |
|---|---|---|
| **DSL Core** | `core/parser.py` `core/ast_nodes.py` | MathLang 構文を AST に解析 |
| **Execution** | `core/evaluator.py` | 推論ステップを再生し、注釈付き出力を生成 |
| **Polynomial** | `core/polynomial.py` `core/polynomial_evaluator.py` | 代数法則による多変数多項式の評価 |
| **Optimization** | `core/optimizer.py` | 代入のインライン化と定数畳み込みによるトレース整理 |
| **SymbolicAI** | `core/symbolic_engine.py` | SymPy と独自ロジックによる式の簡約と説明生成 |
| **Knowledge** | `core/knowledge_registry.py` `core/knowledge/` | 適用ルールの知識ベース |
| **Causal** | `core/causal/` | 誤りの原因推定 |
| **Fuzzy** | `core/fuzzy/` | ファジィ判定 |
| **Logging** | `core/learning_logger.py` `core/log_formatter.py` | 学習ログの記録と整形 |
| **Interfaces** | JupyterLab / Streamlit | 対話的な教材・デモ環境 |
| **Testing** | `tests/` (pytest) | パーサ・評価器の意味論を保護 |

その他、`arithmetic_engine.py` `fraction_engine.py` `unit_engine.py` `validation_engine.py`
`hint_engine.py` `i18n.py` など、教育用途に必要な要素が揃っています。

---

## 3層のCLI構成

用途別に Edu / Pro / Demo の3つのCLIを持ちます。

```bash
# ファイル実行
python main.py --file edu/examples/pythagorean.mlang

# インライン実行
python main.py -c "problem: 1 + 1\nend: 2"

# 多項式モード
python main.py --mode polynomial --file edu/examples/polynomial_arithmetic.mlang

# 反実仮想シミュレーション
python main.py --file edu/examples/counterfactual_demo.mlang \
  --counterfactual '{"phase": "step", "index": 2, "expression": "8 * 4"}'

# Edu / Pro / Demo
python -m edu.cli.main --scenario arithmetic
python -m pro.cli.main --mode causal --scenario basic
python -m demo.demo_cli --scenario minimal
```

### シナリオ設定ファイル

各CLIは `scenarios/config.json` にシナリオ定義を持ちます。
`file`・`mode`（`symbolic` / `polynomial` / `causal`）・`counterfactual` ペイロードを列挙しておくと、
`--scenario` が自動的に拾います。

**CLIのフラグを変更せずに新しい `.mlang` プログラムを追加できる**設計で、
CI から使う際に安定した契約になっています。

---

## 因果解析 — 間違いの「原因」を推定する

エラーが発生した場合、収集した `LearningLogger` レコードを元に因果推論エンジンが解析を行い、
`== Causal Analysis ==` セクションとして**推定原因と修正候補ステップ**が追加表示されます。

「どこで間違えたか」だけでなく「なぜ間違えたか」を返そうとする設計です。

---

## LearningLogger — 推論過程を機械可読にする

`problem → step → end` のイベントを（知識ベースが供給したルールIDを含めて）JSON として記録します。

```python
from pathlib import Path
from core.learning_logger import LearningLogger
from core.parser import Parser
from core.evaluator import Evaluator, SymbolicEvaluationEngine
from core.symbolic_engine import SymbolicEngine
from core.knowledge_registry import KnowledgeRegistry

source = """problem: (2 + 3) * 4
step: 5 * 4
end: 20"""

logger = LearningLogger()
program = Parser(source).parse()
symbolic_engine = SymbolicEngine()
registry = KnowledgeRegistry(Path("core/knowledge"), symbolic_engine)
engine = SymbolicEvaluationEngine(symbolic_engine, registry)
Evaluator(program, engine=engine, learning_logger=logger).run()
print(logger.to_list())
```

実行可能なウォークスルーが `notebooks/Learning_Log_Demo.ipynb` にあります。
`core.log_formatter.format_records` で人間可読な形式にも変換できます。

**推論の記録が構造化JSONとして取り出せる**——この設計が、後続プロジェクトで
「推論アーティファクト」という概念に発展していきます。

---

## 実行モード

| モード | 挙動 |
|---|---|
| `symbolic` | SymPy と知識レジストリでステップを検証する |
| `polynomial` | 式を展開してから比較する |
| `causal` | 因果推論エンジンによる誤り原因の解析を含む |

---

## 学習支援ツールから、系譜の起点へ

MathLang は**数学学習支援プロダクトとして作られました。**
研究のために作った実験言語ではありません。

しかし、学習支援を成立させるために解いた問題が、そのまま後続すべての土台になりました。

| 学習支援のために必要だったこと | 後の発展形 |
|---|---|
| 変形の前後と根拠（`before` / `after` / `note`）を書かせる | ReasonScript の **Reason IR** |
| 同じ手順からは同じ結果が再生されること | ReasonScript の **決定論的 ExecutionPlan** |
| 学習ログを機械可読に残す（`LearningLogger`） | ReasonScript の **ReasoningModel / ReasoningEvaluationReport** |
| 「もし前提が違ったら」を扱う（`counterfactual`） | VisionWorldModel の**状態遷移検証** |
| 同値だが形が違う解答を区別して判定する | MRA の **Evidence 検証**、VisionWorldModel の **4値判定** |
| 誤りの原因を推定し、複数の修正候補を返す | Design_BrainModel の **Repair ループ** |

**「学習者が納得できるように、過程を検査可能にする」**という要求と、
**「AIの推論を、人間が検証できるようにする」**という後年の要求は、
技術的にはほぼ同じ問題でした。

学習者に対して「なぜその変形が正しいのか」を示せないツールは使えません。
同じことが、AIの推論についても言えます。
MathLang は、その最初の答えにあたります。

→ [設計思想の詳細](../design-philosophy.md) · [開発年表](../timeline.md)
