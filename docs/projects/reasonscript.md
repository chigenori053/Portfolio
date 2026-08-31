# ReasonScript

> **推論を記述するための、状態遷移記述言語。**
> Python をコンパイラ/ツールチェーン、Rust を単一の実行ランタイムとする **Hybrid DSL**。

| | |
|---|---|
| **リポジトリ** | https://github.com/chigenori053/ReasonScript |
| **開始** | 2026-04 |
| **現在バージョン** | v0.5.5.8（2026-08-30） |
| **言語種別** | **状態遷移記述言語（semantic reasoning state-transition language）** |
| **実装形態** | **Hybrid DSL** — コンパイラ/ツールチェーン: Python / 実行ランタイム: Rust（単一のネイティブ実行ホスト） |
| **規模** | 実装約154,300行 · Python 637ファイル 98,518行 · Rust 215ファイル 55,826行 · テスト関連ファイル455件 / **CI 1,240件パス（実行確認済み）** · 仕様書102本を含むドキュメント356本 |
| **ライセンス** | Apache-2.0 |
| **位置づけ** | ポートフォリオ全体の**基盤**。MRA の3ドメインモデルが依存 |

---

## 何を記述する言語なのか

ReasonScript の目的は、**推論を記述すること**です。
そのための言語形式として選択したのが**状態遷移**です。

仕様は、この位置づけを明確に否定形で述べています。

> *"The Semantic Language is **not** a knowledge representation language.
> It is a **semantic reasoning state-transition language**."*
> — `ReasonScript_Semantic_Language_Core_v0.2.md`（2026-06-15 凍結）

**知識を書く言語ではなく、推論の状態遷移を書く言語である。**
この区別が中核にあります。知識は書くものではなく、推論の結果として生成されるものだ、という立場です。

### 中核となる4原則

> 1. **Knowledge is not primitive. Knowledge is generated.**（知識は原始的なものではない。生成されるものである）
> 2. **Reasoning precedes Knowledge.**（推論が知識に先行する）
> 3. **Every Knowledge object contains complete evidence.**（すべての知識オブジェクトは完全な根拠を持つ）
> 4. **Semantic reasoning is deterministic.**（意味推論は決定論的である）

同一のグラフ・計画・制約が与えられれば、ランタイムは構造的に等しい結果と、
等しい正規化 JSON を出力します。

---

## 6つの状態遷移プリミティブ

言語の意味論は、6つのプリミティブによる状態遷移として定義されています。

```
  goal      → 望ましい状態を宣言する
  derive    → 候補となる推論を生成する
  prove     → 導出を検証する
  apply     → 検証済みの変更をコミットする
  converge  → 状態を安定させる
  rollback  → 直前の安全な状態へ戻す
```

各プリミティブは型付きのペイロードを持ちます。

| プリミティブ | 型 | 意味 |
|---|---|---|
| `goal` | `Symbol` | ゴール識別子 |
| `derive` | `Symbol` | 推論戦略のラベル |
| `converge` | `Symbol` | 安定化のラベル |
| `rollback` | `State` | 復元先の安全なチェックポイント |
| `apply` | `State` | コミットされたランタイム状態 |
| `prove` | `Proof` | 不変条件。`invalid` を含む場合は**自動ロールバックを起動** |

### 証明の失敗が、状態遷移として扱われる

`Proof` の内部文字列に `invalid` が含まれる場合、それは**決定論的な証明失敗**として扱われ、
**直前の安全な `State` チェックポイントへの自動ロールバックが起動します。**

`Symbol` は状態を変更しません。状態を書き込むのは `apply` だけであり、
復元するのは `rollback` だけです。**状態を触れる操作が2つに限定されている**——
この制約が、ロールバック安全性を言語レベルで成立させています。

「推論が失敗したときに、どこまで戻るか」を後付けのエラーハンドリングではなく、
**言語の意味論そのものに組み込んだ**設計です。

---

## 解こうとしている問題

LLMを使ったワークフローには構造的な弱点があります。

- **同じ入力でも実行結果が変わる** — 再現性がなく、検証もデバッグもできない
- **なぜその結論に至ったかが残らない** — 事後に監査できない
- **失敗したときに安全に戻せない** — ロールバックの単位が定義されていない

