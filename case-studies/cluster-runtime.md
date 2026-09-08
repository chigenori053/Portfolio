# Case Study: Cluster Runtime — Worker調整・Retry・Timeoutを備えた分散実行

> **改訂履歴：** 本ページは公開時、ReasonScriptのポートフォリオ側ドキュメント（概要2行のみ）を根拠に
> 意図的に薄く書いていました。その後、**ReasonScript本体（`ClusterRuntime/` クレート）のソースコードを
> 直接調査**したところ、worker調整・retry・timeout・heartbeatメッセージ・障害シミュレーションによる
> テストが実装として確認できたため、2026-09-08に内容を全面的に更新しました。
> 過大表示を避けるため、**確認できたことと確認できていないことを引き続き明確に分けて**記述します。

| | |
|---|---|
| **実体プロジェクト** | [ReasonScript](https://github.com/chigenori053/ReasonScript) — `ClusterRuntime/`（Rustクレート）+ `toolchain/cluster_runtime_cmd.py`（CLI） |
| **主要言語** | Rust（ランタイム本体）/ Python（CLIアダプタ・テスト） |
| **Status** | **TESTED** — ソースコードと単体テストで直接確認済み。ReasonScript全体のCIスイート（`./reason ci`、1,240テストPASS実行確認済み、2026-08-31）に含まれる |

---

## 1. Problem

`Dynamic ReasonUnit` を複数のワーカープロセスに分配して実行する際、(a) ワーカーが失敗した場合にタスクを失わずに再割当てできること、(b) 応答しないワーカーで処理全体が止まらないこと、(c) 実行の全経緯（誰が何をいつ送ったか）を後から検証できること、が必要になる。

## 2. Requirements

- コーディネータとワーカー間のメッセージに一意なチェックサムを付け、改ざん・欠落を検出できること
- タスク失敗時に、設定された上限回数まで別ワーカーへ再割当て（retry）できること
- タスクが一定時間内に完了しない場合はタイムアウトとして扱えること
- ワーカーの生存確認（heartbeat）の仕組みを持つこと
- 単一ノード実行（single-node）と複数ワーカー実行（local_process）の結果を比較・検証できること
- 障害シナリオ（`fail_task_once`）を注入してretry経路自体をテストできること

## 3. Constraints

- 決定論（`deterministic`設定）を維持したまま並行実行を行う必要がある
- リソース上限（`max_workers` `max_tasks` `max_logical_steps` `max_message_bytes` `max_state_bytes`）を設定として明示し、超過時は診断コード付きで打ち切ること

## 4. Architecture

`ClusterRuntime/src/` は責務ごとにモジュール分割されている。

| モジュール | 役割 |
|---|---|
| `config.rs` | クラスタ設定（`ClusterConfig` / `Limits` / `ExecutionConfig` / `TestingConfig`） |
| `planner.rs` | タスクをlogical stepとpartitionに分割する実行計画（`ClusterPlan`） |
| `worker.rs` | ワーカープロセスの実行（ReasonScriptランタイムホストをサブプロセスとして起動し、標準入出力でリクエスト/レスポンスをやり取り） |
| `messages.rs` | チェックサム付きの`ClusterMessage`（`schema_version` / `message_id` / `sender` / `receiver` / `message_type` / `checksum`） |
| `runtime.rs` | コーディネータ本体（`run_cluster`）。ワーカー登録・タスク割当・失敗検知・再割当て・タイムアウト判定を実装 |
| `diagnostics.rs` | 構造化された診断コード（例: `CRR-RUN-003` `CRR-RUN-004` `CRR-RUN-005`） |
| `dynamic/` | Dynamic ReasonUnit向けの拡張（ライフサイクル管理・分岐の枝刈り・予算ベースの収束制御） |

メッセージプロトコルは `worker_register → worker_ready → heartbeat`（登録時のハンドシェイク）→ `task_assign → task_start → (task_failure | 完了)` という流れで、すべてのメッセージが `ClusterMessage` としてログ（`cluster_messages.jsonl`）に記録される。

## 5. Design Decisions

- **ワーカー障害時の再割当て**：`runtime.rs`の`run_cluster`は、タスク失敗（`task_failure`, reason: `worker_unavailable`）を検知すると、失敗したワーカーを`unavailable`に遷移させ、`config.execution.max_retries`の範囲内で**別のワーカーへ再割当て**する（`attempt:2`のtask_assignを再送）。上限を超えた場合は診断`CRR-RUN-004: Worker retry limit exceeded`を出す
- **タイムアウトの明示的な実装**：`started.elapsed() >= Duration::from_millis(timeout_ms)`を判定し、超過時に`CRR-RUN-003: Task timeout`を返す関数が独立して実装されている
- **障害シミュレーションをテスト設定として一級化**：`TestingConfig.fail_task_once`により、特定タスクIDを意図的に1回失敗させることができ、retry経路をプロダクションコードと同じパスでテストできる
- **予算による収束制御（Dynamic ReasonUnit）**：`dynamic/runtime.rs`は、メッセージ数・ユニット数・logical step数に上限（budget）を設け、超過時は`budget_terminated`として明示的に打ち切る。ブランチ（分岐）は「dominated」と判定されたものが枝刈り（pruning）される

## 6. Implementation

- タスクのライフカイクル（`pending → ready → assigned → running → completed/failed`）を状態遷移として管理し、不正な遷移は`transition()`関数で診断として記録
- Dynamic ReasonUnitは9段階のライフサイクル（`proposed → validated → registered → ready → assigned → running → waiting/completed/failed/suspended → retired/replaced/cancelled`）を持ち、`suspended → ready`の再活性化には`max_reactivations`の上限があり、超過時は`DRU-LFC-003: reactivation limit exceeded`
- CLIアダプタ（`toolchain/cluster_runtime_cmd.py`、`reason cluster` サブコマンド）から、計画（plan）・実行（execute）・シミュレーション（simulate）・検証（verify）・比較（compare）を呼び出せる

## 7. Testing

Rust側テスト（`ClusterRuntime/tests/`）で確認できる主なアサーション：

- `plans[0].partitions[1].fallback_used` — フォールバック割当ての検証
- `assert!(!allowed(Some("retired"), "ready"))` 等 — ライフサイクルの不正遷移が拒否されることの検証
- `assert_eq!(result.summary["status"], "completed")` / `"budget_terminated"` — 正常完了と予算超過打ち切りの両方を検証

Python側テスト（`tests/cluster_runtime/test_cluster_runtime_rust.py`）で確認できる主なテスト関数：

- `test_rust_cluster_runtime_crate_passes` — RustクレートのテストスイートをPython側のテストとして実行
- `test_cluster_worker_executes_tensor_computation_through_rust_host` — ワーカーが実際にRustランタイムホストを介してテンソル計算を実行することを検証
- `test_local_process_workers_and_all_artifacts` — 複数ワーカープロセスでの実行と、全成果物ファイルの生成を検証
- `test_dynamic_reason_unit_scenarios` — Dynamic ReasonUnitの13の受け入れシナリオ

これらはReasonScript全体の`./reason ci`スイート（1,240テスト、2026-08-31実行してPASS確認済み）に含まれる。

## 8. Problems Found

（本ポートフォリオ作成時点で、ClusterRuntime固有の障害修正コミットの履歴までは追跡していない）

## 9. Root Cause

（該当なし）

## 10. Fix / Redesign

（該当なし。ただし`dynamic_reason_unit_cluster_execution_v0_1.md`によれば、Dynamic ReasonUnit拡張は既存の静的Cluster Runtimeの挙動を変更しない、という互換性方針のもとで追加された）

## 11. Verification

`ClusterRuntime/`のソースコードと`ClusterRuntime/tests/`・`tests/cluster_runtime/`のテストコードを2026-09-08に直接調査し、retry・timeout・heartbeatメッセージ・ライフサイクル管理・予算ベース収束制御の実装とテストの存在を確認した。

## 12. Result

- コーディネータ/ワーカー間のメッセージ交換、失敗時の再割当て、タイムアウト、リソース予算による打ち切りが、いずれも実装とテストの両方で確認できる
- 障害注入（`fail_task_once`）による再現可能なretryテストという、テスト容易性を意識した設計になっている

## 13. Known Limitations

- **確認できたのは複数プロセス（`local_process`モード）での協調実行**であり、複数マシン・ネットワーク越しの分散実行が実装・検証されているかは本ポートフォリオ作成時点で確認できていない
- heartbeatは**ワーカー登録時のハンドシェイクメッセージ**として存在し、`heartbeat_interval_ms`/`heartbeat_miss_limit`という設定項目もあるが、**この間隔・上限に基づいて継続的にワーカーの生存を監視するループ**は、調査した範囲のコードでは確認できなかった。周期的な死活監視として主張はしない
- 実際のプロダクション運用（長時間・高負荷下での挙動）についての実測データはない

---

→ [ReasonScript Case Study](reasonscript.md) · [Persistent Graph Runtime Case Study](persistent-graph-runtime.md) · [Evidence Index](../evidence/evidence-index.md)
