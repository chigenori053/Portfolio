# 開発年表 — 問題意識の変遷

2025年11月から2026年8月までの約10ヶ月間の流れです。
各リポジトリはコミット履歴が単一のため、日付は GitHub のリポジトリ作成日・更新日、
および仕様書・CHANGELOG の記載日に基づいています。

---

## 全体の流れ

```
2025-11 ────┬─── mathlang            「推論過程を記述できるか？」
            │
            └─── COHERENT            「Transformer 以外で LLM 同様の推論は可能か？」
                     │               （推論モデル: BrainModel）
                     │
2026-01 ─────────── Design_BrainModel v1
                     │               「設計意図からコードを想起できるか？」
                     │
2026-04 ─────────── ★ ReasonScript    「そもそも言語処理系から作り直すべきでは？」
                     │
                     ▼
              ★ MRA — Molecular Reasoning Architecture（開発中）
                     │               「知識を Molecule として表現し、
                     │                ドメインをまたいで推論できるか？」
     ┌───────────────┼───────────────────────┐
     ▼               ▼                       ▼
2026-07          2026-08                （再設計予定）
VisionWorldModel  LanguageModel       Design_BrainModel v2
  視覚ドメイン      言語ドメイン         ソフトウェア設計ドメイン
```

問題意識が **「記述する」→「制御する」→「基盤から作る」→「基盤の上で、複数ドメインに展開する」** と
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
**COHERENT はプロジェクトのブランド名**であり、中核の推論モデルは **BrainModel** です。

**問い：Transformer 以外の推論モデルで、LLM と同様または近い推論を実現できるか？**

製品開発ではなく**理論検証**が目的でした。中核仮説はこうです。

> **情報の密度を上げることで、ニューラルネットワークに頼らない推論が可能になるか。**

この仮説を2つの技術で検証します。

**① HolographicMemory（光学記憶）** — 物理現象である光学干渉を数理的にシミュレーションし、
**正しい / あいまい / 間違い**の判定と**言語生成（想起）**ができるかを検証する。

**② MemorySpace** — 永続化した HolographicMemory の記憶領域と計算空間を構築し、
推論内容から記憶を検索して、**該当があれば想起・なければ初めて推論を実行する**ことで、
計算資源を高効率に運用できるかを検証する。

### 検証結果

| 検証項目 | 結果 |
|---|---|
| 三値判定 | **成立** — `AcceptStore` / `ReviewStore` / `RejectStore` として実装 |
| 計算資源の効率化 | **実証** — 最適しきい値 θ\*=0.7 で**誤想起率 0%・計算削減 80.0%** |
| 数式の正誤・同値判定 | **成功** |
| 言語生成 | **部分的に成功** — 単語・多言語 100%、カタカナ 100%、漢字 60〜80% |

言語生成は、言語意味空間を埋め込んだ **DynamicHolographicMemory を含む3つの
HolographicMemory**（Dynamic / Static / Causal）で構成した MemorySpace で検証されました。
**記号そのものは保存せず、属性ホログラムから動的に生成する**という制約下での成果です。

日英60語を共有空間に格納しても劣化率 0.00%、`light` の問い合わせで `ひかり` と `light` が
ともに想起される言語横断連想も確認されています。

漢字が 60〜80% に留まった原因は、次元を1024→2048に拡大しても変化しなかったことから、
**記憶容量ではなく属性定義の重複**であると特定されました。

> mathlang が「過程を書けるようにした」のに対し、
> COHERENT は「過程を**自律的に生成し、記憶に沈殿させる**」ところまで進めています。
> そして**ここで有望に見えた HolographicMemory と MemorySpace が、
> 次の Design_BrainModel で実運用規模の壁に突き当たります。**

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

### そして、3つの限界に突き当たる

**DBM v1 は未完成プロダクトとして終わっています。** 理由は3つです。

| 限界 | 内容 |
|---|---|
| **推論爆発** | 構造化推論の実行時に **Unit 候補の生成が非線形に増加**し、探索空間が制御を超えて膨張。**システムフリーズ**に至った。現状は**強引に抑え込んでいる**状態で、**安定稼働に至ったと判断する材料がない** |
| **学習能力の不足** | 中核の推論 Core であった **MemorySpace** と **HolographicMemory** が、期待していた学習能力を発揮しなかった |
| **容量の非現実性** | **HolographicMemory のデータ容量が想定より大きく**、実用に耐えないと判断した |

これらは個別のバグではなく、**推論の実行そのものを制御・検証する仕組みが
アプリケーション層に散在している**という構造的な問題でした。

> **この判断が、次の転換を生みます。**
> パッチを当てるのではなく、**決定論的で、実行が有界で、検証可能な言語基盤を先に作る。**

