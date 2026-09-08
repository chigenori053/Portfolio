# Backend Project: KuKKA — プログラミング教室 予約・運営システム

> 研究プロジェクトではなく、**実在する子供向けプログラミング教室「KuKKA」のための実務システム**です。
> ブランドサイトと、体験教室の予約・スケジュール管理・問い合わせ対応を行う管理画面から構成されています。
> 2026年1月に着手し、3月中旬を最後に開発が一時停止しています。未完成であることを前提に、
> 実際に確認できる実装範囲と、確認できていない欠落点の両方を記載します。

| | |
|---|---|
| **リポジトリ** | https://github.com/chigenori053/ClassRoom_WebPage |
| **開始 〜 最終コミット** | 2026-01-27 〜 2026-03-14（以降、開発停止中） |
| **技術スタック** | Next.js 16（App Router）· React 19 · TypeScript · Prisma ORM · PostgreSQL |
| **構成** | 公開サイト（`/`）と管理者用サブアプリ（`admin/`）の2つのNext.jsアプリ |
| **Status** | **PROTOTYPE（開発一時停止中）** |
| **ライセンス** | 実務プロジェクトのため非公開ライセンス方針（コードは閲覧・評価目的で公開） |

---

## 1. Problem

プログラミング教室の運営には、体験教室の予約受付、既存スケジュールとの重複防止、保護者からの問い合わせ管理、講師向けのカレンダー確認という、複数の実務ワークフローが絡む。これらを手作業（電話・メール・紙のカレンダー）で回すのではなく、Webサイトと連動した最小限の運営システムとして構築する。

## 2. Requirements

- 保護者が体験教室の空き枠を確認し、Web上で予約できること
- 予約が特定の枠（TrialSlot）に対して一意に紐づき、二重予約を防げること
- 問い合わせフォームからの入力を管理側で一覧・状態管理できること
- 講師側がスケジュールをカレンダー形式で確認・編集できること
- 既存のGoogle Calendar運用と連携できること

## 3. Constraints

- サイトの主眼はブランド訴求（教育方針の言語化）であり、機能を前面に出さない設計方針が要求されていた（`docs/Design_note.md`）
- 個人開発のため、インフラ・運用コストを抑える必要があった（Vercelデプロイ前提のNext.js構成）

## 4. Architecture

公開サイトと管理サイトを別々のNext.jsアプリとして分離し、共通のPostgreSQLデータベースをPrisma経由で参照する構成。

```
公開サイト (src/)                管理サイト (admin/)
  courses / booking / contact      dashboard / bookings / schedules
  column（ブログ）/ access          inquiries / columns（CRUD）/ calendar
        │                                  │
        └──────────► PostgreSQL (Prisma) ◄─┘
                          │
                   Google Apps Script 経由で
                   Google Calendar と同期
```

### データモデル（Prismaスキーマ）

| モデル | 役割 |
|---|---|
| `Article` | ブログ/コラム記事（下書き/公開のステータス管理） |
| `Schedule` | 教室のスケジュール（Google Calendarイベントと`gasEventId`で紐付け） |
| `TrialSlot` | 体験教室の予約可能枠（OPEN/FULL/CLOSED） |
| `Inquiry` | 問い合わせ（NEW/REPLIED/CLOSED） |
| `Booking` | 予約。`TrialSlot`と1:1のユニーク外部キーで二重予約を防止 |

## 5. Design Decisions

- **予約と枠を1:1のユニーク制約で結合**：`Booking.trialSlotId` を `@unique` にすることで、1つの枠に対して2件以上の予約が成立しない制約をDBレベルで保証している
- **外部カレンダーとの疎結合**：Google Calendarとの同期を、アプリのコアロジックに直接埋め込まず `lib/gas.ts`（Google Apps Script経由のブリッジ）として分離し、`gasEventId` で紐付ける設計
- **公開サイトと管理サイトのアプリ分離**：権限の異なる2つの利用者層（保護者/教室運営者）に対して、Next.jsアプリそのものを分けることでルーティングと関心を分離

## 6. Implementation

- APIルート: `/api/bookings`（予約作成）、`/api/contact`（問い合わせ）、`/api/trial-slots`（枠一覧）、`/api/trial-slots/[id]`（admin: 枠の個別操作）、`/api/calendar-events`（admin: カレンダー連携）
- 管理画面: ダッシュボード、予約一覧、スケジュールのカレンダービュー（`CalendarView.tsx`）、問い合わせ一覧、コラムのCRUD（新規作成・編集・削除）
- Prismaマイグレーション: `init` → `add_trial_slot` → `add_inquiry` の順で、要件の追加に合わせてスキーマを段階的に拡張した記録が残っている

## 7. Testing

**自動テストは実装されていない。** リポジトリ内にテストファイル・テストランナーの設定は確認できなかった。

## 8. Problems Found

開発時点で顕在化した障害の記録（issue、バグ修正コミット等）は確認できていない。3月14日の最終コミット「WebPage更新」以降、開発が停止しており、未解決の不具合が残っている可能性はあるが特定できていない。

## 9. Root Cause

（該当なし——障害分析ではなく、開発中断の理由は本ポートフォリオ作成時点で本人以外からは確認できない）

## 10. Fix / Redesign

（該当なし）

## 11. Verification

GitHub上のコミット履歴・ディレクトリ構成・Prismaスキーマ・APIルートの実在を直接確認した。実際にデプロイされた状態でのE2E動作確認は本ポートフォリオ作成時点では行っていない。

## 12. Result

- 実在する教室ビジネスの要件（予約・重複防止・問い合わせ管理・外部カレンダー連携）から出発した、小規模だが実務的なREST API設計とリレーショナルなデータモデリングの実例
- Prismaマイグレーション履歴が、要件追加に応じてスキーマを進化させたプロセスの記録として残っている

## 13. Known Limitations

- **認証・認可が実装されていない。** 管理画面（`admin/`）へのアクセス制御の仕組みはリポジトリ内で確認できなかった
- 自動テスト・CI/CDパイプライン・Dockerによるコンテナ化は行われていない
- トップレベルの `README.md` は `create-next-app` のボイラープレートのままで、プロジェクト固有の説明が整備されていない
- 2026-03-14を最後に開発が停止しており、以降の要件変更・不具合修正は反映されていない

---

→ [Backend Engineering Evidence](../backend-engineering/overview.md) · [Evidence Index](../evidence/evidence-index.md)