ReasonScript は、これらを**ライブラリやプロンプト技法ではなく、言語仕様のレベルで**解決しようとする処理系です。

### 直接の開発動機

抽象的な問題意識だけが出発点ではありません。
先行プロジェクト [Design_BrainModel v1](design-brainmodel.md) が、**3つの具体的な限界**で
実用水準に届かなかったことが直接の契機です。

| DBM v1 の限界 | ReasonScript での対応 |
|---|---|
| 構造化推論で候補が非線形に増加し、**システムフリーズ**に至った（強引な抑制のみで、安定稼働の根拠がない） | 探索を **ExecutionPlan として明示化**。`1,000 live value` ポリシー、境界のある autograd ライフサイクル、ループトレースの有界化により、**リソース上限を言語ランタイムの責務にした** |
| 記憶機構が期待した**学習能力を発揮しなかった** | 連想記憶に学習を期待しない設計へ。正規知識は明示的に書く |
| 分散表現の**容量が実用に耐えなかった** | 連想層を再構築可能な非正規層として再定義 |

**推論の実行を制御・検証する仕組みが、アプリケーション層に散在していた**——
これが v1 の構造的な問題でした。ReasonScript は、その責務を言語基盤側に引き受けています。

---

## コアとなる仕組み

### 決定論的コンパイルパイプライン

ソースコードは4段階の中間表現を経て、検証済み・再現可能な実行結果になります。

```
  .rsn ソース
      │
      ▼
  Surface AST        ← 構文の忠実な表現
      │
      ▼
  Semantic AST       ← 名前解決・型・スコープの確定
      │
      ▼
  Reason IR          ← 推論の正規中間表現（JSON Schema で検証）
      │
      ▼
  ExecutionPlan      ← 決定論的な実行計画
      │
      ▼
  InferenceResult    ← 検証済みの実行結果
```

**同一入力からは、必ず同一の ExecutionPlan と InferenceResult が生成されます。**
Reason IR は `schemas/reason_ir.schema.json` によって機械的に検証されます。

推論プリミティブ（`goal`/`derive`/…）以外の通常の関数・`calculation`ブロックは、
これとは別に **Computation IR**（`reason-computation-ir`）という中間表現へ Python 側で lowering され、
Rust ランタイムホストが直接実行します。両方の中間表現を Python がコンパイルし、
実行そのものは後述のとおり Rust 側に一本化されています。

### 推論アーティファクト（Reasoning Artifacts）

「どうやってその結果に到達したか」を、バージョン付きの検査可能な記録として残します。

| アーティファクト | 役割 |
|---|---|
| `ReasoningModel` | 推論の構造そのものを表現するモデル |
| `ReasoningEvaluationReport` | 推論の評価結果レポート |
| `ReasoningRuntimeResult` | ランタイム実行の記録 |

### ReasonUnit Object（RUO）

推論単位を可搬・正規な形式で表現するオブジェクトフォーマット。ネイティブなランタイム型、CLI統合、
レガシー形式からの移行パスを備えています。v0.5.5系で、`ruo.*` の16関数すべてが
**Rustネイティブ**（`reason-object-core`）で実行されるようになりました。

### クロス言語DTO契約

**Rust / Python / TypeScript / Go / Java の5言語が、単一の規範契約を共有します。**
（`docs/specifications/Common_DTO_Specification_v0.1.md`、バインディングは `dto/` 配下）

言語をまたいでも推論結果の表現がずれないことを保証する設計で、多言語環境でAI推論システムを組む際の
実務的な課題に対する解になっています。

なお **Go と Java は DTO バインディングのみ**（それぞれ数百行）であり、
処理系の実装言語ではありません。

---

## Hybrid DSL — Python コンパイラ/ツールチェーン + Rust 単一実行ランタイム

ReasonScript は単一言語で実装された処理系ではありません。ただし**役割分担は v0.5.4系から大きく変わりました。**
v0.5.4.5 時点では「実行系は Python、ランタイムは Rust」という説明が成立していましたが、
2026-08-24〜08-30 にかけて完了した **Runtime Rust Consolidation Plan（Phase 0〜9）** により、
**Python は本番実行から退役し、実行そのものは Rust ランタイムホストに一本化されました。**

