# VisionWorldModel

> **観測されたものと、推論されたものを、決して混ぜない世界モデル。**
> 根拠が不十分なら判断を保留する——ACCEPT / REVISE / DEFER / ABSTAIN の4値判定。

| | |
|---|---|
| **リポジトリ** | https://github.com/chigenori053/VisonWorldModel<br><sub>※ リポジトリ名は `Vison`（`i` 欠落）のタイポですが、URL維持のためそのままにしています</sub> |
| **開始** | 2026-07 |
| **主要言語** | **ReasonScript**（`src/` の全モデル定義）· Python（約7,700行 / 検証スクリプト） |
| **基盤** | ReasonScript `>=0.5.1` |
| **規模** | 検証スクリプト13本 · テスト14ファイル · ドキュメント37本 |
| **ライセンス** | 全権利留保（開発中の MRA を構成するため） |

---

## 位置づけ — MRA の視覚ドメインモデル

**MRA（Molecular Reasoning Architecture）の視覚ドメインモデル**です。
[LanguageModel](languagemodel.md)（言語ドメイン）· Design_BrainModel v2（ソフトウェア設計ドメイン）と
並んで、共通基盤の上に構築されます。

同時に、**ReasonScript で実際にドメインモデルを書いた最初の本格的な実装**でもあります。
`src/` 配下は `.rsn`（ReasonScript ソース）で構成されており、
言語が実用に耐えるかを検証する役割も担っています。

### なぜ「分子構造」を題材にしたのか

MRA は知識を **型付き Atom と Bond からなる Molecule** として表現するアーキテクチャです。
つまり分子構造は、比喩ではなく**表現形式そのもの**です。

実在の分子（原子・結合・価数・構造異性）を題材に選ぶことで、
**表現形式と対象領域の構造が一致し、かつ正解が曖昧さなく決まる**条件が得られます。
Molecular 表現が正しく機能するかを検証するうえで、これ以上ない試験台になっています。

```
src/
  atom.rsn                          bond.rsn
  atom_state.rsn                    bond_state.rsn
  bond_formation_request.rsn        bond_dissociation_request.rsn
  bond_dynamic_state.rsn            bond_state_conflict.rsn
  adaptive_context.rsn              adaptive_risk_evaluation.rsn
  adaptive_reversibility.rsn        adaptive_abstention.rsn
  adaptive_structural_reasoning_main.rsn        ...
```

---

## 2つの軸

このプロジェクトは2つのテーマが並走しています。

### 軸1：分子構造の決定論的検証

原子・結合・分子という**明確な構造を持つ対象**を題材に、
「構造の変更を、いかに安全に・不可逆な破壊なく行うか」を検証します。

### 軸2：VisionWorldModel — 画像から世界モデルへ

**観測された VisualAtom** と **推論された World 構成要素** を分離して保持し、
ハッシュで連結された判断（Decision）を生成します。

```bash
python3 scripts/vision_world_model_validation.py suite
python3 -m pytest -q tests/test_vision_world_model_validation.py
```

---

## ACCEPT / REVISE / DEFER / ABSTAIN — 判断しないという選択肢

VisionWorldModel の判断は4値です。

| 判断 | 意味 |
|---|---|
| **ACCEPT** | 根拠が十分。受理する |
| **REVISE** | 修正して再検討する |
| **DEFER** | 判断を**保留**する |
| **ABSTAIN** | 判断を**棄権**する |

**「わからない」を正常系として扱う**——ここがこのプロジェクトの中心的な主張です。

LLMは根拠が薄くても何らかの答えを返してしまいます。VisionWorldModel はその逆で、
根拠が不十分であることを**明示的な出力**として返します。判断の各結果はハッシュで連結され、
後から追跡可能です。

---

## 視覚テストレポート

画像 → 世界モデルへの変換を、視覚的に検証できます。

```bash
python3 scripts/vision_visual_test.py demo
```

`artifacts/vision_world_model/visual_tests/index.html` を開くと、
入力画像それぞれについて **VisualAtom · VisualBond · スコア · Decision のオーバーレイ**を
並べて比較できます。

### Phase 2 — 外部画像の解析

制御された画像だけでなく、任意の PNG / JPEG を解析できます。

```bash
python3 scripts/vision_phase2_analysis.py analyze path/to/image.png \
  --output artifacts/vision_world_model/phase2/sample
```

`report.html` が生成されます。日本語ガイドは `docs/VisionWorldModel_Phase2_Guide_ja.md`。

---

## Phase 構成 — 段階的な検証の積み上げ

各Phaseが、検証コマンド・テスト・機械可読な成果物（JSON）・人間可読レポートをセットで持ちます。

### Phase 1 — 分子構造の固定表現