---

## 2026年4月 — 基盤：言語処理系から作り直す

### ReasonScript（2026-04-12）

**問い：決定論的で検証可能な推論を書くための言語は、どうあるべきか？**

ここで**言語処理系そのものを作る**という選択がなされます。ポートフォリオ最大の投資であり、
**DBM v1 の3つの限界を解決するための、直接の帰結**です。

| DBM v1 の限界 | ReasonScript / MRA での対応 |
|---|---|
| 推論爆発 | 探索を **ExecutionPlan として明示化**し、実行を有界に保つ。`1,000 live value` ポリシー、境界のある autograd ライフサイクル、ループトレースの有界化——リソース上限を**言語ランタイムの責務**にした |
| 学習能力の不足 | 連想記憶に学習を期待しない設計へ転換。**正規知識は Molecular Memory に明示的に書く**（Truth Boundary） |
| 容量の非現実性 | HolographicMemory を**再構築可能な非正規層**として再定義。容量が問題なら破棄して作り直せる |

**Truth Boundary が理念であると同時に実務的な結論でもある**のは、この経緯によります。
「連想記憶は候補を出すだけ」という原則は、**記憶が勝手に賢くなることを期待して失敗した**
経験から導かれています。

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
COHERENT / BrainModel（推論アーキテクチャの理論検証）
    ↓ 実運用規模へ
Design_BrainModel v1（コーディングエージェント）
    ↓ 限界に直面し、さらに抽象化
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
| COHERENT / BrainModel | 想起（System 1）/ 推論（System 2）、Accept / Review / Reject |
| Design_BrainModel | 設計意図 / システム構造 / コード、提案する者 / 実装する者 |
| ReasonScript | 構文 / 意味 / 中間表現 / 実行計画 |
| VisionWorldModel | 観測 / 推論 |
| LanguageModel | 連想記憶 / 正規知識 |

**混ぜてはいけないものを混ぜない。** これが全体を貫くただ一つのモチーフです。

---

## MRA という到達点

2026年8月時点で、系譜は **MRA（Molecular Reasoning Architecture）** に収束しています。

知識を**型付き Atom と Bond からなる Molecule** として表現し、
Evidence と Provenance を伴う正規データとして扱う推論アーキテクチャです。
ReasonScript がその実装手段を、MRA が表現と推論の枠組みを担います。

そして MRA は、ドメインごとに専用モデルを持つ形で展開されます。

| ドメインモデル | 対象 | 状態 |
|---|---|---|
| **VisionWorldModel** | 視覚 — 観測から世界状態を推定する | Phase 3C-1 まで検証完了 |
| **LanguageModel** | 言語 — 自然言語から意味構造を再活性化する | Phase 0 完了 |
| **Design_BrainModel v2** | ソフトウェア設計 — 設計案からシステム構造を生成し、コードを想起する | 再設計予定 |

**視覚・言語・ソフトウェア設計という無関係な3ドメインが、同一の表現形式と推論契約を共有する。**
これが成立するかどうかが、MRA の妥当性そのものの検証になります。

Design_BrainModel v1 は、この構造の中では**先行実装**として位置づけられます。
独立した Rust プロジェクトとして推論・記憶・検証を自前で作り切ったことで、
共通基盤が満たすべき要件が明らかになりました。v2 ではそれらを ReasonScript と MRA Base に委ね、
ソフトウェア設計ドメイン固有の推論に集中します。

---

## 現在地

| プロジェクト | 状態 |
|---|---|
| ReasonScript | v0.5.4.5 リリース済み（Apache-2.0）。ReasonGraph/World ビューア、パッケージレジストリ、SDK公開APIマニフェストが未実装 |
| MRA | 開発中。Molecule / Evidence / Provenance と Truth Boundary を仕様として確立 |
| VisionWorldModel | Phase 3C-1 まで検証完了。適応的構造推論に着手 |
| LanguageModel | Phase 0 完了。Holographic Core 実装がこれから |
| Design_BrainModel | **v1 は未完成プロダクト**（推論爆発は強引な抑制のみで安定の根拠がなく、記憶機構も期待した学習と容量効率に至らず）。ReasonScript + MRA Base による v2 再設計を予定 |
| COHERENT | 理論検証を継続中。計算削減 80.0%（誤想起率 0%）· 数式の正誤判定 · 単語/多言語の想起 100% を実証。漢字生成 60〜80% が残課題（属性定義の重複が原因と特定済み） |
| mathlang | 2025-11 で更新停止（後続プロジェクトに発想が継承済み） |

---

→ [設計思想](design-philosophy.md) · [プロジェクト一覧](../README.md#プロジェクト一覧--projects)