| 層 | 言語 | 担当 | 現在の位置づけ |
|---|---|---|---|
| **コンパイラフロントエンド / ツールチェーン** | **Python** | パーサ・バリデータ・意味解析、Reason IR / Computation IR への lowering、`reason` CLI、CI、アーティファクト管理、SDK、Conformance | 現役。**唯一のフロントエンド** |
| **実行ランタイム** | **Rust** | `reason-runtime-host`（`ReasonRuntime/` ワークスペース）が Computation IR VM・Tensor演算・RUO・Vision・Reasoning core を実行 | 現役。**唯一の本番実行パス** |
| Python 実行系（AST evaluator, Computation IR interpreter, Tensor/RUO/Vision runtime） | Python | 差分テスト・ベンチマークの参照実装 | **参照専用に降格。** standalone実行・project実行・project検証・Tensorアーティファクトコマンド・マニフェスト生成からの import は禁止 |
| **DTO バインディング** | TypeScript / Go / Java | 型定義の共有のみ | 現役 |

### なぜ変わったのか

`docs/development/python_reference_runtime.md` は、この方針を明確に定めています。

> *"Product execution uses `reason-runtime-host`; a missing host, unsupported lowering,
> capability denial, bridge failure, or native runtime error is returned as a structured
> diagnostic without executing Python."*

Python 評価器は差分テストとベンチマークからのみ import が許され、参照実装としての価値
（差分オラクル）がネイティブの golden vector や独立した仕様ハーネスに置き換わり次第、
削除される予定です（Phase 9 deletion gate、現時点では未削除）。

`HybridRuntime/` `RuntimeReal/` といった旧ディレクトリは、差分テストと SDK/DTO 互換性テストが
依存しているため現時点でも残っていますが、**本番実行パスからは外れています**。
Rust 側で重複していた `ReasonComputationRuntime` `NativeReasonUnitRuntime` `VisionRuntime`
ディレクトリと、未参照になった `Legacy/runtime`・`RuntimeComplex` プレースホルダは削除され、
`ReasonRuntime/`（`computation-ir` `tensor-core` `reason-object-core` `reasoning-core`
`vision-core` `runtime-cli` の6クレート）に統合されています。

`Legacy/elixir_runtime/` に Elixir 実装が残っていますが、これは
**開発初期に分散ランタイムとして導入を計画していたもの**で、その後の設計収束によって
不要となり、現行の構成からは外れています（`Legacy/` 配下にのみ存在）。

---

## 言語の見た目

```reasonscript
module Basic {

    fn Value() -> int {
        return 42
    }

    calculation Result {
        result = Value()
    }

}
```

`calculation` ブロックが特徴的で、通常の関数とは別に「計算・推論の単位」を言語構文として持ちます。

v0.5.5.8 で言語化された代数的 Enum / Optional / パターンマッチングは、こう書きます。

```reasonscript
module OptionalMatch {
    fn Score(value: optional<int>) -> int {
        match value {
            some(x) => return x
            none => return 0
        }
    }

    calculation Answer {
        result = Score(some(42))
    }
}
```

---

## ランタイム構成

Rust ランタイム統合の完了により、**本番実行は `reason-runtime-host` 1つに一本化**されています。
以前の「目的別に複数のランタイムを持つ」構成のうち、Python 側とレガシー Rust ワークスペースは
差分テスト用の参照実装としてのみ残っています。

| ランタイム | 言語 | 現在の役割 |
|---|---|---|
| `reason-runtime-host`（`ReasonRuntime/crates/runtime-cli`） | Rust | **唯一の本番実行ホスト**。Computation IR VM・Tensor・RUO・Vision・Reasoning core をすべて実行 |
| `computation-ir` / `tensor-core` / `reason-object-core` / `reasoning-core` / `vision-core` | Rust | `reason-runtime-host` が呼び出すライブラリクレート |
| `RuntimeReal` / `HybridRuntime` | Python / Rust | 差分テスト・SDK/DTO互換性テスト専用の参照実装。**本番実行では使用されない** |
| `ClusterRuntime` | Rust | Dynamic ReasonUnit クラスタ実行 |
| `VisualizationRuntime` | Safe-Rust | 意味構造の可視化ランタイム。型で「実行時に壊れないこと」を保証 |

---

## ツールチェーン

言語単体ではなく、開発体験まで含めて構築されています。

