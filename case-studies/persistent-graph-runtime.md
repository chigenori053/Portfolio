# Case Study: Persistent Graph Runtime — 構造レベルの永続化とトランザクションモデル

> **注記（スコープの明確化）：** このCase Studyが扱っているのは、journal / fsync / WAL といった
> ディスクレベルの耐久性保証を持つストレージエンジンではありません。実体は
> [VisionWorldModel](../docs/projects/visionworldmodel.md) が確立した、**「不変な候補構築 → グラフ検証 →
> 明示的な状態移行 → アトミックなコミット」という構造レベルの永続化・トランザクションモデル**です。
> 本ポートフォリオは以前、証拠の伴わない性能主張を外部レビューで指摘され訂正した経緯があるため、
> 本ページでも実装範囲を実際に確認できるものに限定して記述します。

| | |
|---|---|
| **実体プロジェクト** | [VisionWorldModel](https://github.com/chigenori053/VisonWorldModel)（ReasonScript製、MRA視覚ドメインモデル） |
| **主要言語** | ReasonScript（`.rsn`） |
| **Status** | **VALIDATED**（Phase 3C-1までの検証コマンド・テスト・機械可読成果物により確認済み） |

---

## 1. Problem

グラフ構造（分子構造: 原子・結合）を変更する際に、変更途中の不整合な状態が外部から観測されたり、失敗時に構造が破壊されたりしないようにする必要がある。またAIによる観測（VisualAtom）と推論結果（World構成要素）を混同せず、どの時点の構造かを追跡可能にする必要がある。

## 2. Requirements

- 構造変更は必ず検証を経てからでなければ適用されないこと
- 変更途中の中間状態が外部から見えないこと（アトミック性）
- 構造のどの世代（バージョン）を参照しているかを明示できること
- 観測されたデータと推論されたデータを永続化レベルで分離できること

## 3. Constraints

- 既存構造を直接書き換える操作は許可しない
- 型（`bond_type`）が変わる変更は、同一構造の更新ではなく新しい構造として扱う

## 4. Architecture

一貫した4段階の操作モデル：

```
1. 不変な候補（immutable candidate）を構築
       ↓
2. グラフ検証（graph validation）
       ↓
3. 明示的な状態移行（explicit state migration）
       ↓
4. アトミックなコミット（atomic commit）
```

構造の世代は `water@generation-1` / `water@generation-2` のように明示的にアドレッシングされ、既存構造への書き換えではなく新世代の構築・検証・切り替えとして扱われる。データベースのマイグレーションやイミュータブルインフラの発想をグラフ構造に適用したもの。

Rustネイティブの `GraphTransaction` が、Python版と同等の操作集合（`unit_additions` 等）をサポートしている。

## 5. Design Decisions

- **書き換えではなく世代交代**：既存構造を直接変更せず、新しい世代（Structure Version）を構築・検証してから切り替える。これにより、検証に失敗した変更が既存構造を破壊するリスクを構造的に排除している。
- **観測と推論の永続化分離**：観測された `VisualAtom` と推論された World 構成要素を別々に永続化し、両者を混同しないようにしている。
- **判断を確定させない選択肢**：`ACCEPT` / `REVISE` / `DEFER` / `ABSTAIN` の4値判定により、根拠が不十分な場合はコミットせず保留・棄権できる設計。

## 6. Implementation

- Phase 3B-1: `enabled` / `transmission` / `mode` といった結合状態の変更を、検証済みかつアトミックな提案（proposal）として適用
- Phase 3B-2: `bond_type` を構造的アイデンティティとして扱い、型が変わる場合は新しい不変のStructure Versionを構築
- Phase 3B-3: 結合の形成（formation）と解離（dissociation）を、いずれも「不変な候補→グラフ検証→明示的状態移行→アトミックコミット」の手順で実装

## 7. Testing

各Phaseが検証コマンド・pytestテスト・機械可読な成果物（JSON）・人間可読レポートをセットで持つ。

```bash
python3 scripts/state_transition_validation.py suite        # Phase 2
python3 scripts/bond_state_transition_validation.py suite    # Phase 3B-1
python3 scripts/bond_type_transition_validation.py suite     # Phase 3B-2
python3 scripts/bond_formation_dissociation_validation.py suite  # Phase 3B-3
```

Phase 3A では、判断をフィクスチャ名や入力IDから直接参照することを構造的に禁止し、状態トレースからのevidence抽出のみで導出させることで、「テストを通すための答えの先読み」を防いでいる。

## 8. Problems Found

初期の分子構造モデルでは、構造変更の途中状態が外部から参照可能である場合、検証前のデータをもとに誤った判断が行われるリスクがあった（設計段階での識別）。

## 9. Root Cause

可変な構造に対して直接パッチを当てる方式では、検証とコミットの間に競合状態が生まれる。

## 10. Fix / Redesign

「不変な候補→グラフ検証→明示的状態移行→アトミックコミット」という手順を全Phase共通のルールとして確立し、以降のすべての構造変更（結合状態、結合型、結合の形成/解離）に適用した。

## 11. Verification

Phase 1〜3C-1の各段階について、検証コマンドの実行結果と機械可読な成果物（`artifacts/validation_summary.json` 等）が存在する。

## 12. Result

- 構造変更が常に「検証済みかつアトミック」であることを、Phase横断で一貫して保証するモデルを確立
- 観測データと推論データの分離を永続化レベルで実現
- ReasonScriptを実際のドメインモデルに適用した最初の本格的な実装として、言語自体の実用性も併せて検証

## 13. Known Limitations

- **ディスク上の耐久性保証（journal / fsync / クラッシュリカバリ）は実装・検証されていない。** このCase Studyが示すのは構造レベルの一貫性モデルであり、電源断やプロセスクラッシュに対する永続化保証ではない
- 複数分子間の相互作用・適応的構造推論はPhase 3C-1以降で展開中であり、検証が完了しているのはPhase 3B-3までの単一分子内の変更が中心

---

→ [VisionWorldModelの詳細](../docs/projects/visionworldmodel.md) · [Backend Engineering Evidence](../backend-engineering/overview.md) · [Evidence Index](../evidence/evidence-index.md)
