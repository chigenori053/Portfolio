# Advanced R&D — MRA（Molecular Reasoning Architecture）

このセクションは、[Backend Engineering Evidence](../backend-engineering/overview.md) で示したSoftware/Backend Engineeringの基盤の**上に位置する**、応用研究です。

```
Software Engineering
       ↓
Backend / Systems Engineering
       ↓
Advanced R&D  ← このページ
```

ReasonScriptという決定論的な言語基盤ができたことで、その上に「知識をどう表現し、AIの推論をどう検証可能にするか」という研究テーマに取り組めるようになりました。中心にあるのが **MRA（Molecular Reasoning Architecture）** — 知識を型付きAtomとBondからなるMoleculeとして表現する推論アーキテクチャです。

---

## 系譜

10ヶ月間の探索の中で、問題意識が「記述する→制御する→基盤から作る→基盤の上で複数ドメインに展開する」と移り変わってきました。詳細な経緯は [開発年表](../docs/timeline.md) を参照してください。

```
mathlang (2025-11)         数学学習支援言語。「過程を第一級のデータにする」という発想の起点
   ↓
COHERENT (2025-11)         非Transformer推論の理論検証（推論モデル: BrainModel）
   ↓
Design_BrainModel v1 (2026-01)   コーディングエージェント。推論爆発等の限界に直面 → 基盤作り直しの判断
   ↓
ReasonScript (2026-04)     決定論的な言語基盤（→ Software/Backend Engineeringの中心的成果）
   ↓
MRA ドメインモデル群 (2026-07〜)
   ├─ VisionWorldModel      視覚ドメイン（Phase 3C-1まで検証済み）
   ├─ LanguageModel         言語ドメイン（Phase 0完了、Truth Boundaryの定式化）
   └─ Design_BrainModel v2  ソフトウェア設計ドメイン（再設計予定）
```

## 中核となる原則 — Truth Boundary

連想記憶（HolographicMemory）は**候補を出すだけ**であり、事実を確定してはならない。「意味的に近い」ことと「その関係が成立する」ことを厳密に区別する、という原則です。

> HolographicMemory は Semantic Activation Field であり、真実の保存先ではない。
> ベクトル類似度・復号結果・ニューラルモデルの確信度だけを根拠として関係を断定してはならない。
> — MRA Holographic Semantic Language Model 仕様書 v0.1, §2.1

この原則は COHERENT の Accept/Review/Reject、VisionWorldModel の観測/推論分離にも共通して現れます。詳細は [設計思想](../docs/design-philosophy.md) を参照してください。

## プロジェクト一覧（詳細ページ）

| プロジェクト | 領域 | Status | 詳細 |
|---|---|---|---|
| COHERENT | 非Transformer推論の理論検証 | 検証継続中（項目により差あり） | [docs/projects/coherent.md](../docs/projects/coherent.md) |
| VisionWorldModel | MRA視覚ドメインモデル | Phase 3C-1まで検証完了 | [docs/projects/visionworldmodel.md](../docs/projects/visionworldmodel.md) |
| LanguageModel | MRA言語ドメインモデル | Phase 0完了（仕様策定） | [docs/projects/languagemodel.md](../docs/projects/languagemodel.md) |
| Design_BrainModel | ソフトウェア設計ドメインモデル | v1未完成 / v2再設計予定 | [docs/projects/design-brainmodel.md](../docs/projects/design-brainmodel.md) |
| mathlang | 系譜の起点（数学学習支援言語） | 開発停止（Apache-2.0） | [docs/projects/mathlang.md](../docs/projects/mathlang.md) |

主張ごとの証拠の強さは [Evidence Index](../evidence/evidence-index.md#advanced-rd) にまとめています。COHERENTの「記憶があれば計算しない」という中核仮説のように、**実装されているが性能としては未測定**の項目も正直に記載しています。

---

→ [README](../README.md) · [Backend Engineering Evidence](../backend-engineering/overview.md) · [設計思想](../docs/design-philosophy.md) · [開発年表](../docs/timeline.md)
