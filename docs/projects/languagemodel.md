# LanguageModel — MRA Holographic Semantic Memory

> **モデルパラメータだけに、言語能力・知識容量・推論能力のすべてを担わせない。**
> 連想記憶と正規知識を厳密に分離した、言語モデル基盤の研究。

| | |
|---|---|
| **リポジトリ** | https://github.com/chigenori053/LanguageModel |
| **開始** | 2026-08 |
| **主要言語** | Python · ReasonScript |
| **基盤** | **ReasonScript v0.5.4.5**（commit `7f29c1c31a06ad70abc4024bc4655873c43797b3` に固定） |
| **状態** | **Phase 0 完了**（基盤固定・仕様策定）。実装はこれから |
| **規模** | 実装 約450行 + 仕様書 |
| **ライセンス** | 全権利留保（開発中の MRA を構成するため） |

---

## 位置づけ — MRA の言語ドメインモデル

**MRA（Molecular Reasoning Architecture）の言語ドメインモデル**です。
[VisionWorldModel](visionworldmodel.md)（視覚ドメイン）· Design_BrainModel v2（ソフトウェア設計ドメイン）と
並んで、共通基盤の上に構築されます。

本プロジェクトの仕様書は、MRA 全体で共有される中核概念——
**Molecule · Evidence · Provenance · Truth Boundary**——を最初に定式化した文書でもあります。
言語ドメインの仕様であると同時に、アーキテクチャ全体の基礎資料としての役割を持っています。

## このプロジェクトの読み方

実装量は最小です。**現時点での主成果物は仕様書**であり、
「何を作るか」を規範語（MUST / MUST NOT / SHOULD / MAY）で厳密に定義した段階にあります。

このポートフォリオに含めているのは、コードの量ではなく
**設計をどこまで詰めてから書き始めるか**という進め方を示すためです。

---

## 検証しようとしている仮説

> 自然言語文から得た意味構造を分散複合表現へ符号化し、容量増加や干渉のある条件でも、
> 関連する正規 Molecule を高再現率で候補化できるか。
> また、候補を Evidence と Provenance により検証し、元の Relation Structure を
> 正確かつ再現可能に復元できるか。

汎用対話モデル全体ではなく、この仮説を検証できる最小システム
**MRA Holographic Semantic Memory v0.1（MHSM）** を最初の実装対象としています。

---

## 責務の分離

「言語モデルにすべてをやらせる」のではなく、4つの担い手に責務を分けます。

| 担い手 | 責務 |
|---|---|
| **Neural Model** | 言語知覚、意味写像、自然言語生成 |
| **HolographicMemory** | 分散的な連想、文脈活性化、パターン補完 |
| **MRA Molecular Memory** | 明示的知識、状態、経験、根拠、来歴の**正規**保存 |
| **ReasonScript / MRA Runtime** | 検証、決定論的操作、推論規則の実行 |

ニューラルモデルは「言葉を扱う」ことに専念し、**知識の正規性はシンボリックな側が持つ**、という分業です。

---

## Truth Boundary — このプロジェクトの中心原理

HolographicMemory は **Semantic Activation Field**（意味活性化の場）であり、真実の保存先ではない。

仕様書 §2.1 から：

1. HolographicMemory の出力は**常に候補とスコア**であり、事実判定ではない
2. ユーザーへの事実応答は、正規 Molecule の取得と Evidence 検証を通過しなければならない
3. **ベクトル類似度、復号結果、ニューラルモデルの確信度だけを根拠として Relation を断定してはならない**
4. HolographicMemory の削除・再構築によって正規知識が失われてはならない
5. 正規知識の変更は Molecular Memory の書き込み経路だけで行う

**4番目が特に効いています。** 連想記憶は「いつ捨てて作り直してもよいキャッシュ」として位置づけられており、
真実はそこにない、ということが可用性の設計として保証されています。

### 表現の三層分離

| 層 | 表現 | 役割 | 正規性 |
|---|---|---|---|
| **Semantic Space** | dense vector / semantic frame | 意味幾何、言語変種の吸収 | 非正規 |
| **HolographicMemory** | bound / superposed vector | 構造を含む連想活性化 | 非正規・**再構築可能** |
| **MRA Molecular Memory** | typed molecule graph | 関係、状態、経験、根拠、来歴 | **正規** |

