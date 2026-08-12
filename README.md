# Portfolio — 検証可能な推論システムの研究開発

*[English version →](README.en.md)*

**「AIの出力を信じる」から「AIの推論を検証する」へ。**

このポートフォリオは、大規模言語モデルが抱える非決定性・検証不可能性・設計意図の喪失という課題に対して、
**決定論的な言語処理系と、記憶と真実を分離した推論アーキテクチャ**で応えようとする一連の研究開発をまとめたものです。

中心には自作のプログラミング言語 **ReasonScript** があり、その上に応用研究プロジェクトが積み上がっています。

---

## 系譜 / Project Lineage

探索期に得た知見が基盤（ReasonScript）に結実し、その上で
**MRA — Molecular Reasoning Architecture** という推論アーキテクチャの構築に向かっています。

```
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
| **[ReasonScript](https://github.com/chigenori053/ReasonScript)** | **推論を記述するための状態遷移記述言語。**決定論的実行とロールバック安全性を言語仕様で保証する [→ 詳細](docs/projects/reasonscript.md) | **Hybrid DSL** — 実行系: Python / ランタイム: Rust | 約133,600行 · テスト1,085件 | Apache-2.0 |

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
| **[mathlang](https://github.com/chigenori053/mathlang)** | **数学学習支援言語**（Python ベースの DSL）。人間が書く数式を Parser で正規化し、SymPy をランタイムに組み込んで**数式の正誤判定**を行う [→ 詳細](docs/projects/mathlang.md) | Python | 約9,200行 | 更新停止（Apache-2.0） |

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
  [Design_BrainModel](docs/projects/design-brainmodel.md) ·
  [COHERENT](docs/projects/coherent.md) ·
  [mathlang](docs/projects/mathlang.md) ·
  [VisionWorldModel](docs/projects/visionworldmodel.md) ·
  [LanguageModel](docs/projects/languagemodel.md)

---

## 現在地と今後 / Status

| | 状態 |
|---|---|
| **ReasonScript** | v0.5.4.5 リリース済み。CI 1,085件パス。ReasonGraph/World ビューア、パッケージレジストリ、SDK公開APIマニフェストが未実装 |
| **MRA** | 開発中。Molecule / Evidence / Provenance のデータモデルと Truth Boundary を仕様として確立した段階 |
| **VisionWorldModel** | Phase 3C-1 まで検証完了。適応的構造推論に着手 |
| **LanguageModel** | Phase 0（基盤固定・仕様策定）完了。Holographic Core 実装がこれから |
| **Design_BrainModel** | **v1 は未完成プロダクト** — ①推論爆発によるシステムフリーズ（現状は強引な抑制のみで、安定稼働の根拠がない）②記憶機構が期待した学習能力を発揮しなかった ③HolographicMemory の容量が実用に耐えない。この3つが **ReasonScript 開発の直接の動機**。**ReasonScript + MRA Base による v2 再設計を予定** |
| **COHERENT** | 言語生成は部分的に成功（文字・単語・多言語で実証、漢字は属性定義の重複により60〜80%）。数式の正誤判定は成功。計算資源の削減（記憶想起による推論スキップ）は **80%** を記録 |
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
