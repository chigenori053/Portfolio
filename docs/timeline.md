# 開発年表 — 問題意識の変遷

2025年11月から2026年8月までの約10ヶ月間の流れです。
各リポジトリはコミット履歴が単一のため、日付は GitHub のリポジトリ作成日・更新日、
および仕様書・CHANGELOG の記載日に基づいています。

---

## 全体の流れ

```
2025-11 ────┬─── mathlang            「推論過程を記述できるか？」
            │
            └─── COHERENT            「記憶と推論を分離できるか？」
                     │
2026-01 ─────────── Design_BrainModel 「AIの生成を、設計から逸脱させずに済むか？」
                     │
2026-04 ─────────── ★ ReasonScript    「そもそも言語処理系から作り直すべきでは？」
                     │
2026-07 ─────────── VisionWorldModel  「観測と推論を分離できるか？」
                     │
2026-08 ─────────── LanguageModel     「言語モデルの責務を分割できるか？」
```

問題意識が **「記述する」→「制御する」→「基盤から作る」→「基盤の上で応用する」** と
移り変わっているのが読み取れます。

---

## 2025年11月 — 起点：推論過程を記述する

### mathlang（2025-11-07）

**問い：数学的思考の「過程」を、検査・再生できるデータとして書けるか？**

答えとして作られたのが MathLang DSL でした。

```text
step:
    before: (3 + 5) * 4
    after: 8 * 4
    note: simplify addition
```

`before` / `after` / `note` を構文として強制する——つまり
**「何を、何に変え、なぜそうしたか」を書かないと、そもそもプログラムにならない**設計です。

さらに `LearningLogger` が推論の全ステップを構造化JSONとして出力し、
`counterfactual` 節で「前提を変えたらどうなるか」を表現できるようにしました。

ここで生まれた発想が、後続すべての土台になります。

| mathlang の要素 | 後の発展形 |
|---|---|
| `step` の before/after/note | ReasonScript の **Reason IR** |
| 同一入力 → 同一トレース | ReasonScript の **決定論的 ExecutionPlan** |
| `LearningLogger` の構造化JSON | ReasonScript の **ReasoningModel / EvaluationReport** |
| `counterfactual` 節 | VisionWorldModel の**状態遷移検証** |

### COHERENT（2025-11-23）

mathlang の直後、わずか2週間後に始まっています。

**問い：記憶と推論を分けたアーキテクチャは成立するか？**

ここで「Recall-First」——System 1（直感的想起）と System 2（論理的推論）の
ハイブリッド——という構造が導入されます。

そして記憶を、ベクトルDBではなく**波の干渉（Optical Holographic Memory）**として
モデル化しました。テキスト・画像・音声を複素数テンソルにエンコードし、
Resonance（干渉強度）で想起する、という設計です。

**Tracer** が思考の各ステップ（State → Action → Result）をエピソードとして記録し、
成功したエピソードが光学メモリにフィードバックされて「直感」として定着する——
という学習ループも設計されました。

> mathlang が「過程を書けるようにした」のに対し、
> COHERENT は「過程を**自律的に生成し、記憶に沈殿させる**」ところまで進めています。

---

## 2026年1月 — 転換：AIの生成そのものを制御する

### Design_BrainModel（2026-01-24）

**問い：コーディングエージェントが生成したコードが、設計から逸脱するのをどう防ぐか？**

ここで対象が**数学・推論そのものから、ソフトウェア開発の現場**に移ります。
そして使用言語も Python から **Rust** に変わります。

問題意識は明確です。

| 課題 | 対応 |
|---|---|
| エージェントは設計意図を保持しない | `design.md` → `DesignUnit` で永続化 |
| LLM出力は非決定的 | 構造化された実行・検証フロー |
| AI生成の変更がアーキテクチャを壊す | 設計・コード・実行を横断した推論 |
| 修正ループがアドホック | Repair を推論プロセスの一部に |
| 安全でない実行がプロジェクトを破壊 | コマンド分類による実行安全制御 |

