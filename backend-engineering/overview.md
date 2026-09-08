# Backend Engineering Evidence

「バックエンドエンジニアとして採用できる根拠」を12項目に分けて一覧化したページです。
各項目について、実際に確認できる実装・成果物へリンクします。証拠が薄い項目（Security・Observability）は、誇張せずGapとして明記しています。

ステータス表記は [Evidence Index](../evidence/evidence-index.md) の語彙（IMPLEMENTED / TESTED / VALIDATED / PROTOTYPE / EXPERIMENTAL / DESIGN / RESEARCH / CONCEPT）に準拠します。

---

## 1. API / Interface Design

| 実装 | 内容 | Status |
|---|---|---|
| ReasonScript `reason` CLI | `ci` / `test` / `view` / `cluster` / `reasoning-runtime` のサブコマンド構成 | IMPLEMENTED |
| Common DTO Specification | Rust / Python / TypeScript / Go / Java の5言語が共有する単一の規範契約 | IMPLEMENTED |
| KuKKA REST API | `/api/bookings` `/api/contact` `/api/trial-slots` 等、Next.js API Routes | PROTOTYPE |

→ [ReasonScript Case Study](../case-studies/reasonscript.md) · [Backend Project: KuKKA](../projects/backend-service.md)

## 2. Data Modeling

| 実装 | 内容 | Status |
|---|---|---|
| Reason IR / Molecule Schema | JSON Schemaによる機械検証付きの中間表現・知識表現 | IMPLEMENTED |
| KuKKAのPrismaスキーマ | `Article` `Schedule` `TrialSlot` `Inquiry` `Booking` の5モデル、リレーション設計（`Booking`↔`TrialSlot`の1:1ユニーク制約） | IMPLEMENTED |

→ [Backend Project: KuKKA](../projects/backend-service.md)

## 3. Persistence

| 実装 | 内容 | Status |
|---|---|---|
| VisionWorldModelの構造永続化モデル | 不変候補構築→グラフ検証→明示的状態移行→アトミックコミット、世代アドレッシング | VALIDATED |
| KuKKAのPostgreSQL永続化 | Prismaマイグレーションによるスキーマ進化の記録 | PROTOTYPE |

> **Gap:** journal / fsync / クラッシュリカバリといったディスクレベルの耐久性保証は、いずれのプロジェクトでも実装・検証していません。

→ [Persistent Graph Runtime Case Study](../case-studies/persistent-graph-runtime.md)

## 4. Transaction Management

| 実装 | 内容 | Status |
|---|---|---|
| `GraphTransaction`（ReasonScript/VisionWorldModel） | Rustネイティブの操作集合（`unit_additions`等）によるアトミックな構造変更 | VALIDATED |
| ReasonScriptの `apply`/`rollback` | 状態を書き込む操作を2つに限定し、証明失敗時は自動ロールバック | IMPLEMENTED |
| KuKKAの一意制約 | `Booking.trialSlotId`のユニーク制約による二重予約防止（DBレベルの整合性） | PROTOTYPE |

→ [ReasonScript Case Study](../case-studies/reasonscript.md) · [Persistent Graph Runtime Case Study](../case-studies/persistent-graph-runtime.md)

## 5. Concurrency

| 実装 | 内容 | Status |
|---|---|---|
| ReasonScript Rustネイティブ Autograd | テープ+VJP、`NumericMode::NativeFast`（rayon並列化、実測1.42倍） | TESTED |

> **Gap:** Webサーバレベルでの並行リクエスト処理（コネクションプーリング、レースコンディション対策等）を実証する成果物は現時点でありません。

## 6. Distributed Execution

| 実装 | 内容 | Status |
|---|---|---|
| `ClusterRuntime`（Rustクレート）/ `reason cluster` CLI | 複数ワーカープロセスへのタスク分配、チェックサム付きメッセージプロトコル、失敗ワーカーの検知と再割当て、タイムアウト判定、予算ベースの収束制御 | **TESTED** |

