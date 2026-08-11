# ReasonScript

> **推論優先（reasoning-first）のプログラミング言語。**
> 決定論的実行、証明可能なAIワークフロー、ロールバック安全なシステムのための言語処理系。

| | |
|---|---|
| **リポジトリ** | https://github.com/chigenori053/ReasonScript |
| **開始** | 2026-04 |
| **現在バージョン** | v0.5.4.5（2026-08-07） |
| **主要言語** | Python（処理系）· Rust（ランタイム）· TypeScript / Go / Java（バインディング）· Elixir |
| **規模** | 実装約135,000行 · Pythonファイル543 · Rustファイル221 · テスト239ファイル / **1,085件パス** · 仕様・ドキュメント600本 |
| **位置づけ** | ポートフォリオ全体の**基盤**。VisionWorldModel と LanguageModel が直接依存 |

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

### 推論アーティファクト（Reasoning Artifacts）

「どうやってその結果に到達したか」を、バージョン付きの検査可能な記録として残します。

| アーティファクト | 役割 |
|---|---|
| `ReasoningModel` | 推論の構造そのものを表現するモデル |
| `ReasoningEvaluationReport` | 推論の評価結果レポート |
| `ReasoningRuntimeResult` | ランタイム実行の記録 |

### ReasonUnit Object（RUO）

推論単位を可搬・正規な形式で表現するオブジェクトフォーマット。ネイティブなランタイム型、CLI統合、
レガシー形式からの移行パスを備えています。

### クロス言語DTO契約

**Rust / Python / TypeScript / Go / Java の5言語が、単一の規範契約を共有します。**
（`docs/specifications/Common_DTO_Specification_v0.1.md`）

言語をまたいでも推論結果の表現がずれないことを保証する設計で、多言語環境でAI推論システムを組む際の
実務的な課題に対する解になっています。

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

---

## ランタイム構成

単一のランタイムではなく、目的別に複数のランタイムを持つ構成です。

| ランタイム | 役割 |
|---|---|
| `RuntimeReal` | 標準の実行ランタイム |
| `HybridRuntime` | Rust実装のハイブリッドランタイム。Reason IR バリデータを含む |
| `NativeReasonUnitRuntime` | ReasonUnit のネイティブ実行 |
| `ClusterRuntime` | Dynamic ReasonUnit クラスタ実行 |
| `VisionRuntime` | 視覚処理向けランタイム |
| `VisualizationRuntime` | Safe-Rust による意味構造の可視化ランタイム |
| `RuntimeComplex` | 複素数演算ランタイム |

---

## ツールチェーン

言語単体ではなく、開発体験まで含めて構築されています。

- **`reason` CLI** — ビルド、実行、検証、CI、アーティファクト管理
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

チェックアウト → ワークスペース検証 → 診断 → アーティファクト → Golden コーパス →
エージェントプロトコル → DTO互換性 → テストスイート、を一気通貫で実行します。
v0.5.4.5 時点で **1,085件のテストがパス**。

### Conformance フレームワーク

```bash
python3 conformance/run_conformance.py
```

全検証レイヤを実行し、認証レポートを更新します。

---

## v0.5.4.5 の主な内容

**追加**
- **Tensor Training Foundation v0.2** — NCHW形式の Conv2d / MaxPool2d / AvgPool2d、
  **リバースモード自動微分**、slice/gather、ステートレスなシード付き乱数テンソル生成、
  境界のある autograd ライフサイクル管理
- **`.rstensor` ファイルプロファイル** — チェックサム検証付き。capability チェックを伴う
  `tensor.load` / `tensor.save`、および JSON / CSV / NumPy 入力に対応した
  `reason tensor import|inspect|verify` コマンド

**修正**
- 到達不能になったテンソル値をランタイム環境から解放し、反復的なテンソルプログラムが
  「1,000 live value」ポリシーを使い切る問題を解消
- ループのトレーススナップショットを、全要素を暗黙的に実体化せずメタデータのシリアライズに変更して有界化
- 数値リテラルの10進指数表記を受理

---

## 仕様書群

`docs/specifications/` に40本以上の仕様書があります。主要なもの：

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
| `KEV-1_Knowledge_Emergence_Validation_Specification_v0.1-draft.md` | 知識創発検証（ドラフト） |

言語表層についても、AST対応・式パターン・文・型指定・名前空間解決がそれぞれ独立した仕様書になっています。

---

## 使ってみる

```bash
git clone git@github.com:chigenori053/ReasonScript.git
cd ReasonScript
pip install -e .
```

```bash
./reason ci --json
./reason reasoning-runtime run examples/v0_8/reasoning_runtime/animal_isa.rsn --json
```

プラットフォーム別インストーラは `docs/installation/`（Linux / macOS / Windows）にあります。

---

## 未実装 / 今後

README で明示的に「まだ実装されていない」と宣言されている項目：

- ReasonGraph / World ビューアの完全版（現状は読み取り専用の土台のみ）
- パッケージレジストリの設計
- LSP シンボルインデックスの、コンパイラソーススパンへの移行
- SDK 公開APIマニフェスト

---

## この設計の背景にある考え方

ReasonScript は「LLMを速く動かす」ための言語ではありません。
**AIの推論を、人間が事後に検査し、再現し、必要なら安全に巻き戻せるようにする**ための言語です。

そのために払っているコストは大きく、仕様書600本、テスト1,085件、5言語のDTOバインディングという規模になっています。
この投資が意味を持つのは、「AIの出力をそのまま信じるわけにはいかない領域」——安全性・監査・規制が
関わる領域でAIを使う場合です。

→ [設計思想の詳細](../design-philosophy.md)
