# 技術経歴書 / Technical Profile

> **注記：** 本書は公開リポジトリ6件の内容から機械的に構成した**技術的成果の記録**です。
> 所属・在籍期間・担当業務などの職歴情報は含まれていません。
> 職務経歴書として提出する場合は、末尾の「[記入が必要な項目](#記入が必要な項目)」を埋めてください。

| | |
|---|---|
| **GitHub** | https://github.com/chigenori053 |
| **専門領域** | 言語処理系設計 · 推論システムアーキテクチャ · AI検証基盤 |
| **主要言語** | Python · Rust |
| **活動期間（本書の対象）** | 2025年11月 〜 2026年8月 |

---

## 要約

**LLMの非決定性・検証不可能性・設計意図の喪失**という課題に対し、
決定論的な言語処理系と、記憶と真実を分離した推論アーキテクチャを設計・実装しています。

10ヶ月間で6つのプロジェクトを構築し、その中核として**プログラミング言語 ReasonScript を
ゼロから開発**（実装約135,000行、テスト1,085件、仕様書600本、Apache-2.0）。
Rust は2プロジェクト合計で約97,000行、うち Design_BrainModel は60を超える crate による
ワークスペースとして設計・実装しています。

現在は、その上に **MRA（Molecular Reasoning Architecture）** ——知識を型付き Atom と Bond からなる
Molecule として表現する推論アーキテクチャ——を構築中です。
**視覚・言語・ソフトウェア設計という3つのドメインに、同一の表現形式と推論契約で展開する**方針で、
それぞれ VisionWorldModel · LanguageModel · Design_BrainModel v2 として進めています。

特徴は、**再現性・検証可能性をライブラリではなく仕様レベルで保証する**という一貫した方針です。

---

## 技術スキル

### 言語処理系・コンパイラ

| スキル | 根拠となる実装 |
|---|---|
| 字句解析・構文解析 | ReasonScript パーサ、mathlang `core/parser.py` |
| AST 設計 | Surface AST と Semantic AST の二層分離、`ast_nodes.py` |
| 中間表現（IR）設計 | Reason IR（JSON Schema による機械検証付き） |
| 実行計画生成 | ExecutionPlan の決定論的生成 |
| 型システム | `ReasonScript_Language_Surface_Type_Specification_v0.1` |
| 名前空間・スコープ解決 | `ReasonScript_Language_Surface_Namespace_Import_Resolution_v0.1` |
| 操作的意味論 | `ReasonScript_Operational_Semantics_v0.1` |
| ABI 設計 | `ReasonScript_ABI_Specification_v0.1` |
| LSP サーバ実装 | ReasonScript LSP Phase 1、Design_BrainModel `apps/lsp` |

### Rust

| スキル | 根拠となる実装 |
|---|---|
| 大規模ワークスペース設計 | Design_BrainModel の **60+ crate** 構成 |
| Safe-Rust ランタイム | ReasonScript Visualization Runtime |
| Cargo によるマルチクレート管理 | `cargo build -p design_cli`、crate 間の責務分離 |
| API バージョニング・非推奨化 | `since` / `note` 属性による V1 → V2 移行設計 |
| 決定論的ハッシュ実装 | FNV-1a 64bit、正規化規則の実装 |
| CLI / REPL 実装 | `dbm` CLI、対話REPL |

### Python

| スキル | 根拠となる実装 |
|---|---|
| 処理系・ランタイム実装 | ReasonScript ツールチェーン（543ファイル） |
| 記号計算 | SymPy 統合（mathlang `symbolic_engine.py`、COHERENT `SymbolicEngine`） |
| 数値計算 | 複素テンソル演算、Conv2d/MaxPool2d/AvgPool2d、リバースモード自動微分 |
| テスト設計 | pytest / unittest、239テストファイル、Golden コーパス |
| パッケージ管理 | uv、pyproject.toml、Python 3.12 |
| 可視化UI | Streamlit（COHERENT Cognitive Simulator）、Jupyter |

### クロス言語・システム設計

| スキル | 根拠となる実装 |
|---|---|
| 多言語DTO契約設計 | **Rust / Python / TypeScript / Go / Java** が単一の規範契約を共有 |
| スキーマ設計 | `reason_ir.schema.json`、`semantic_frame.schema.json`、`molecule.schema.json` |
| バイナリファイル形式設計 | `.rstensor`（チェックサム検証・capability チェック付き） |
| 分散実行 | Dynamic ReasonUnit クラスタ実行（計画・実行・シミュレーション・検証・比較） |
| バージョニング戦略 | 構造世代の明示的アドレッシング（`water@generation-1`） |

### AI・推論システム

| スキル | 根拠となる実装 |
|---|---|
| 分散表現（HRR / VSA） | LanguageModel の Binding / Superposition / Cleanup 設計 |
| 連想記憶アーキテクチャ | COHERENT Optical Holographic Memory、Resonance Recall |
| マルチモーダル統合 | テキスト・画像・音声の複素数テンソルへの統一符号化 |
| 記号推論 | 知識レジストリ、ルール適用、証明エンジン |
| 因果推論 | mathlang / COHERENT の `causal/` モジュール、誤り原因推定 |
| ファジィ判定 | `fuzzy/` モジュール、Fuzzy Judge / DecisionEngine |
| エージェント設計 | Action-Based Reasoning、Planner → Executor → Validation → Repair ループ |
| 世界モデル | VisionWorldModel、Design_BrainModel `world_model` crate |
| 推論アーキテクチャ設計 | **MRA** — Molecule / Evidence / Provenance によるドメイン横断の知識表現と、視覚・言語・ソフトウェア設計への展開設計 |

### 品質保証・開発プロセス

| スキル | 根拠となる実装 |
|---|---|
| CI パイプライン設計 | `./reason ci` — 8段階の検証パイプライン、1,085テスト |
| 適合性検証フレームワーク | `conformance/run_conformance.py`、認証レポート自動更新 |
| 決定論ゲート | ハッシュ・テンプレート選択・出力テキストの完全一致検証 |
| Golden コーパステスト | ReasonScript `golden/` |
| 仕様駆動開発 | MUST/SHOULD 規範語による仕様書、実装より先に仕様を確定 |
| Phase 駆動の段階的検証 | 各Phaseが検証コマンド・テスト・機械可読成果物を伴う |

### 開発ツール構築

CLI · REPL · IDE · VS Code 拡張 · LSP · ブラウザ Playground · ターミナルUI（curses）·
Streamlit ダッシュボード · HTML レポート生成

---

## 主要成果

### 1. ReasonScript — 推論優先プログラミング言語

**期間：** 2026年4月 〜 現在（v0.5.4.5）
**役割：** 設計・実装のすべて
**規模：** 実装約135,000行 / テスト1,085件 / 仕様書600本

決定論的実行とロールバック安全性を**言語仕様として保証する**処理系を、ゼロから設計・実装。

**技術的な達成：**

- 4段階の中間表現（Surface AST → Semantic AST → Reason IR → ExecutionPlan）による
  決定論的コンパイルパイプラインを設計
- **5言語（Rust / Python / TypeScript / Go / Java）が単一の規範DTO契約を共有**する
  クロス言語バインディングを構築
- 目的別に7つのランタイムを実装（標準・ハイブリッド・ネイティブ・クラスタ・視覚・可視化・複素数）
- Tensor Training Foundation v0.2 として、NCHW Conv2d/MaxPool2d/AvgPool2d と
  **リバースモード自動微分**を実装
- `reason view` として、ソースコードと4段階の中間表現を**対応付けて表示する**
  ターミナルUIを構築
- 8段階のCIパイプラインと適合性検証フレームワークを整備し、1,085件のテストを維持

→ [詳細](projects/reasonscript.md)

### 2. Design_BrainModel — AIエージェント向け安全制御層

**期間：** 2026年1月 〜（v1 実装完了 / **v2 再設計予定**）
**役割：** 設計・実装のすべて
**規模：** Rust 約49,900行 / 60+ crate

AIコーディングエージェントに、設計意図の保持と実行安全性を与える推論制御レイヤー。
推論・記憶・アーキテクチャ評価の機構をすべて自前で構築した独立実装であり、
**共通基盤（ReasonScript / MRA）が満たすべき要件を洗い出した先行実装**にあたります。
v2 では、それらを基盤に委ねたうえで、MRA のソフトウェア設計ドメインモデルとして再設計します。

**技術的な達成：**

- 60を超える crate による Rust ワークスペースを、責務ごとに分離して設計
- **決定論の実現手段を仕様で凍結** — ハッシュアルゴリズム、シード、浮動小数点の整形精度、
  空文字列と `None` の区別、テンプレート選択の曖昧性閾値まで規定
- Planner → Executor → Validation → Repair Loop → Convergence Control という
  自律実行ループを実装
- コマンド分類による実行安全制御を実装
- **Agent Operational Charter** として、複数AIエージェントの権限階層と役割分担を仕様化
- Phase 6（大規模）· Phase 7（実リポジトリ）· Phase 8（人間評価）まで検証を実施

→ [詳細](projects/design-brainmodel.md)

### 3. COHERENT — 光学干渉メモリによる推論システム

**期間：** 2025年11月 〜 現在
**役割：** 設計・実装のすべて
**規模：** Python 約43,000行 / 327ファイル

記憶を波の干渉としてモデル化し、System 1（想起）と System 2（推論）を統合した Reasoning LM。

**技術的な達成：**

- テキスト・画像・音声を複素数テンソルに符号化し、**単一空間で扱うマルチモーダル記憶**を実装
- 干渉強度（Resonance）による連想想起機構を実装
- 思考を離散的な Action として出力し、実行結果を観察して仮説を修正する
  自己修正エージェントを実装
- 成功エピソードを記憶にフィードバックし「直感」として定着させる学習ループを設計
- 記憶の干渉と推論過程をリアルタイム可視化する Streamlit UI を構築
- 記号計算・微積分・線形代数・統計・幾何・三角関数の各エンジンを統合

→ [詳細](projects/coherent.md)

### 4. VisionWorldModel — MRA 視覚ドメインモデル

**期間：** 2026年7月 〜 現在
**役割：** 設計・実装のすべて
**規模：** ReasonScript + Python 約7,700行

MRA の視覚ドメインモデルであり、ReasonScript を実用した最初の本格的実装。
Molecular 表現の妥当性を、正解が曖昧さなく決まる実在の分子構造を題材に検証しています。

**技術的な達成：**

- **ReasonScript でドメインモデル全体を記述**し、自作言語の実用性を検証
- 観測された VisualAtom と推論された World 構成要素を、永続化レベルで分離
- `ACCEPT` / `REVISE` / `DEFER` / `ABSTAIN` の4値判定を実装し、
  **判断の保留・棄権を正常系として設計**
- 「不変な候補の構築 → グラフ検証 → 明示的な状態移行 → アトミックなコミット」という
  構造変更モデルを確立
- 構造世代の明示的アドレッシング（`water@generation-1`）を実装
- Phase 1 〜 3C-1 の各段階で、検証コマンド・テスト・機械可読成果物を整備

→ [詳細](projects/visionworldmodel.md)

### 5. LanguageModel — MRA 言語ドメインモデル

**期間：** 2026年8月 〜 現在（Phase 0 完了）
**役割：** 仕様策定

MRA の言語ドメインモデル。その仕様書は、MRA 全体で共有される中核概念
（Molecule · Evidence · Provenance · Truth Boundary）を最初に定式化した文書でもあります。

**技術的な達成：**

- **Truth Boundary** を定式化 — 連想記憶を「候補を出す層」に限定し、
  事実確定を正規データと Evidence 検証に委ねる原則を仕様化
- 言語能力・知識・推論を、Neural Model / HolographicMemory / Molecular Memory /
  ReasonScript Runtime の4者に分割する責務設計
- HRR / VSA に基づく Binding / Superposition / Cleanup の設計
- Concept / Relation / Event / Experience の4種 Molecule と、
  Evidence · Provenance によるデータモデルを設計
- `foundation.lock.json` により基盤をコミットハッシュ単位で固定し、再現性の範囲を明示

→ [詳細](projects/languagemodel.md)

### 6. mathlang — 数学的思考過程のDSL

**期間：** 2025年11月
**役割：** 設計・実装のすべて
**規模：** Python 約9,200行

**技術的な達成：**

- 変形の前後と根拠（`before` / `after` / `note`）を**構文として強制する**DSLを設計
- 反実仮想（counterfactual）を言語機能として実装
- 推論ステップを構造化JSONとして記録する LearningLogger を実装
- 誤りの原因を推定する因果解析エンジンを実装
- Edu / Pro / Demo の3層CLIと、JSON設定によるシナリオ機構を構築

→ [詳細](projects/mathlang.md)

---

## 特徴的な設計判断

技術面接などで説明できる、意図的な設計判断を挙げます。

### 決定論を「努力目標」にしない

Design_BrainModel では、再現性を担保するために
**ハッシュアルゴリズム・シード・浮動小数点の文字列化精度（`{:.6}`）・
リストのソート・`Option` の符号化・空文字列と `None` の区別**まで仕様に明記しました。

実装者の裁量が入る余地を潰すことで、「たまたま今は一致している」状態を排除しています。

### 「わからない」を出力にする

VisionWorldModel の `DEFER` / `ABSTAIN`、Phase 3A の `UNDETERMINED` は、
すべて**判断を確定しないことを正常な結果として返す**設計です。

根拠が不十分なまま答えを出すことのほうがリスクである、という判断に基づいています。

### 自作言語を、自分で都合よく変えない

LanguageModel は ReasonScript をコミットハッシュで固定し、
「基盤を変更せず、公開CLIと決定論的契約のみを利用する」と明記しています。

自作基盤は改変が容易なぶん、応用側の都合で歪みやすい。それを規律で防いでいます。

### テストを通すための近道を、構造的に禁じる

VisionWorldModel Phase 3A では、判断を状態トレースからの evidence 抽出のみで導出し、
**フィクスチャ名や入力IDによる参照を一切使わない**と明記しています。

検証系を自作する際に最も起きやすい自己欺瞞を、設計で防いでいます。

### AIエージェントを、能力ではなく権限で分離する

Design_BrainModel の Agent Operational Charter では、
ResearchAgent（仕様案を出すが実装しない）と CodingAgent（実装するがアーキテクチャを再定義しない）を
分離しました。

提案する者と実装する者を分けることで、AIが自分の提案を自分で正当化する経路を断っています。

---

## 数値サマリ

| 指標 | 値 |
|---|---|
| プロジェクト数 | 6 |
| 実装総行数（概算・依存関係除く） | **約253,000行** |
| うち Python | 約150,000行（1,446ファイル） |
| うち Rust | 約97,000行（760ファイル） |
| うち TypeScript ほか | 約6,000行 |
| 設計・仕様ドキュメント | **約780本** |
| ReasonScript CI テスト | 1,085件パス |
| Rust crate 数（Design_BrainModel） | 60+ |
| 対応言語バインディング | 5言語（Rust / Python / TypeScript / Go / Java） |
| 開発期間 | 約10ヶ月（2025-11 〜 2026-08） |

---

## 記入が必要な項目

本書を職務経歴書として使う場合、以下を追記してください。

- [ ] 氏名・連絡先
- [ ] 学歴
- [ ] 職歴（会社名 / 在籍期間 / 役職 / 担当業務）
- [ ] 業務での開発経験（本書はすべて個人プロジェクトが対象）
- [ ] チーム開発の経験・規模
- [ ] 保有資格
- [ ] 語学力
- [ ] 希望条件（職種 / 勤務地 / 稼働形態）

---

→ [ポートフォリオ トップ](../README.md) · [設計思想](design-philosophy.md) · [開発年表](timeline.md)
