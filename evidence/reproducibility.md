# Reproducibility — 再現手順

各プロジェクトの最小実行手順。詳細は各プロジェクトページの「Reproduction」セクションを参照。

| プロジェクト | 前提環境 | 最小実行コマンド | 想定所要時間 |
|---|---|---|---|
| [ReasonScript](../projects/reasonscript.md) | Python 3.12+(3.14.0で動作確認済み) | `pip install -e .` → `./reason ci --json` | CI フル実行で数分程度 |
| [VisionWorldModel](../projects/vision_world_model.md) | ReasonScript `>=0.5.1` インストール済み | `reason ci --json` | 数分程度 |
| [LanguageModel](../projects/language_model.md) | ReasonScript v0.5.4.5(指定コミット) | `reason project-validate . --json` | 数秒〜数分 |
| [Design_BrainModel](../projects/design_brainmodel.md) | Rust/Cargo、macOS(Apple Silicon推奨) | `cargo build -p design_cli --bin dbm` → `cargo test -p design_cli` | 数分程度(規模非公開) |
| [COHERENT](../projects/coherent.md) | Python 3.12+、uv | `uv sync` → `python3 -m pytest report/ -k word_gen` | 単語想起テストは数秒〜数十秒 |
| [mathlang](../projects/mathlang.md) | Python 3.12 | `python main.py --file edu/examples/pythagorean.mlang` | 即時 |

## 再現できないものの明示

すべての成果が再現可能なわけではない。**再現できない項目を隠さず明示する**方針を取る。

| 項目 | 状態 | 理由 |
|---|---|---|
| COHERENT のカタカナ・漢字生成(100% / 60〜80%) | ❌ 再現不可 | レポートが参照する生成スクリプト(`runner_katakana.py`、`runner_kanji.py`)が現在のリポジトリに未収録 |
| COHERENT の計算削減効果(θ掃引) | ⚠️ 設計確認のみ再現可能、性能としては未測定 | サンプル数5件、共鳴スコアがテストコード内の定数のため、実ワークロードでの効果は測定していない |

外部成果物(学習済みモデルの重み、大規模データセットなど)が必要な検証は、現時点でどのプロジェクトにも存在しない。

## CI・検証コマンドの実行確認履歴

| コマンド | 実行日 | 結果 |
|---|---|---|
| `./reason ci --json`(ReasonScript) | 2026-08-12 | 全ステージ PASS・1,116件のテスト通過(commit `0efb2ab`, Python 3.14.0) |

→ [検証マトリクス](verification_matrix.md) · [テスト戦略](test_strategy.md)
