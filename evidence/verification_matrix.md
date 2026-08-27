# Verification Matrix — 検証観点別のEvidence一覧

テスト総数だけでは「何を保証するテストか」が伝わらないため、検証観点ごとに実装・証拠・対応プロジェクトを整理する。

| 検証観点 | 内容 | 対応プロジェクト | 証拠 |
|---|---|---|---|
| **Schema validation** | 中間表現・データモデルのJSON Schema検証 | ReasonScript(Reason IR)、LanguageModel(Molecule/Semantic Frame) | `schemas/reason_ir.schema.json`、`schemas/molecule.schema.json`、`schemas/semantic_frame.schema.json` |
| **Invalid-input rejection** | 不正な入力・解答の拒否 | mathlang(ValidationEngine)、COHERENT(SymbolicEngine同値判定) | `core/validation_engine.py` + pytest(mathlang) |
| **Canonical serialization** | 正規化されたシリアライズ(ハッシュ前ソート、固定精度整形) | Design_BrainModel(`{:.6}`浮動小数点整形、Optionの符号化) | `DESIGN.md` 正規化規則 |
| **Deterministic planning** | 同一入力から同一実行計画の生成 | ReasonScript(ExecutionPlan) | コンパイルパイプライン設計 + CI 1,116件通過 |
| **Byte-identical artifacts** | 同一入力での完全一致するハッシュ/出力 | Design_BrainModel(`snapshot_v2`ハッシュの完全一致要求) | 決定論ゲート仕様(`DESIGN.md`) |
| **Golden tests** | 期待値との比較によるコンパイルパイプライン保護 | ReasonScript | `golden/` コーパス、`./reason ci` |
| **Atomic state transitions** | 不変候補→検証→状態移行→アトミックコミット | VisionWorldModel(Phase 3B-1〜3B-3) | 検証スクリプト + JSON成果物 |
| **Rollback** | 証明失敗時の自動ロールバック | ReasonScript(`prove`が`invalid`を含む場合の自動`rollback`) | 言語意味論(`apply`/`rollback`の2操作限定) |
| **Artifact integrity** | 成果物のチェックサム検証 | ReasonScript(`.rstensor`) | `reason tensor import\|inspect\|verify` |
| **Provenance verification** | 根拠・来歴の記録と検証 | COHERENT(DecisionLog)、LanguageModel(Evidence/Provenance、Audit Logger) | `coherent/core/memory/space/`、LanguageModel仕様書§2.1 |
| **Cross-runtime compatibility** | 複数言語/ランタイム間での契約共有 | ReasonScript(Rust/Python/TypeScript/Go/Javaの単一DTO契約) | `Common_DTO_Specification_v0.1.md` |
| **Regression prevention** | CI・Conformanceによる回帰防止 | ReasonScript(CI 1,116件)、VisionWorldModel(`reason ci`) | `./reason ci --json`、`conformance/run_conformance.py` |

## 証拠の強さの表記方針

本ポートフォリオ全体で、以下の分類を用いて主張の裏付けを明示する。

| 分類 | 意味 |
|---|---|
| **実測データあり** | 一次データ(CSV/JSONログ)に基づく実行結果 |
| **実行結果あり** | テストスイート・検証スクリプトの実行で確認 |
| **設計確認のみ** | 実装が設計どおり動くことをコードで確認したが、性能・効果としては未測定 |
| **未測定** | 実装はされているが、効果・性能を測る実験を行っていない |
| **再現不可** | レポートに記載はあるが、生成に必要なスクリプトが未収録 |

各プロジェクトページの「Validation Evidence」セクションで、この分類に従って主張を格付けしている。個別の格付け例は [COHERENT](../projects/coherent.md#validation-evidence) を参照。

→ [再現性](reproducibility.md) · [テスト戦略](test_strategy.md) · [ベンチマークサマリ](benchmark_summary.md)