論理的にも永続化上も分離することが MUST とされています。

### 構造と近さの分離

> 「意味的に近い」と「特定の関係が成立する」は区別する。
> たとえば `Alice` と `Book` が近く活性化されても、`Alice --likes--> Book` の
> Relation Molecule と有効な Evidence がなければ、その関係は成立しない。

埋め込みベクトルの類似度をそのまま知識として扱うRAG的な手法への、明確な批判になっています。

---

## システム構成

```text
Language Input
  → Input Normalizer
  → Semantic Encoder / Frame Parser
  → Typed Semantic Frame
  → Holographic Encoder
  → HolographicMemory Index
  → Candidate Molecule IDs + Activation Scores     ← ここまでは「候補」でしかない
  → Molecular Memory Retrieval
  → Schema / Evidence / State Validation           ← ここで初めて事実になる
  → Model A/B/C Reasoning Interface
  → Response Molecule
  → Language Decoder
```

### コンポーネント責務

| コンポーネント | 入力 → 出力 | 責務 |
|---|---|---|
| Input Normalizer | text, locale → normalized text | Unicode、空白、文境界、入力IDの正規化 |
| Semantic Encoder | normalized text → semantic frames | entity linking、predicate、role、modality、否定、時制の抽出 |
| Symbol Registry | labels / roles → symbol vectors | **version 付き**基底ベクトルの一意管理 |
| Holographic Encoder | semantic frame → composite vector | binding、superposition、正規化 |
| HolographicMemory | query vector → candidates | 近似探索、連想想起、activation 計算 |
| Molecular Memory | molecule IDs → molecules | 正規知識と版の永続化 |
| Validator | candidates + context → validated claims | schema、evidence、state、conflict の検査 |
| Reasoning Adapter | validated molecules → response molecule | Model A/B/C と ReasonScript への境界 |
| Audit Logger | pipeline events → trace artifact | 入出力、版、閾値、**棄却理由**の記録 |

Audit Logger が「棄却理由」まで記録する設計になっているのが、検証可能性への一貫した姿勢です。

---

## データモデル — Molecule

知識は「型付き Atom と Bond からなる、検証可能な単位」= **Molecule** として表現されます。

| 種別 | 表すもの |
|---|---|
| **Concept Molecule** | 人・物・抽象概念などの同一性と属性 |
| **Relation Molecule** | subject / predicate / object 等の関係 |
| **Event Molecule** | agent / patient / recipient / 時刻 を伴う出来事 |
| **Experience Molecule** | 状況・行為・結果・観測を関連付けた経験記録 |

これに **Evidence**（主張を支持または反証する参照可能な根拠）と
**Provenance**（入力・抽出器・モデル・時刻・変換履歴などの来歴）が付随します。

### HRR / VSA の用語

| 用語 | 定義 |
|---|---|
| **Semantic Frame** | predicate と型付き role-filler の集合 |
| **Binding** | role と filler を非可換的または識別可能に合成する演算 |
| **Superposition** | 複数の bound vector を固定次元ベクトルへ重畳する演算 |
| **Activation** | クエリに対する候補 Molecule の関連度。**真偽値ではない** |
| **Cleanup** | 近似復号ベクトルを既知の記号または Molecule 候補へ対応付ける処理 |

Holographic Reduced Representation（HRR）/ Vector Symbolic Architecture（VSA）に基づく
分散表現を採用しています。

---

## Model A / B / C

将来的に3つのモデルへ分岐する構想がありますが、v0.1 では interface stub で構いません。

| モデル | 役割 |
|---|---|
| **Model A** — Learning and Representation | 意味表現の学習、encoder 校正、新規概念候補の形成 |
| **Model B** — World Modeling and Future Prediction | 検証済みの World-State / Event Molecule から状態遷移・将来候補を生成 |
| **Model C** — Reasoning and Decision | 検証済み候補と過去の Experience Molecule から、規則・制約に基づく結論候補を形成 |

> **三モデルとも Holographic activation を事実として扱ってはならない。**

