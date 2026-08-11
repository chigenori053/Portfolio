# MathLang

> **Mathematical Thinking Language** — 数学的思考の「過程」そのものを記述し、
> 学習者とAIの双方が検査・再生・改善できるようにするDSLとツール群。

| | |
|---|---|
| **リポジトリ** | https://github.com/chigenori053/mathlang |
| **開始** | 2025-11 |
| **主要言語** | Python 3.12（実装 約9,200行）· Jupyter Notebook |
| **規模** | `core/` に23モジュール · テスト30ファイル · ドキュメント29本 |
| **位置づけ** | 系譜の起点。「推論過程を第一級のデータにする」という発想の出発点 |

---

## コンセプト

答えではなく、**答えに至る過程**を書く言語です。

- **Process-first** — 最終解答よりも、途中の作業ステップを重視する
- **AI-assisted** — 人間の推論と SymbolicAI（SymPy）を組み合わせ、式の説明と変換を行う
- **Reproducible** — 同一入力からは常に同一の実行トレースが再生される
- **Lightweight** — Python 3.12 のモジュール構成。教育と研究の両方に使える

「なぜその変形をしたのか」が消えてしまう、という数学教育の課題に対する、言語設計からの回答です。

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

## この設計の背景にある考え方

MathLang は表面的には教育ツールですが、扱っている問題は
**「推論の過程を、検査・再生・検証できるデータとして表現するにはどうすればよいか」**です。

- `step` に `before` / `after` / `note` を強制する → 後の **Reason IR**
- 同一入力から同一トレース → 後の **決定論的 ExecutionPlan**
- `LearningLogger` の構造化JSON → 後の **ReasoningModel / ReasoningEvaluationReport**
- `counterfactual` による反実仮想 → 後の **世界モデル・状態遷移検証**

系譜の起点として、後続すべてに繋がるアイデアがここに揃っています。

→ [設計思想の詳細](../design-philosophy.md) · [開発年表](../timeline.md)
