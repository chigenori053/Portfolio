# Portfolio — 検証可能な推論システムの研究開発

*[English version →](README.en.md)*

**「AIの出力を信じる」から「AIの推論を検証する」へ。**

**AIの推論を検証可能にするための言語処理系と推論アーキテクチャを、個人で設計・実装しています。**
以下6リポジトリはすべて個人プロジェクトであり、設計・実装・検証を単独で担当しています
（AIコーディングエージェントを併用）。

- **GitHub** — [@chigenori053](https://github.com/chigenori053)
- **専門** — 言語処理系設計 · 推論システムアーキテクチャ · AI検証基盤
- **主言語** — Python · Rust

## 2分で見る場合

| | |
|---|---|
| **最初に見るなら** | **[ReasonScript](https://github.com/chigenori053/ReasonScript)** — 推論を記述する状態遷移記述言語。Python 実行系 + Rust ランタイムの Hybrid DSL（Apache-2.0） |
| **検証済みの成果** | `./reason ci` を実行し**全ステージ PASS / 1,116テスト通過**を確認（2026-08-12, commit `0efb2ab`）<br>COHERENT で**日英60語の想起 100%・言語混在による劣化率 0.00%**（実測共鳴値つきCSVあり）<br>ReasonScript を**実際に使って**別ドメイン（VisionWorldModel）のモデルを `.rsn` で全面記述 |
| **正直に言うと** | Design_BrainModel v1 は**未完成プロダクト**（推論爆発を抑え込んではいるが安定の根拠がない）。COHERENT の計算削減効果は**未測定**。詳細は各ページに証拠の強さつきで記載しています |
| **もっと読むなら** | [設計思想](docs/design-philosophy.md) · [開発年表](docs/timeline.md) · [技術経歴書](docs/technical-profile.md) |

<sub>本ポートフォリオでは、**主張ごとに証拠の強さ（実測 / 実行結果 / 設計確認のみ / 未測定 / 再現不可）を明示**しています。裏付けの弱い数字を強い成果として提示しないことを方針としています。</sub>

---

このポートフォリオは、大規模言語モデルが抱える非決定性・検証不可能性・設計意図の喪失という課題に対して、
**決定論的な言語処理系と、記憶と真実を分離した推論アーキテクチャ**で応えようとする一連の研究開発をまとめたものです。

中心には自作のプログラミング言語 **ReasonScript** があり、その上に応用研究プロジェクトが積み上がっています。

---

## 系譜 / Project Lineage

リサーチ期に固めたコンセプトが探索期の実装に結実し、そこで得た知見が基盤（ReasonScript）に
結実して、その上で **MRA — Molecular Reasoning Architecture** という推論アーキテクチャの
構築に向かっています。

```
  ■ リサーチ期 ────────────────────────────────────────────────────

   2025-01   運営するプログラミング教室で、数学学習コースの新設を検討
                 │                 学習コーチとして LLM を活用するアイディアに着想
                 │                 → 教育の専門家に相談。「難しい」とフィードバック
                 │                   （当時の LLM は数学計算に難があり、出力も不安定）
                 │
                 │  SymbolicAI の存在を知る
                 │  ▼
                 │  Wolfram Alpha（教育分野の既存サービス）の存在を知り、実機で検証
                 │  → 入力した数式に対し、途中計算を含む段階的な評価ができないと判明
                 ▼
             MathLang のコンセプトへ到達
             「途中計算式を含む数式を理解・評価できる、専用の教育言語」

  ■ 探索期 ────────────────────────────────────────────────────────

   2025-11   mathlang              数学学習支援言語（Python ベースの DSL）
                 │                 数式の正誤判定 ──「過程を第一級のデータにする」
                 ▼
   2025-11   COHERENT              理論検証プロジェクト（推論モデル名: BrainModel）
                 │                 「Transformer 以外で LLM 同等の推論は可能か」
                 │                 HolographicMemory + MemorySpace ── MRA 記憶モデルの前身
                 ▼
   2026-01   Design_BrainModel v1  コードを想起するコーディングエージェント
                 │                 Rust / 60+ crates ── 未完成プロダクト
                 │
                 │  推論爆発・記憶機構の限界に突き当たり、
                 │  その解決のため基盤から作り直す判断へ
                 ▼
  ■ 基盤 ──────────────────────────────────────────────────────────

   2026-04   ★ ReasonScript        推論を記述する状態遷移記述言語（Apache-2.0）
                 │                 Hybrid DSL: Python 実行系 + Rust ランタイム
                 │                 Surface AST → … → ExecutionPlan
                 │
  ■ MRA ───────┴──────────────────────────────────────────────────

            ★ MRA — Molecular Reasoning Architecture   ← 現在開発中
              知識を型付き Atom / Bond からなる Molecule として表現する推論モデル
                 │
     ┌───────────┼────────────────────────┐
     ▼           ▼                        ▼
 VisionWorldModel   LanguageModel      Design_BrainModel v2
  視覚ドメイン       言語ドメイン         ソフトウェア設計ドメイン
  2026-07           2026-08              （再設計予定）

  観測と推論の分離   Truth Boundary        設計案 → システム構造
  ACCEPT/REVISE/    連想記憶と正規知識     → コードの想起
  DEFER/ABSTAIN     の分離
```

### リサーチ期について — なぜ GitHub に残っていないか

2025年1月からの数ヶ月は、コードよりも**要件の検討と専門家への相談**が中心の期間でした。
運営するプログラミング教室で数学学習コースの新設を検討する中で「学習コーチとしてLLMを
活用できないか」という着想を得ましたが、教育の専門家に相談した結果、当時のLLMの数学計算の
不安定さを理由に「難しい」というフィードバックを受けています。
その後 SymbolicAI を知り、続けて教育分野の既存サービスとして Wolfram Alpha を知って
実際に検証したところ、**入力した数式に対して途中計算を含む段階的な評価ができない**という
限界を確認しました。ここから「途中計算式を含む数式を理解・評価できる専用の教育言語」という
MathLang のコンセプトに至っています。
この期間はリポジトリとして残せる実装物がなく、コンセプトを固めるための検討・相談の記録が
中心だったため、GitHub 上には痕跡がありません。**探索期の起点である mathlang は、この
リサーチ期の結論をそのまま実装に落としたもの**です。

### この構造が意味すること

**一つの基盤の上に、視覚・言語・ソフトウェア設計という異なる3ドメインの応用が乗ります。**
応用が1つしかない基盤は「その応用のために作ったもの」にしか見えませんが、
3ドメインに展開されるなら、基盤設計が実際に汎用だったことの検証になります。

各ドメインモデルは基盤を改変せず、ReasonScript の公開CLIと決定論的契約のみを利用します。
`LanguageModel` は基盤をコミットハッシュ単位（`7f29c1c`）で固定しています。

---

## プロジェクト一覧 / Projects

### 基盤 — Foundation

| プロジェクト | 概要 | 主言語 | 規模 | ライセンス |
|---|---|---|---|---|
| **[ReasonScript](https://github.com/chigenori053/ReasonScript)** | **推論を記述するための状態遷移記述言語。**決定論的実行とロールバック安全性を言語仕様で保証する [→ 詳細](docs/projects/reasonscript.md) | **Hybrid DSL** — 実行系: Python / ランタイム: Rust | 約133,600行 · **CI 1,116件パス（実行確認済み）** | Apache-2.0 |

#### 外部検証 — External Verification

ReasonScript 単体の CI とは別に、外部プロジェクトで基盤仕様（決定論・観測可能性・独立検証可能性）が
現実的なワークロードでも成立するかを検証しています。

| プロジェクト | 検証内容 | 結果 | 状態 |
|---|---|---|---|
| **[Transformer_Test](https://github.com/chigenori053/Transformer_Test)**（RS-DT-JP-GREET-001） | `.rsn` のみで実装した Transformer による日本語挨拶8クラス分類。決定論・観測非干渉・独立検証（Rust）を検証 [→ 詳細](docs/projects/transformer-test.md) | **決定論成立**（チェックポイント SHA-256 完全一致）・**観測が計算に非干渉**なことを実測確認。基盤の型検査バグ3件を発見・修正。Model A〜D の構造的優位性そのものは**未検証** | Phase 0〜6 実装済み / Phase 7〜9（本実験）は性能制約により一時中断 |

### MRA ドメインモデル — Molecular Reasoning Architecture

開発中の推論アーキテクチャ MRA を構成する、ドメイン別のモデル群です。

| プロジェクト | ドメイン | 概要 | 規模 | 状態 |
|---|---|---|---|---|
| **[VisionWorldModel](https://github.com/chigenori053/VisonWorldModel)** | 視覚 | 観測と推論を分離し、判断を保留できる世界モデル [→ 詳細](docs/projects/visionworldmodel.md) | 約7,700行 | Phase 3C-1 |
| **[LanguageModel](https://github.com/chigenori053/LanguageModel)** | 言語 | 連想記憶と正規知識を分離した言語モデル基盤 [→ 詳細](docs/projects/languagemodel.md) | 仕様書中心 | Phase 0 |
| **[Design_BrainModel](https://github.com/chigenori053/Design_BrainModel)** | ソフトウェア設計 | 設計案からシステム構造を生成し、そこからコードを**想起する**コーディングエージェント [→ 詳細](docs/projects/design-brainmodel.md) | 約57,000行 · 60+ crate | **v1 未完成 / v2 再設計予定** |

### 探索期 — Exploration

MRA と ReasonScript に至る過程で構築した、先行プロジェクトです。

| プロジェクト | 概要 | 主言語 | 規模 | 状態 |
|---|---|---|---|---|
| **[COHERENT](https://github.com/chigenori053/COHERENT)** | **理論検証プロジェクト**（推論モデル名 **BrainModel**）。Transformer に依らない推論の成立可能性を、光学干渉のシミュレーションと記憶再利用によって検証 [→ 詳細](docs/projects/coherent.md) | Python | 約43,000行 | 検証継続中 |
| **[mathlang](https://github.com/chigenori053/mathlang)** | **数学学習支援言語**（Python ベースの DSL）。人間が書く数式を Parser で正規化し、SymPy をランタイムに組み込んで**数式の正誤判定**を行う [→ 詳細](docs/projects/mathlang.md) | Python | 約9,200行 | 開発停止（Apache-2.0） |

---

## 一貫する設計思想 / Design Principles

全プロジェクトを貫く5つの原則があります。詳細は **[設計思想](docs/design-philosophy.md)** を参照してください。

### 1. 決定論と再現性を仕様で保証する

「同じ入力からは同じ実行計画が生成される」を、努力目標ではなく**言語仕様と検証ゲート**として実装しています。
Design_BrainModel ではハッシュアルゴリズム（FNV-1a 64bit）、シード値、浮動小数点の文字列化精度（`{:.6}`）、
テンプレート選択の曖昧性閾値（`1e-6`）まで仕様で凍結しています。

### 2. 「記憶」と「真実」を分離する — Truth Boundary

連想記憶（ホログラフィック記憶）は**意味活性化の場**であって、真実の保存先ではありません。

> ベクトル類似度・復号結果・ニューラルモデルの確信度だけを根拠として、関係を断定してはならない。
> — *MRA Holographic Semantic Language Model 仕様書 v0.1, §2.1 Truth Boundary*

「意味的に近い」と「その関係が成立する」は厳密に区別され、事実応答は必ず正規データと Evidence 検証を通過します。
この原則は COHERENT / BrainModel の想起→検証、VisionWorldModel の観測→推論の分離にも共通しています。
COHERENT では `Accept` / `Review` / `Reject` の三値判定として実装され、判定には必ず根拠ログが伴います。

### 3. 判断しないことを、正当な出力にする

VisionWorldModel は `ACCEPT` / `REVISE` / `DEFER` / `ABSTAIN` の4値で判断を返します。
根拠が不十分なら**保留・棄権する**ことを設計上の正常系として扱う。これは無理に答えを出すLLMへの対案です。

### 4. 仕様書を先に書き、Phase で刻む

MUST / MUST NOT / SHOULD / MAY の規範語による仕様書を実装より先に置き、
Phase 0 → 1 → 2 → 3A → 3B-1 → 3B-2 → 3B-3 → 3C-1 と細かく刻んで進めます。
各Phaseは検証コマンドと機械可読な成果物（JSON）を必ず伴います。

### 5. AIエージェントとの協働そのものを設計対象にする

`AGENT.md` / `Rule.md` / `AGENTS.md` により、権限階層と役割分担を明文化しています。

```
権限順位:  specs/  >  Rule.md  >  TASK_STATE.yaml  >  実装コード
役割分担:  Architect(人間) / ResearchAgent / CodingAgent / ValidationAgent
```

> *"DBM doesn't compete with Claude Code — it provides the structure and safety that Claude Code tends to lack."*
> — Design_BrainModel README

---

## 技術スタック / Tech Stack

| 領域 | 技術 |
|---|---|
| **言語処理系** | 字句・構文解析、AST設計、中間表現(IR)、実行計画生成、型仕様、名前空間解決、**状態遷移意味論の設計** |
| **Rust** | 言語ランタイム実装、60+ crate のワークスペース設計、Safe-Rust、Cargo、LSP サーバ |
| **Python** | 言語実行系・ツールチェーン実装、SymPy による記号計算、pytest、uv |
| **クロス言語** | Rust / Python / TypeScript / Go / Java の共通DTO契約 |
| **数値計算** | 複素テンソル、Conv2d/MaxPool2d/AvgPool2d、リバースモード自動微分 |
| **AI・推論** | HRR / VSA（分散表現）、記号推論、因果推論、ファジィ判定、マルチモーダル統合 |
| **ツールチェーン** | CLI、REPL、IDE、VS Code拡張、LSP、ブラウザPlayground、CI パイプライン |
| **品質保証** | Conformance フレームワーク、Golden コーパス、決定論ゲート、スキーマ検証 |

---

## ドキュメント / Documentation

- **[設計思想](docs/design-philosophy.md)** — 5つの原則の詳細と、それが各プロジェクトでどう実装されているか
- **[開発年表](docs/timeline.md)** — 2025-11 から現在までの流れと、各段階での問題意識の変化
- **[技術経歴書](docs/technical-profile.md)** — スキルセットと成果を職務経歴書形式で整理
- **プロジェクト詳細** — [ReasonScript](docs/projects/reasonscript.md) ·
  [Transformer_Test（外部検証）](docs/projects/transformer-test.md) ·
  [Design_BrainModel](docs/projects/design-brainmodel.md) ·
  [COHERENT](docs/projects/coherent.md) ·
  [mathlang](docs/projects/mathlang.md) ·
  [VisionWorldModel](docs/projects/visionworldmodel.md) ·
  [LanguageModel](docs/projects/languagemodel.md)

---

## 現在地と今後 / Status

| | 状態 |
|---|---|
| **ReasonScript** | v0.5.4.5 リリース済み。**`./reason ci` を実行し、全ステージ PASS / 1,116件のテスト通過を確認**（2026-08-12、commit `0efb2ab`、Python 3.14.0）。ReasonGraph/World ビューア、パッケージレジストリ、SDK公開APIマニフェストが未実装 |
| **Transformer_Test（外部検証）** | RS-DT-JP-GREET-001。決定論（チェックポイント SHA-256 完全一致）・観測非干渉を実測確認、基盤の型検査バグ3件を発見・修正。Model A〜D の構造的優位性の検証（本実験 Phase 7〜9）は性能制約により一時中断 |
| **MRA** | 開発中。Molecule / Evidence / Provenance のデータモデルと Truth Boundary を仕様として確立した段階 |
| **VisionWorldModel** | Phase 3C-1 まで検証完了。適応的構造推論に着手 |
| **LanguageModel** | Phase 0（基盤固定・仕様策定）完了。Holographic Core 実装がこれから |
| **Design_BrainModel** | **v1 は未完成プロダクト** — ①推論爆発によるシステムフリーズ（現状は強引な抑制のみで、安定稼働の根拠がない）②記憶機構が期待した学習能力を発揮しなかった ③HolographicMemory の容量が実用に耐えない。この3つが **ReasonScript 開発の直接の動機**。**ReasonScript + MRA Base による v2 再設計を予定** |
| **COHERENT** | **単語・多言語の想起 100%（60語・実測データあり）**が最も確度の高い成果。数式の正誤判定も成立。文字生成はカタカナ100%・漢字60〜80%（生成スクリプト未収録のため再現不可）。「記憶があれば計算しない」機構は**5件の固定シナリオによる設計確認のみ**で、性能としては未測定 |
| **mathlang** | 2025-11 で更新停止。学習支援のために解いた課題（過程の記述・再生・同値判定）が後続すべての土台になった |

---

## ライセンス方針 / Licensing

**基盤ツールは開き、研究アーキテクチャ本体は留保する**という方針です。

| 対象 | 方針 |
|---|---|
| **ReasonScript** | **Apache-2.0**。MRA の実装手段であり、研究対象ではないため公開 |
| **mathlang** | **Apache-2.0**。MRA とは独立した初期実験のため公開 |
| **MRA ドメインモデル**（VisionWorldModel / LanguageModel / Design_BrainModel） | **全権利留保。** 開発中のアーキテクチャを構成するため、現時点ではライセンスを付与していません |
| **COHERENT** | **全権利留保。** 研究・検証プロジェクトのため |

留保しているリポジトリのコードも、**閲覧と評価のために公開しています。**
利用をご希望の場合は、各リポジトリの Issue でご相談ください。