- **`reason` CLI** — ビルド、実行、検証、CI、アーティファクト管理
- **`reason test`** — v0.5.5.8 で**実行ベースに刷新**。以前は静的なコンパイル・検証のみで
  実行時ロジックの失敗を `PASS` と誤報告しうる問題があったが、`assert` / `assert_eq` を
  Rust ホスト / Computation IR VM 上で実際に実行し、`COMPILE_ERROR` / `ASSERTION_FAILURE` /
  `RUNTIME_ERROR` を区別して報告する
- **`reason view`** — ターミナル上の CodeViewer。`.rsn` ソースと、そこから生成された
  Surface AST / Semantic AST / Reason IR / ExecutionPlan を**対応付けて表示**する
  （curses製の対話UI、`--json` / `--plain` 出力、ファイルツリーブラウザ付き）
- **`reason cluster`** — クラスタ実行の計画・実行・シミュレーション・検証・比較
- **IDE** — `apps/reasonscript-ide`
- **VS Code 拡張** — `vscode-extension/`（MITライセンス）
- **ブラウザ Playground** — `playground/`
- **LSP サーバ** — Phase 1 実装済み

### CI パイプライン

```bash
./reason ci --json
```

チェックアウト → 環境検証（バージョン一貫性） → ワークスペース検証 → 診断 → アーティファクト →
Golden コーパス → エージェントプロトコル → DTO互換性 → テストスイート、を9ステージで一気通貫実行します。
**本ポートフォリオの更新にあたり実際に実行したところ、全9ステージ PASS・1,240件のテスト通過を確認しました**
（2026-08-31 / commit `edfd477` / Python 3.14.0）。

ソースからビルドする場合は、先に Rust ランタイムホストのビルドが必要です。

```bash
cargo build --manifest-path ReasonRuntime/Cargo.toml --bin reason-runtime-host
```

### Conformance フレームワーク

```bash
python3 conformance/run_conformance.py
```

全検証レイヤを実行し、認証レポートを更新します。

---

## v0.5.4.5 → v0.5.5.8 の主な変化

2026-08-12（v0.5.4.6）から 2026-08-30（v0.5.5.8）にかけて、アーキテクチャと言語機能の両方に
大きな変更が入りました。

### アーキテクチャ: Rust ランタイム統合（Runtime Rust Consolidation Plan, Phase 0–9）

- **Phase 6–7**：推論4関数（`runtime.search` / `simulate` / `predict` / `plan`）を Rust の
  in-process reasoning core に接続し、**Python 実行系の本番フォールバックを撤廃**
- **Phase 8**：Rust ワークスペースを `ReasonRuntime/` に統合。重複していた
  `ReasonComputationRuntime` `NativeReasonUnitRuntime` `VisionRuntime` ディレクトリと
  重複 Cargo lockfile を削除
- **Phase 9**：未参照になった `Legacy/runtime` と `RuntimeComplex` プレースホルダを削除

### 言語機能: Modernization Phases 0–5（v0.5.5.8）

- **Phase 0** 実行可能チェック契約 — CLI/ツールチェーン全体で構造化チェック契約を強制
- **Phase 1** 列挙型・Optional・パターンマッチング統合 — 代数的 Enum / Optional と
  網羅的/ワイルドカード `match` のネイティブランタイムサポート
- **Phase 2** 文字列・コレクション標準ライブラリ — `string.*` とコレクション操作関数
- **Phase 3** 実行ベーステストフレームワーク — 上述の `reason test` 刷新
- **Phase 4** 制御された再帰 — コールグラフ循環解析と `max_call_depth`（既定128）による
  スタックガード、`RT-CALL-003` 診断
- **Phase 5** モジュール・マニフェスト・ReasonGraph 整合性 — Rust ネイティブの
  `GraphTransaction` が Python 版と完全同等の操作集合（`unit_additions` 等）をサポート

### v0.5.5.3〜v0.5.5.6 で追加された計算機能

- `optimizer.*`（SGD / Momentum / Adam / AdamW）、`relation.*`（関係代数のフィルタ/ソート/distinct）
  の各 namespace
- Computation IR のオプティマイザ（定数畳み込み・デッドコード除去・局所CSE）
- Rust ネイティブの Autograd（テープ + VJP）、`NumericMode::NativeFast`
  （rayon並列化。700×700行列積で実測1.42倍の高速化、シーケンシャル実装とのビット完全一致を保証）

