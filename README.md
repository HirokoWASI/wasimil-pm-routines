# wasimil-pm-routines

WASIMIL チームのプロジェクト管理を自動化する Claude Code Routines のコード化リポジトリです。
実装ガイド v2.0（2026-05-12）に基づく構成です。

## 構成

```
.
├── README.md
├── config/
│   └── team-capacity.md      # ClickUp Doc (3ed22-86858) のスナップショット
├── routines/
│   ├── 1-daily-task-coach.md            # 毎朝 09:00 / 平日（コーチ + AI 妥当性レビュー）
│   ├── 2-weekly-performance.md          # 月曜 09:00
│   ├── 3-sprint-planning.md             # API trigger（手動）
│   ├── 4-friday-capacity-warning.md     # 金曜 16:00
│   ├── 5-estimate-accuracy-feedback.md  # 平日 18:00 / Hubstaff 活用
│   ├── 6-overrun-alert.md               # 平日 14:00 / Hubstaff 活用
│   └── 7-morning-task-briefing.md       # 平日 08:00 / 本日のタスク Daily Plan
└── docs/
    └── rollout-checklist.md        # 段階的ロールアウト＆実装チェックリスト
```

## 運用フロー

| Routine | トリガー | 出力先 | 目的 |
|---|---|---|---|
| 1. 毎朝のタスク整備コーチ＋AIレビュー | Weekdays 09:00 JST | 各担当者へSlack DM + Hiroko DM | Due Date / Estimate 入力促進＆妥当性チェック |
| 2. 週次パフォーマンスレポート | Monday 09:00 | `#biz-dev` | チーム可視化 |
| 3. スプリントプランニング AI | API trigger | `#biz-dev` | 工数推定＆負荷バランス（Hubstaff 実績ベース） |
| 4. 金曜キャパシティ警告 | Friday 16:00 | Hiroko DM | 過負荷の事前検知 |
| 5. 見積精度フィードバックループ | Weekdays 18:00 | ClickUp コメント + Hiroko DM | Routine 3 推定モデルの精度向上 |
| 6. 進行中タスクの工数超過アラート | Weekdays 14:00 | 担当者 DM + Hiroko DM | スコープ崩壊の当日中検知 |
| 7. 本日のタスクブリーフィング | Weekdays 08:00 | 各担当者へ Slack DM | 朝一の認知負荷低減、着手順の迷い解消 |

すべての Routine は `claude-sonnet-4-5` で動作し、ClickUp + Slack コネクターを使用します。
Routine 5・6 は Hubstaff → ClickUp 連携で同期される `time_spent` フィールドを活用します。

## 実装手順

1. `claude.ai/code/routines` を開く
2. "New Routine" → `routines/<番号>-<名前>.md` の **プロンプト** セクションをコピー&ペースト
3. **設定** セクションの値（モデル/トリガー/コネクター）をUI上で入力
4. "Run Now" でテスト
5. 本番運用に移行

詳細は `docs/rollout-checklist.md` を参照。

## 重要 ID

| 項目 | ID / URL |
|---|---|
| ClickUp Workspace | 3617858 |
| Team Capacity Config Doc | 3ed22-86858 |
| Team Capacity Config URL | https://app.clickup.com/3617858/docs/3ed22-86858 |
| Slack `#biz-dev` Channel | C06EJ9VKT45 |
| Sprint 40 List | 901817938451 |
