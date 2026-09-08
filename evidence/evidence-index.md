# Evidence Index

このポートフォリオ全体の主張を、Claim（主張）/ Evidence（根拠）/ Status（状態）の3列に分解した一覧です。
**主張の強さを実装量ではなく検証の強さで区別する**ことを目的としています。

## Status語彙

| Status | 意味 |
|---|---|
| **VALIDATED** | 実行結果・実測データによって検証済み |
| **TESTED** | テストが存在し、実行して結果を確認している |
| **IMPLEMENTED** | 実装され、動作することを直接確認しているが、体系的なテストは伴わない |
| **PROTOTYPE** | 動く部分はあるが未完成・未検証の範囲が大きい、または開発が停止している |
| **EXPERIMENTAL** | 限定的な条件下での設計確認のみ。性能や汎用性は未測定 |
| **DESIGN** | 仕様・設計として定式化されているが、実装はこれから、または部分的 |
| **RESEARCH** | 仮説検証段階。立証も反証もされていない |
| **CONCEPT** | 構想段階。実装に着手していない |

---

## Backend / Systems Engineering

| Claim | Evidence | Status |
|---|---|---|
| 決定論的コンパイラ・ランタイムを設計・実装できる | ReasonScript（[Case Study](../case-studies/reasonscript.md)）— `./reason ci` 9ステージ・1,240テスト、2026-08-31実行してPASS確認 | **VALIDATED** |
| クロス言語のAPI/データ契約を設計できる | Common DTO Specification（Rust/Python/TypeScript/Go/Java 5言語） | **IMPLEMENTED** |
| 構造レベルの永続化・トランザクションモデルを設計できる | VisionWorldModel（[Case Study](../case-studies/persistent-graph-runtime.md)）— Phase 3C-1まで検証コマンド・成果物あり | **VALIDATED** |
| ディスクレベルの耐久性（journal/fsync等）を実装できる | — | **未検証（主張していない）** |
| worker調整・retry・timeoutを伴う分散実行を実装できる | ReasonScript `ClusterRuntime`（[Case Study](../case-studies/cluster-runtime.md)）— ソースコード・単体テストで直接確認（2026-09-08調査）。retry/timeout/failure-injectionテストは`./reason ci`スイートに含まれる | **TESTED** |
| 複数マシンにまたがるネットワーク分散実行ができる | — | **未検証（確認できたのは単一マシン上の複数プロセス協調のみ）** |
| 実務ドメインに接続したREST API・DB設計ができる | KuKKA ClassRoom_WebPage（[Backend Project](../projects/backend-service.md)）— Prisma 5モデル、4回のマイグレーション、5APIルート | **PROTOTYPE**（開発一時停止中） |
| 認証・認可を伴うセキュアなAPIを実装できる | — | **未検証（KuKKAには認証なし。Gapとして明記）** |
| 障害を構造的に診断し、根本原因に基づいて設計を修正できる | [debugging-and-failure-analysis.md](../case-studies/debugging-and-failure-analysis.md)（3件） | **VALIDATED**（診断・修正・再検証の記録あり） |
| CI/テスト基盤を設計・運用できる | ReasonScript CI（9ステージ）、Conformance framework、Golden corpus | **VALIDATED** |

## Advanced R&D

| Claim | Evidence | Status |
|---|---|---|
| 単語・多言語の想起 | COHERENT — 60語で100%、劣化率0.00%（実測共鳴値つきCSVあり） | **VALIDATED** |
| 数式の正誤・同値判定 | COHERENT — 10ケース全PASS | **TESTED** |
| 三値判定（Accept/Review/Reject）の成立 | COHERENT — `MemorySpace`として実装・運用 | **IMPLEMENTED** |
| 記憶再利用による計算資源の効率化 | COHERENT — 5件の固定シナリオによる設計確認のみ、共鳴スコアはテスト内定数 | **EXPERIMENTAL**（性能は未測定） |
| 文字生成（カタカナ・漢字） | COHERENT — 100%/60〜80%だが生成スクリプト未収録 | **再現不可** |
| Truth Boundary（連想記憶と正規知識の分離）の定式化 | LanguageModel 仕様書 v0.1 §2.1 | **DESIGN** |
| MRA Holographic Semantic Memoryの実装 | LanguageModel — Phase 0（基盤固定・仕様策定）完了、実装約450行 | **DESIGN**（実装はこれから） |
| AIコーディングエージェントの権限分離設計 | Design_BrainModel Agent Operational Charter | **IMPLEMENTED** |
| コーディングエージェントの中核推論機構 | Design_BrainModel v1 — 推論爆発・学習能力不足・容量問題により未完成と自己評価 | **PROTOTYPE**（v1は未完成、v2再設計予定） |

---

## この一覧の読み方

- 同じ「実装した」でも、VALIDATEDとPROTOTYPEでは意味が異なります。表の右列を必ず確認してください
- 「未検証（主張していない）」の項目は、設計案が例示した内容の一部に対応しますが、既存資料では裏付けが取れなかったため、あえて主張していません
- 個別の実測データ・一次データの詳細は各Case Study・[docs/projects/](../docs/projects/)配下のページを参照してください

→ [README](../README.md) · [Backend Engineering Evidence](../backend-engineering/overview.md) · [Advanced R&D](../advanced-rd/overview.md)
