# Case Study: Debugging & Failure Analysis

「作れること」だけでなく「壊れたシステムを診断し、根本原因に基づいて設計を修正できること」を示すため、
実際に発生し、記録が残っている3件の事例をまとめる。設計案では他にも複数の事例候補が挙がっていたが、
本ポートフォリオの既存資料で裏付けが取れているのはこの3件のみであり、他は含めていない。

---

## 1. Design_BrainModel v1 の推論爆発

| | |
|---|---|
| **プロジェクト** | [Design_BrainModel](../docs/projects/design-brainmodel.md) |
| **症状** | 構造化推論の実行時、Unit候補の生成が非線形に増加し、探索空間が制御を超えて膨張。システムフリーズに至った |

### Problem

コーディングエージェントが設計意図に基づいてシステム構造を推論する過程で、候補生成が制御不能な速度で増加した。

### Root Cause

推論の実行そのものを制御・検証する仕組みが、アプリケーション層に散在していたこと。個別のバグではなく構造的な問題。

### Fix（と、あえて「解決」と言わない判断）

現状はヒューリスティックによって強引にフリーズを抑え込んでいるが、**その抑制は原理に基づいたものではない**。開発者自身の評価として「落ちなくなったこと」と「安定していること」は別であると明記し、**安定稼働に至ったと判断する材料がない**という結論を出している。パッチを重ねるのではなく、決定論的・有界・検証可能な言語基盤（ReasonScript）を先に作るという判断に至った。

### Verification

Design_BrainModel v1はPhase 6（大規模）・Phase 7（実リポジトリ）・Phase 8（人間評価）まで検証を実施しているが、推論爆発の抑制そのものについては安定動作を主張する検証結果は提示していない。

### Result

- 根拠のない「安定した」という主張を避け、未解決の限界として明示した
- この診断が、ReasonScriptにおける `ExecutionPlan` としての探索の明示化、`1,000 live value` ポリシー、境界のある autograd ライフサイクル管理、ループトレースの有界化という具体的な設計対応につながった

---

## 2. `reason test` の静的検証のみによる誤PASS

| | |
|---|---|
| **プロジェクト** | [ReasonScript](reasonscript.md) |
| **症状** | テストフレームワークが静的なコンパイル・検証のみを行い、ランタイム側の実行時ロジックの失敗を `PASS` と誤報告しうる状態だった |

### Problem

`reason test` が `assert` / `assert_eq` を実際には実行せず、コンパイルが通ることだけを合格の条件にしていた。

### Root Cause

コンパイル成功とランタイム上の正しい振る舞いを同一視していた設計上の見落とし。

### Fix

v0.5.5.8（Modernization Phase 3）で、`assert` / `assert_eq` をRustランタイムホスト / Computation IR VM上で実際に実行し、結果を `COMPILE_ERROR` / `ASSERTION_FAILURE` / `RUNTIME_ERROR` の3種類に区別して報告する、実行ベースのテストフレームワークへ刷新した。

### Verification

刷新後のテストフレームワークを含む形で `./reason ci` を実行し、全9ステージPASS・1,240件のテスト通過を確認済み（2026-08-31, commit `edfd477`）。

### Result

テストの合格が「コンパイルが通った」ではなく「実際に実行して検証済み」であることを保証する状態に修正された。

---

## 3. Rustランタイム統合時の重複実装の識別・削除

| | |
|---|---|
| **プロジェクト** | [ReasonScript](reasonscript.md) |
| **症状** | Rust側に目的が重複するランタイム実装（`ReasonComputationRuntime` `NativeReasonUnitRuntime` `VisionRuntime` の各ディレクトリ、重複するCargo lockfile）が並存していた |

### Problem

2026年8月の Runtime Rust Consolidation Plan（Phase 0–9）に先立ち、Python実行系からRustランタイムへの機能移行が段階的に進んだ結果、同じ責務を持つRust実装が複数のディレクトリに分散していた。

### Root Cause

機能追加のたびに新しいランタイムディレクトリが作られ、既存実装の廃止（deletion gate）が後回しになっていたこと。

### Fix

Phase 8でRustワークスペースを `ReasonRuntime/` に統合し、重複していた `ReasonComputationRuntime` `NativeReasonUnitRuntime` `VisionRuntime` ディレクトリと重複Cargo lockfileを削除。Phase 9で、統合の結果として未参照になった `Legacy/runtime` と `RuntimeComplex` のプレースホルダを削除した。

### Verification

統合後の `ReasonRuntime/`（6クレート構成）を本番実行ホストとして `./reason ci` を実行し、全ステージPASSを確認。`HybridRuntime` / `RuntimeReal` は差分テスト専用の参照実装として意図的に残し、本番実行パスからは明示的に外した。

### Result

本番実行パスが `reason-runtime-host` 1つに一本化され、「実装が2系統あれば両者が食い違う余地が生まれる」というリスクが解消された。

---

## まとめ

3件に共通するのは、**「動くようになった」ことと「その動作の根拠が説明できる」ことを区別している**点である。特に1件目（推論爆発）は、抑制はできているが安定の根拠がないという限界を隠さずに開示し、パッチではなく基盤の作り直しという判断につなげている。この判断基準は、[design-philosophy.md](../docs/design-philosophy.md#原則3--判断しないことを正当な出力にする) の「判断しないことを正当な出力にする」という原則を、自身のプロダクト評価に適用した結果でもある。

→ [ReasonScript Case Study](reasonscript.md) · [Design_BrainModelの詳細](../docs/projects/design-brainmodel.md) · [Evidence Index](../evidence/evidence-index.md)
