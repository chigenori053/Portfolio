# Case Study: ReasonScript — 決定論的コンパイラ・ランタイムの設計と実装

> このCase Studyは、ReasonScriptを「推論言語の研究」としてではなく、
> **言語処理系エンジニアリング（コンパイラ設計・ランタイム実装・CI/テスト基盤）**として評価するための要約です。
> 研究的な位置づけ・設計思想の詳細は [docs/projects/reasonscript.md](../docs/projects/reasonscript.md) を参照してください。

| | |
|---|---|
| **リポジトリ** | https://github.com/chigenori053/ReasonScript |
| **実装形態** | Hybrid DSL — コンパイラ/ツールチェーン: Python / 実行ランタイム: Rust（単一ネイティブホスト） |
| **規模** | 実装約154,300行（Python 98,518行 / Rust 55,826行）、仕様書102本、CI 1,240件 |
| **Status** | **VALIDATED** — `./reason ci` を実行し全9ステージ PASS・1,240テスト通過を確認済み（2026-08-31, commit `edfd477`） |

---

## 1. Problem

LLMベースの推論ワークフローには構造的な弱点がある。同じ入力でも実行結果が変わり（非決定性）、なぜその結論に至ったかが残らず（検証不可能性）、失敗したときにどこへ安全に戻すべきかが定義されない（ロールバック単位の不在）。これをライブラリやプロンプト技法ではなく、**言語仕様のレベル**で解決する処理系を作る。

先行プロジェクト Design_BrainModel v1 が、構造化推論の実行時に候補が非線形に増加してシステムフリーズに至った経験（[debugging-and-failure-analysis.md](debugging-and-failure-analysis.md#1-design_brainmodel-v1-の推論爆発)）が直接の動機になっている。

## 2. Requirements

- 同一入力から常に同一の実行結果が得られること（決定論）
- 推論の各ステップが検証可能であり、失敗時に安全な状態へ戻せること
- 実行を1つの言語ランタイムに一本化し、実装系統の分岐によって決定論が崩れないこと
- 複数言語（Rust / Python / TypeScript / Go / Java）間で推論結果の表現が一致すること

## 3. Constraints

- 単一ネイティブ実行ホストへの一本化が必要（Python/Rustの二重実行系は決定論を保証できない）
- 状態を変更する操作は仕様レベルで限定しなければならない
- 中間表現はスキーマで機械検証できる形式でなければならない

## 4. Architecture

4段階の中間表現を経る決定論的コンパイルパイプライン。

```
.rsn ソース → Surface AST → Semantic AST → Reason IR → ExecutionPlan → InferenceResult
```

推論プリミティブ以外の通常コードは別途 Computation IR へ lowering され、Rustランタイムホスト（`reason-runtime-host`）が実行する。Reason IR は JSON Schema（`reason_ir.schema.json`）で機械的に検証される。

言語の意味論は6プリミティブ（`goal` / `derive` / `prove` / `apply` / `converge` / `rollback`）による状態遷移として定義され、**状態を書き込む操作は `apply` と `rollback` の2つに限定**されている。

## 5. Design Decisions

- **状態変更操作を2つに限定**：`Proof` が `invalid` を含む場合、直前の安全な `State` チェックポイントへの自動ロールバックを言語意味論に組み込んだ。エラーハンドリングを後付けにせず、ロールバック安全性を言語仕様そのものに埋め込む設計判断。
- **実行系の一本化**：当初はPython実行系とRustランタイムの両方が実行経路として存在したが、2026年8月の Runtime Rust Consolidation（Phase 0–9）でPythonの本番実行フォールバックを撤廃し、実行を `reason-runtime-host` 1つに統合した。「実装が2系統あれば両者が食い違う余地が生まれる」という判断に基づく。
- **クロス言語DTO契約**：Rust / Python / TypeScript / Go / Java の5言語が単一の規範契約（`Common_DTO_Specification_v0.1.md`）を共有する設計とし、多言語環境での推論結果表現のずれを防いだ。

## 6. Implementation

- Reason IR・Computation IR の2つの中間表現とそれぞれのlowering処理をPython側に実装
- Rustランタイムを単一ワークスペース（`ReasonRuntime/`、6クレート: `computation-ir` `tensor-core` `reason-object-core` `reasoning-core` `vision-core` `runtime-cli`）に統合
- ReasonUnit Object（RUO）の全16関数をRustネイティブ（`reason-object-core`）で実装
- `reason` CLI（build/test/view/cluster/ci）、LSPサーバ（Phase 1）、VS Code拡張、ブラウザPlaygroundを構築

## 7. Testing

- `./reason ci --json` が9ステージ（チェックアウト→環境検証→ワークスペース検証→診断→アーティファクト→Golden→エージェントプロトコル→DTO互換性→テストスイート）を一気通貫実行
- Conformance framework（`conformance/run_conformance.py`）による全検証レイヤの実行と認証レポート更新
- Golden コーパステスト、差分テスト（Python参照実装 vs Rustランタイム）

## 8. Problems Found

`reason test` が、以前は静的なコンパイル・検証のみを行っており、**実行時ロジックの失敗を `PASS` と誤報告しうる**という問題があった（詳細: [debugging-and-failure-analysis.md](debugging-and-failure-analysis.md#2-reason-test-の静的検証のみによる誤pass)）。

## 9. Root Cause

テストフレームワークが `assert` / `assert_eq` を実際に実行せず、コンパイルの成否のみを判定基準にしていたため、ランタイム側の不整合を検出できていなかった。

## 10. Fix / Redesign

v0.5.5.8（Modernization Phase 3）で、`assert` / `assert_eq` をRustホスト / Computation IR VM上で実際に実行し、`COMPILE_ERROR` / `ASSERTION_FAILURE` / `RUNTIME_ERROR` を区別して報告する実行ベースのテストフレームワークに刷新した。

## 11. Verification

本ポートフォリオ作成にあたり実際に `./reason ci --json` を実行し、**全9ステージ PASS・1,240件のテスト通過を確認済み**（2026-08-31、commit `edfd477`、Python 3.14.0）。

## 12. Result

- 決定論的なコンパイル〜実行パイプラインが、CIによって継続的に検証される状態を確立
- 実行系統を1つに統合したことで、Python/Rust間の実装乖離という潜在的な決定論リスクを解消
- 5言語のクロス言語DTO契約により、多言語環境での型不整合リスクを設計段階で排除

## 13. Known Limitations

- ReasonGraph / World ビューアは読み取り専用の土台のみで、完全版は未実装
- パッケージレジストリ、SDK公開APIマニフェストは未実装
- GitHub Actions等の外部CIサービスとの統合有無は本ポートフォリオ作成時点で未確認（検証しているのは `./reason ci` をローカル実行した結果）

---

→ [ReasonScriptの詳細](../docs/projects/reasonscript.md) · [Cluster Runtime Case Study](cluster-runtime.md) · [Backend Engineering Evidence](../backend-engineering/overview.md)