---

## 仕様書群

`docs/specifications/` に102本の仕様書があります。主要なもの：

| 仕様書 | 内容 |
|---|---|
| `ReasonScript_Language_Specification_v0.1.md` | 言語仕様 |
| `ReasonScript_Semantic_Language_Core_v0.2.md` | 意味論コア（**2026-06-15 凍結**） |
| `ReasonScript_Operational_Semantics_v0.1.md` | 操作的意味論 |
| `Common_DTO_Specification_v0.1.md` | クロス言語DTO契約 |
| `ReasonScript_Computation_Model_v0.1.md` | 計算モデル |
| `ReasonScript_ABI_Specification_v0.1.md` | ABI仕様 |
| `ReasonScript_Agent_Development_Protocol_v1_0.md` | エージェント開発プロトコル |
| `Conformance_Framework_Specification_v0.1.md` | 適合性検証フレームワーク |
| `ReasonScript_Execution_Based_Test_Framework_v0_1.md` | 実行ベーステスト仕様（Phase 3） |
| `ReasonScript_Controlled_Recursion_Phase4_v0_1.md` | 制御された再帰仕様（Phase 4） |
| `docs/development/runtime_rust_consolidation_plan.md` | Rust ランタイム統合計画（Phase 0–9、`COMPLETED`） |
| `KEV-1_Knowledge_Emergence_Validation_Specification_v0.1-draft.md` | 知識創発検証（ドラフト） |

言語表層についても、AST対応・式パターン・文・型指定・名前空間解決がそれぞれ独立した仕様書になっています。

---

## 使ってみる

```bash
git clone git@github.com:chigenori053/ReasonScript.git
cd ReasonScript
pip install -e .
cargo build --manifest-path ReasonRuntime/Cargo.toml --bin reason-runtime-host
```

```bash
./reason ci --json
./reason reasoning-runtime run examples/v0_8/reasoning_runtime/animal_isa.rsn --json
```

プラットフォーム別インストーラは `docs/installation/`（Linux / macOS / Windows）にあります。

---

## 未実装 / 今後

README 自体は v0.5.4.5 時点の記述のまま更新されていませんが、`docs/roadmap.md` の
直近の状態を見る限り、以下は引き続き未着手です（v0.5.5.8 時点で確認）：

- ReasonGraph / World ビューアの完全版（現状は読み取り専用の土台のみ。`reason view` が
  ソース〜ExecutionPlan の対応表示を提供するが、ReasonGraph と World 自体のビューアは未実装）
- パッケージレジストリの設計
- LSP シンボルインデックスの、コンパイラソーススパンへの移行
- SDK 公開APIマニフェスト

---

## この設計の背景にある考え方

ReasonScript は「LLMを速く動かす」ための言語ではありません。
**AIの推論を、人間が事後に検査し、再現し、必要なら安全に巻き戻せるようにする**ための言語です。

推論を**状態遷移として記述させる**という選択が、その中心にあります。
推論を自由なテキスト生成として扱えば、どこで何が起きたかは追跡できません。
`goal` → `derive` → `prove` → `apply` という遷移として書かせれば、
**各ステップに検証点が生まれ、失敗時の戻り先が定義されます。**

同じ理由で「知識を書く言語ではない」と明示されています。
知識を直接書けてしまうと、その知識がどこから来たのかが失われる。
だから **知識は推論の結果として生成され、必ず完全な根拠を伴う**という原則が置かれています。

Python の本番実行フォールバックを撤廃し、実行を Rust ランタイムホスト1つに一本化した
v0.5.5系の変更も、同じ理由に基づきます。**実装が2系統あれば、両者が食い違う余地が生まれる。**
決定論を言語レベルで保証するなら、本番の実行経路は1つであるべきだ、という判断です。

そのために払っているコストは大きく、仕様書102本、CI テスト1,240件、5言語のDTOバインディングという規模になっています。
この投資が意味を持つのは、「AIの出力をそのまま信じるわけにはいかない領域」——安全性・監査・規制が
関わる領域でAIを使う場合です。

→ [設計思想の詳細](../design-philosophy.md) · [開発年表](../timeline.md)