v0.1 の評価対象は、候補検索から Relation Recovery までに限定されています。

---

## 基盤の固定 — foundation.lock.json

このプロジェクトの実務的に面白い点は、**基盤をコミットハッシュ単位で固定している**ことです。

```json
{
  "schema_version": "mhsm-foundation-lock/0.1",
  "project": "mra-holographic-language-model",
  "reason_script": {
    "version": "0.5.4.5",
    "commit": "7f29c1c31a06ad70abc4024bc4655873c43797b3",
    "repository": "git@github.com:chigenori053/ReasonScript.git",
    "language_core": "0.7",
    "platform": "0.2",
    "runtime_backend": "RuntimeReal"
  },
  "contracts": {
    "reason_ir": "reason-ir/0.1",
    "common_dto": "0.1",
    "semantic_language_core": "0.2",
    "mra_holographic_memory": "0.1"
  }
}
```

> **`LanguageModel` は基盤を変更せず、その公開 CLI と決定論的な Reasoning 契約を利用する独立プロジェクトである。**

自作言語を自分で使う際に、**基盤側を都合よく書き換えない**という規律を明文化しています。
契約（contracts）まで版で固定することで、再現性の範囲が明確になります。

---

## スコープ

### v0.1 の対象

- 英語の単文を中心とした、限定された日本語単文を含む意味フレーム抽出
- Concept / Relation / Event / Experience Molecule の作成と参照
- Role-Filler 構造の holographic binding
- Molecule ID を返す連想検索
- **Exact Relation Recovery** と **Evidence Validation**
- メモリ容量・重畳数・ノイズに対する**劣化測定**
- 決定論的なオフライン評価と監査ログ
- 単一プロセスで動作する研究用参照実装

### v0.1 の非対象（明示的に除外）

- 大規模な事前学習済み基盤モデルの新規学習
- 自律エージェント、外部ツール実行、オンライン自己改変
- **HolographicMemory からの直接的な自然言語回答**
- ベクトルだけを用いた Evidence / Provenance / 時間状態の代替
- 無制限の長文理解、全言語対応、商用SLA

**やらないことを仕様書に書く**——スコープクリープを設計段階で防いでいます。

---

## 再現性の要件

> 同一のモデル、ベクトル辞書、メモリスナップショット、検索設定、乱数 seed、入力に対する
> 候補順位と検証結果は再現可能でなければならない。
> GPU 等による非決定性が残る場合は、**その範囲と許容差を記録する**。

非決定性を「ないことにする」のではなく、**残る場合は範囲を記録する**という現実的な扱いです。

---

## Phase 0 成果物

| 成果物 | 内容 |
|---|---|
| `MRA_Holographic_Semantic_Language_Model_Specification_v0.1.md` | 本体仕様書 |
| `docs/PHASE0_FOUNDATION.md` | Phase 0 基盤 |
| `docs/PHASE1_HOLOGRAPHIC_CORE.md` | Phase 1 設計 |
| `docs/REASONSCRIPT_PYTHON_BOUNDARY.md` | ReasonScript / Python の境界定義 |
| `foundation.lock.json` | 基盤固定情報 |
| `schemas/semantic_frame.schema.json` | Semantic Frame スキーマ |
| `schemas/molecule.schema.json` | Molecule スキーマ |

### 検証

```bash
../ReasonScript-v0.5.4.5/reason project-validate . --json
../ReasonScript-v0.5.4.5/reason check
python3 -m unittest tests/test_holographic.py -v
```

---

## この設計の背景にある考え方

現在の言語モデルは、言語能力・知識・推論のすべてを単一のパラメータ集合に押し込んでいます。
その結果として、知識の更新が難しく、根拠が示せず、幻覚が原理的に排除できません。

LanguageModel はその前提を置き換えようとしています。
**連想は連想として高速に働かせ、事実の確定は検証された正規データが行う。**

この構図は [COHERENT](coherent.md) の「想起 → 検証」、
[VisionWorldModel](visionworldmodel.md) の「観測 → 推論」と同一であり、
系譜全体で繰り返し現れる中心的なアイデアです。

→ [設計思想の詳細](../design-philosophy.md)