```bash
python3 scripts/molecular_validation.py suite
```
→ `artifacts/validation_summary.json`

### Phase 2 — 状態遷移の検証

```bash
python3 scripts/state_transition_validation.py suite
```
→ `artifacts/state_transition_validation_summary.json`

### Phase 3A — 状態から推論へのマッピング

```bash
python3 scripts/reasoning_mapping_validation.py suite
```

決定論的な三状態推論タスク（`CLASS_A` / `CLASS_B` / `UNDETERMINED`、conflict はステータスとして扱う）を実装。

**判断は Phase 2 の状態トレースから、バージョン付きの evidence 抽出と ReasonScript の readout ルールを
通じて導出されます。フィクスチャ名や入力IDによる参照は一切使いません。**

——テストに通すための「答えの先読み」を構造的に禁じている点が重要です。

### Phase 3B-1 — 結合状態の遷移

```bash
python3 scripts/bond_state_transition_validation.py suite
```

Bond の定義とトポロジーは不変に保ったまま、`enabled` / `transmission` / `mode` の変更を、
**検証済みかつアトミックな提案（proposal）** として適用します。

### Phase 3B-2 — 結合型の遷移とバージョニング

```bash
python3 scripts/bond_type_transition_validation.py suite
```

`bond_type` を**構造的アイデンティティ**として扱います。型が変わるなら、それはもう別の構造である——
という立場から、新しい不変の Structure Version を構築・検証し、状態を明示的に移行し、
候補をアトミックにコミットします。

**構造の世代は明示的に指定しなければなりません：** `water@generation-1` / `water@generation-2`

### Phase 3B-3 — 結合の形成と解離

```bash
python3 scripts/bond_formation_dissociation_validation.py suite
```

H1 と H2 の間に `B3` を generation 3 として形成し、generation 4 として安全に解離します。
両操作とも、不変な候補・グラフ検証・明示的な状態移行・アトミックなコミットを経ます。

### Phase 3C-1 以降

分子の分割と結合（split / join）、複数分子の相互作用、適応的構造推論へと展開しています。

---

## 一貫する操作モデル

Phase 3B 以降で確立された、構造変更の手順が徹底されています。

```
  1. 不変な候補（immutable candidate）を構築
         ↓
  2. グラフ検証（graph validation）
         ↓
  3. 明示的な状態移行（explicit state migration）
         ↓
  4. アトミックなコミット（atomic commit）
```

既存の構造を直接書き換えることはありません。**新しい世代を作り、検証し、切り替える。**
データベースのマイグレーションやイミュータブルインフラの発想を、世界モデルに持ち込んだ設計です。

---

## 適応的構造推論（Adaptive Structural Reasoning）

`src/` の `adaptive_*.rsn` 群が、判断の適応的な制御を担います。

| ファイル | 役割 |
|---|---|
| `adaptive_risk_evaluation.rsn` | リスク評価 |
| `adaptive_cost_evaluation.rsn` | コスト評価 |
| `adaptive_reversibility.rsn` | **可逆性の評価** |
| `adaptive_budget.rsn` | 推論予算の管理 |
| `adaptive_oscillation.rsn` | 判断の振動検出 |
| `adaptive_abstention.rsn` | **棄権の判断** |
| `adaptive_constraint_evaluation.rsn` | 制約評価 |
| `adaptive_explanation.rsn` | 説明生成 |
| `adaptive_transaction_dispatch.rsn` | トランザクション制御 |

**可逆性**と**棄権**が独立したモジュールになっていることに、このプロジェクトの思想が表れています。
「戻せるか」と「やらないでおくか」を、推論の一級要素として扱っています。

---

## ReasonScript との統合

```toml
# reason.toml
[project]
reason_version = ">=0.5.1"

[compiler]
language_core = "0.7"
platform = "0.2"

[runtime]
backend = "RuntimeReal"
```

プロジェクト全体の検証：

```bash
reason ci --json
```

---

## この設計の背景にある考え方

VisionWorldModel が扱っているのは、表面的には分子構造と画像解析ですが、
本質的な問いは **「AIが世界の状態を推定するとき、観測と推論の境界をどう守るか」** です。

- 観測された VisualAtom と、推論された World 構成要素を**永続化レベルで分離**
- 根拠が足りなければ **DEFER / ABSTAIN** を返す
- 構造変更は必ず**新世代の構築 → 検証 → アトミックな切り替え**

この「観測と推論の分離」は、[LanguageModel](languagemodel.md) の **Truth Boundary**
（連想記憶と正規知識の分離）と同じ構図です。対象が視覚か言語かの違いだけで、原理は共通しています。

→ [設計思想の詳細](../design-philosophy.md)
