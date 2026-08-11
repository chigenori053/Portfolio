# Portfolio — 検証可能な推論システムの研究開発

*[English version →](README.en.md)*

**「AIの出力を信じる」から「AIの推論を検証する」へ。**

このポートフォリオは、大規模言語モデルが抱える非決定性・検証不可能性・設計意図の喪失という課題に対して、
**決定論的な言語処理系と、記憶と真実を分離した推論アーキテクチャ**で応えようとする一連の研究開発をまとめたものです。

中心には自作のプログラミング言語 **ReasonScript** があり、その上に応用研究プロジェクトが積み上がっています。

---

## 系譜 / Project Lineage

```
                        ┌──────────────────────────────┐
   2025-11              │  mathlang                    │  推論過程を記述するDSL
   ────────────         │  MathLang                    │  教育 → 研究への入口
                        └──────────────┬───────────────┘
                                       │
                        ┌──────────────▼───────────────┐
   2025-11              │  COHERENT                    │  想起(System 1) × 推論(System 2)
   ────────────         │  Optical Holographic Memory  │  光学干渉メモリ
                        └──────────────┬───────────────┘
                                       │
                        ┌──────────────▼───────────────┐
   2026-01              │  Design_BrainModel           │  AIエージェントの安全・整合性制御層
   ────────────         │  Rust / 60+ crates           │  設計意図の永続化
                        └──────────────┬───────────────┘
                                       │
                        ┌──────────────▼───────────────┐
   2026-04              │  ★ ReasonScript              │  基盤言語・決定論的実行処理系
   ────────────         │  v0.5.4.5 / 6言語バインディング │  Surface AST → … → ExecutionPlan
                        └──────────────┬───────────────┘
                                       │
                    ┌──────────────────┴──────────────────┐
                    │                                     │
      ┌─────────────▼─────────────┐       ┌───────────────▼──────────────┐
      │  VisionWorldModel         │       │  LanguageModel               │
      │  2026-07                  │       │  2026-08                     │
      │  観測と推論の分離          │       │  MRA Holographic Semantic    │
      │  ACCEPT/REVISE/DEFER/     │       │  Memory / Truth Boundary     │
      │  ABSTAIN                  │       │                              │
      └───────────────────────────┘       └──────────────────────────────┘
```

下流2プロジェクトは基盤を改変せず、ReasonScript の公開CLIと決定論的契約のみを利用します。
`LanguageModel` は基盤をコミットハッシュ単位（`7f29c1c`）で固定しています。

---

## プロジェクト一覧 / Projects

| プロジェクト | 概要 | 主言語 | 規模 | 詳細 |
|---|---|---|---|---|
| **[ReasonScript](https://github.com/chigenori053/ReasonScript)** | 決定論的実行とロールバック安全性を言語仕様で保証する推論優先言語 | Python / Rust / TS / Go / Java | 約135,000行 · テスト1,085件 | [→ 詳細](docs/projects/reasonscript.md) |
| **[Design_BrainModel](https://github.com/chigenori053/Design_BrainModel)** | AIコーディングエージェントに設計意図と実行安全性を与える推論制御層 | Rust | 約57,000行 · 60+ crate | [→ 詳細](docs/projects/design-brainmodel.md) |
| **[COHERENT](https://github.com/chigenori053/COHERENT)** | 光学干渉メモリとアクション予測型推論を融合した Reasoning LM | Python | 約43,000行 | [→ 詳細](docs/projects/coherent.md) |
| **[mathlang](https://github.com/chigenori053/mathlang)** | 数学的思考の過程そのものを記述・再生・検証するDSL | Python | 約9,200行 | [→ 詳細](docs/projects/mathlang.md) |
| **[VisionWorldModel](https://github.com/chigenori053/VisonWorldModel)** | 観測と推論を分離し、判断を保留できる世界モデル検証系 | ReasonScript / Python | 約7,700行 | [→ 詳細](docs/projects/visionworldmodel.md) |
| **[LanguageModel](https://github.com/chigenori053/LanguageModel)** | 連想記憶と正規知識を分離した言語モデル基盤（Phase 0・仕様策定段階） | Python / ReasonScript | 仕様書中心 | [→ 詳細](docs/projects/languagemodel.md) |

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
この原則は COHERENT の想起→検証、VisionWorldModel の観測→推論の分離にも共通しています。

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
| **言語処理系** | 字句・構文解析、AST設計、中間表現(IR)、実行計画生成、型仕様、名前空間解決 |
| **Rust** | 60+ crate のワークスペース設計、Safe-Rust ランタイム、Cargo、LSP サーバ |
| **Python** | 処理系実装、SymPy による記号計算、pytest、uv |
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
| **Design_BrainModel** | 自律実行ループ・REPL・実行安全制御は実装済み。Git統合と DesignUnit 統合が進行中 |
| **COHERENT** | Phase 2（Coding Agent Integration）に向けて開発中 |
| **VisionWorldModel** | Phase 3C-1 まで検証完了。適応的構造推論に着手 |
| **LanguageModel** | Phase 0（基盤固定・仕様策定）完了。Holographic Core 実装がこれから |

---

<sub>各リポジトリのライセンスは個別に確認してください。ReasonScript はルート LICENSE が未確定です（`vscode-extension/` のみ MIT）。</sub>