worker coordination・retry・timeout・heartbeatメッセージは、ソースコード（`ClusterRuntime/src/runtime.rs`等）と単体テストの両方で確認済み。ただし**複数マシンにまたがるネットワーク分散実行**（single-machine複数プロセスを超える範囲）と、**heartbeat間隔に基づく継続的な生存監視ループ**は未確認。詳細は [Cluster Runtime Case Study](../case-studies/cluster-runtime.md) を参照してください。

## 7. Fault Tolerance

| 実装 | 内容 | Status |
|---|---|---|
| ReasonScriptの自動ロールバック | 証明失敗（`Proof`に`invalid`を含む）時、直前の安全な`State`へ自動復帰 | IMPLEMENTED |
| ClusterRuntimeのワーカー再割当て | ワーカー失敗検知時、`max_retries`の範囲で別ワーカーへ再割当て。障害注入テスト（`fail_task_once`）で経路自体を検証済み | TESTED |
| VisionWorldModelの`DEFER`/`ABSTAIN` | 根拠不十分時に判断を確定させない、という正常系としての設計 | VALIDATED |
| Design_BrainModel v1の失敗分析 | 推論爆発を「抑制できているが安定の根拠はない」と正直に評価した事例 | 参照: [debugging-and-failure-analysis.md](../case-studies/debugging-and-failure-analysis.md) |

## 8. Error Handling

| 実装 | 内容 | Status |
|---|---|---|
| `reason test`の構造化診断 | `COMPILE_ERROR` / `ASSERTION_FAILURE` / `RUNTIME_ERROR` の区別、`RT-CALL-003`（再帰深度超過）等 | TESTED |
| ランタイムの構造化診断 | ホスト不在・未対応lowering・capability拒否・bridge失敗を、Pythonを実行せず構造化診断として返す | IMPLEMENTED |

## 9. Security

> **Gap:** 認証・認可・暗号化といった一般的なWeb/ネットワークセキュリティの実装例は、本ポートフォリオには現時点でありません。KuKKAの管理画面には認証機構が確認できず、既知の限界として明記しています（[Backend Project: KuKKA §13](../projects/backend-service.md#13-known-limitations)）。

最も近い実装は Design_BrainModel の**コマンド分類による実行安全制御**（AIエージェントが実行するコマンドを危険度で分類し、破壊的操作を制限する仕組み）ですが、これはネットワーク越しのアクセス制御ではなく、ローカル実行エージェントの安全制御です。

→ [Design_BrainModelの詳細](../docs/projects/design-brainmodel.md)

## 10. Testing

| 実装 | 内容 | Status |
|---|---|---|
| ReasonScript CI | `./reason ci` 9ステージ・1,240テスト、実行して全PASSを確認済み（2026-08-31, commit `edfd477`） | VALIDATED |
| Conformance Framework | 全検証レイヤの実行と認証レポート自動更新 | IMPLEMENTED |
| Golden コーパス / 差分テスト | Python参照実装とRustランタイムの出力比較 | IMPLEMENTED |

→ [ReasonScript Case Study](../case-studies/reasonscript.md)

## 11. Observability

| 実装 | 内容 | Status |
|---|---|---|
| DecisionLog / Audit Logger（COHERENT, LanguageModel） | 判定結果に必ず根拠ログを添える設計。棄却理由まで記録 | DESIGN〜IMPLEMENTED（プロジェクトにより異なる） |
| `reason view` CodeViewer | ソースと中間表現（Surface AST〜ExecutionPlan）の対応表示 | IMPLEMENTED |

> **Gap:** これらは開発時のトレース/監査ログであり、本番運用を想定したメトリクス収集・アラート・ダッシュボード（Prometheus/Grafana相当）の実装例ではありません。

## 12. CI/CD

| 実装 | 内容 | Status |
|---|---|---|
| `./reason ci --json` | 9ステージの検証パイプライン。ローカル実行で全PASSを確認済み | VALIDATED |

> **Gap:** GitHub Actions等の外部CIサービスとの統合有無は、本ポートフォリオ作成時点で未確認です。確認できているのは `./reason ci` をローカルで実行した結果のみです。

---

→ [README](../README.md) · [Evidence Index](../evidence/evidence-index.md) · [Advanced R&D](../advanced-rd/overview.md)