この時期に、**決定論を仕様レベルで凍結する**という手法が確立されます。
FNV-1a 64bit、シード値、`{:.6}` の浮動小数点整形、`1e-6` の閾値——
ハッシュの取り方まで `DESIGN.md` に書き込まれました。

同時に **Agent Operational Charter**（`AGENT.md`）が作られ、
Architect（人間）/ ResearchAgent（Gemini）/ CodingAgent（Codex）/ ValidationAgent という
役割分担と、`specs/` > `Rule.md` > `TASK_STATE.yaml` > 実装コード という権限階層が定義されます。

60を超える crate による Rust ワークスペースという規模も、この時期に構築されています。
Phase 6（大規模）· Phase 7（実リポジトリ）· Phase 8（人間評価）まで検証が進みました。

> **この段階での気づきが、次の転換を生みます。**
> 決定論的な推論を実現しようとすると、既存の言語とランタイムでは足りない。

---

## 2026年4月 — 基盤：言語処理系から作り直す

### ReasonScript（2026-04-12）

**問い：決定論的で検証可能な推論を書くための言語は、どうあるべきか？**

ここで**言語処理系そのものを作る**という選択がなされます。ポートフォリオ最大の投資です。

```
.rsn → Surface AST → Semantic AST → Reason IR → ExecutionPlan → InferenceResult
```

4段階の中間表現を経て、同一入力から同一の実行計画が生成されることを保証する。
これは mathlang の「同一トレース」と Design_BrainModel の「決定論ゲート」を、
言語仕様として一般化したものと読めます。

### 構築されたもの

| 領域 | 内容 |
|---|---|
| **言語** | 構文、意味論、操作的意味論、型仕様、名前空間解決 |
| **推論アーティファクト** | ReasoningModel / ReasoningEvaluationReport / ReasoningRuntimeResult |
| **オブジェクト形式** | ReasonUnit Object（RUO）と移行パス |
| **ランタイム** | RuntimeReal / HybridRuntime / NativeReasonUnitRuntime / ClusterRuntime / VisionRuntime / VisualizationRuntime / RuntimeComplex |
| **クロス言語** | Rust / Python / TypeScript / Go / Java の共通DTO契約 |
| **ツール** | `reason` CLI、`reason view` CodeViewer、IDE、VS Code拡張、LSP、Playground |
| **品質** | CI パイプライン（1,085テスト）、Conformance フレームワーク、Golden コーパス |

### バージョンの歩み

**v0.5.4.5（2026-08-07）** — 最新リリース。
Tensor Training Foundation v0.2（NCHW Conv2d / MaxPool2d / AvgPool2d、
リバースモード自動微分）と `.rstensor` ファイルプロファイルを追加。
同時にオープンソース公開に向けて、内部検証レポートや監査アーティファクトを整理しています。

**2026-06-15** — `Semantic Language Core v0.2` を凍結。

---

## 2026年7月 — 応用①：観測と推論を分ける

### VisionWorldModel（2026-07-24）

**問い：AIが世界の状態を推定するとき、観測と推論の境界をどう守るか？**

ReasonScript を**実際に使う**最初の本格的なプロジェクトです。
`src/` 配下はすべて `.rsn` ソースで書かれています。

分子構造（原子・結合）という明確な構造を持つ題材を使い、
「構造の変更を、いかに不可逆な破壊なく行うか」を検証しました。

### 確立された操作モデル

```
不変な候補を構築 → グラフ検証 → 明示的な状態移行 → アトミックなコミット
```

既存構造を直接書き換えず、**新しい世代を作って切り替える**。
構造の世代は `water@generation-1` / `water@generation-2` のように明示的に指定します。

### そして、判断しないという選択肢

`ACCEPT` / `REVISE` / `DEFER` / `ABSTAIN` の4値判定が導入されます。
`adaptive_abstention.rsn`（棄権）と `adaptive_reversibility.rsn`（可逆性）が
独立モジュールとして存在することに、この時点での問題意識が表れています。

