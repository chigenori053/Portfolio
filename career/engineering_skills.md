# Engineering Skills — 実務能力への変換表

独自研究の用語だけで語ると専門性が伝わりにくいため、企業開発で利用できる一般的な能力へ対応付ける。

| 研究開発上の成果 | 企業開発で利用できる能力 | 根拠 |
|---|---|---|
| Reason IR / ExecutionPlan(決定論的中間表現) | コンパイラ、データ処理基盤、実行計画生成の設計 | [ReasonScript](../projects/reasonscript.md) |
| Canonicalization(正規化・固定精度シリアライズ) | 再現可能な処理、キャッシュ設計、監査ログ設計 | [Design_BrainModel](../projects/design_brainmodel.md)(FNV-1a、`{:.6}`固定精度) |
| Golden Test / 回帰テスト | QA、テスト自動化、品質保証 | [ReasonScript](../projects/reasonscript.md)(`golden/`コーパス、CI 1,116件) |
| Evidence / Provenance(根拠・来歴の記録) | AIガバナンス、監査ログ、説明可能性の実装 | [LanguageModel](../projects/language_model.md)、[COHERENT](../projects/coherent.md)(DecisionLog) |
| Atomic Transaction / Rollback | 安全な状態更新、障害復旧設計 | [ReasonScript](../projects/reasonscript.md)(`apply`/`rollback`)、[VisionWorldModel](../projects/vision_world_model.md)(アトミックコミット) |
| Python / Rust Runtime実装 | バックエンド、CLI、基盤ソフトウェア開発 | [ReasonScript Runtimeケーススタディ](../case-studies/reasonscript_runtime.md) |
| 仮説検証・失敗分析(推論爆発の切り分け等) | AI評価、PoC設計、実験設計 | [Design_BrainModel](../projects/design_brainmodel.md#limitations)、[COHERENT](../projects/coherent.md) |
| 技術仕様書(MUST/MUST NOT規範語) | 要件整理、設計文書、チーム共有 | [Test Strategy](../evidence/test_strategy.md) |
| プログラミング教育 | 技術説明、オンボーディング、教材設計 | [Professional Profile](professional_profile.md) |
| PM経験 | スコープ管理、課題管理、関係者調整 | [Professional Profile](professional_profile.md) |

## 技術スタック

| 領域 | 技術 |
|---|---|
| 言語処理系 | 字句・構文解析、AST設計、中間表現(IR)、実行計画生成、型仕様、名前空間解決、状態遷移意味論の設計 |
| Rust | 言語ランタイム実装、60+ crateのワークスペース設計、Safe-Rust、Cargo、LSPサーバ |
| Python | 言語実行系・ツールチェーン実装、SymPyによる記号計算、pytest、uv |
| クロス言語 | Rust / Python / TypeScript / Go / Javaの共通DTO契約 |
| 数値計算 | 複素テンソル、Conv2d/MaxPool2d/AvgPool2d、リバースモード自動微分 |
| AI・推論 | HRR / VSA(分散表現)、記号推論、因果推論、ファジィ判定、マルチモーダル統合 |
| ツールチェーン | CLI、REPL、IDE、VS Code拡張、LSP、ブラウザPlayground、CIパイプライン |
| 品質保証 | Conformanceフレームワーク、Goldenコーパス、決定論ゲート、スキーマ検証 |

## 数値サマリ

| 指標 | 値 |
|---|---|
| プロジェクト数 | 7(mathlang、COHERENT、Design_BrainModel、ReasonScript、MRA、VisionWorldModel、LanguageModel) |
| 実装総行数(概算・依存関係およびLegacy除く) | 約251,000行(Python約150,000行、Rust約95,000行、TypeScript/Go/Java約6,200行) |
| ReasonScript CIテスト | 1,116件パス(実行確認済み、2026-08-12) |
| Rust crate数(Design_BrainModel) | 60+ |
| 対応言語バインディング | 5言語(Rust / Python / TypeScript / Go / Java) |
| 開発期間 | 約1年7ヶ月(2025-01のリサーチ期含む。実装を伴う期間は2025-11〜) |

> 行数は`find`によるファイル単位の集計であり、`.git/`・`node_modules/`・`.venv*/`・`site-packages/`・`target/`・`Legacy/`を除外している。**行数は開発量の目安であり、品質や難易度を示すものではない。** AIコーディングエージェントを併用した開発であるため、実装速度は従来の個人開発と単純比較できない。実質的な評価は[検証マトリクス](../evidence/verification_matrix.md)の再現可能な検証結果を参照。

→ [Professional Profile](professional_profile.md) · [希望職種](career_direction.md) · [検証マトリクス](../evidence/verification_matrix.md)