Phase 1 → 2 → 3A → 3B-1 → 3B-2 → 3B-3 → 3C-1 と、
各段階で検証コマンド・テスト・JSON成果物・レポートを揃えながら進みました。

---

## 2026年8月 — 応用②：言語モデルの責務を分割する

### LanguageModel（2026-08-10）

**問い：言語能力・知識・推論を、単一のパラメータ集合から分離できるか？**

現時点で最新のプロジェクトであり、**Phase 0（基盤固定・仕様策定）が完了した段階**です。
実装は約450行、主成果物は仕様書です。

### 責務の分割

| 担い手 | 責務 |
|---|---|
| Neural Model | 言語知覚、意味写像、自然言語生成 |
| HolographicMemory | 分散的な連想、文脈活性化、パターン補完 |
| MRA Molecular Memory | 明示的知識・状態・経験・根拠・来歴の**正規**保存 |
| ReasonScript / MRA Runtime | 検証、決定論的操作、推論規則の実行 |

### Truth Boundary

COHERENT で芽生え、VisionWorldModel で「観測 vs 推論」として実装された分離が、
ここで **Truth Boundary** として明文化されます。

> HolographicMemory は Semantic Activation Field であり、真実の保存先ではない。
> ベクトル類似度、復号結果、ニューラルモデルの確信度だけを根拠として Relation を断定してはならない。

### 基盤の固定

`foundation.lock.json` により、ReasonScript v0.5.4.5 を**コミットハッシュ単位**で固定。

> `LanguageModel` は基盤を変更せず、その公開 CLI と決定論的な Reasoning 契約を利用する
> 独立プロジェクトである。

自作言語を自分で使う際に、**基盤側を都合よく書き換えない**という規律です。

---

## 通してみると

### 1. 対象の移動

```
数学教育 → 認知アーキテクチャ → ソフトウェア開発 → 言語処理系 → 視覚 → 言語
```

題材は移り変わっていますが、問いは一貫しています。
**「推論を、検査・再現・検証可能にするにはどうすればよいか」**

### 2. 抽象度の上下運動

```
mathlang（応用）
    ↓ 抽象化
COHERENT（アーキテクチャ）
    ↓ 抽象化
Design_BrainModel（制御レイヤー）
    ↓ 抽象化
ReasonScript（基盤言語）        ← 最も深い層まで降りた
    ↓ 具体化
VisionWorldModel / LanguageModel（応用）
```

応用から入り、必要に迫られて基盤まで降り、基盤ができてから再び応用に戻る——
という往復運動になっています。ReasonScript が谷底にあたります。

### 3. 「分離」という一貫したモチーフ

| プロジェクト | 分離しているもの |
|---|---|
| mathlang | 答え / 過程 |
| COHERENT | 想起（System 1）/ 推論（System 2） |
| Design_BrainModel | 設計意図 / 実装コード、提案する者 / 実装する者 |
| ReasonScript | 構文 / 意味 / 中間表現 / 実行計画 |
| VisionWorldModel | 観測 / 推論 |
| LanguageModel | 連想記憶 / 正規知識 |

**混ぜてはいけないものを混ぜない。** これが全体を貫くただ一つのモチーフです。

---

## 現在地

| プロジェクト | 状態 |
|---|---|
| ReasonScript | v0.5.4.5 リリース済み。ReasonGraph/World ビューア、パッケージレジストリ、SDK公開APIマニフェストが未実装 |
| Design_BrainModel | 自律実行ループ・実行安全制御は実装済み。Git統合と DesignUnit 統合が進行中 |
| COHERENT | Phase 2（Coding Agent Integration）に向けて開発中 |
| VisionWorldModel | Phase 3C-1 まで検証完了。適応的構造推論に着手 |
| LanguageModel | Phase 0 完了。Holographic Core 実装がこれから |
| mathlang | 2025-11 で更新停止（後続プロジェクトに発想が継承済み） |

---

→ [設計思想](design-philosophy.md) · [プロジェクト一覧](../README.md#プロジェクト一覧--projects)
