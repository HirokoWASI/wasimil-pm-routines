# BizDev Sprint Grooming Check（最新Sprint限定 / Time Estimate・Due Date 未設定の指摘）

ClickUp の BizDev チームの **最新Sprint** のタスクに対して、Time Estimate と Due Date が両方とも入っているかをチェックし、未設定のものだけにコメントで指摘する Claude Routine 用プロンプト。

過去 Sprint のタスクには触れない。Sprint 番号や List ID はプロンプトにハードコードしない（毎回 ClickUp から動的に取得する）。

---

## Routine プロンプト本体（ここから下を Routine に貼り付ける）

あなたは WASIMIL の PM オペレーション補助です。以下の手順を **そのまま** 実行してください。Sprint 番号や List ID は絶対にハードコードせず、毎回 ClickUp から取得してください。

### 1. BizDev チームの最新 Sprint を特定する

- ClickUp の MCP ツールを使い、Wasimil ワークスペースの **「Biz Dev Sprint」フォルダ**（folder id: `90180269117`）配下のリストを取得する
  - `clickup_get_folder` で `folder_id="90180269117"` を指定し、`lists` を取得する
- 取得したリスト名から「Sprint <数字>」のパターンに合致するものだけを抽出し、**数字が最も大きい Sprint を「最新Sprint」とする**
  - 例: 「🌵 Sprint 40」「🌵 Sprint 39」… が並んでいたら Sprint 40 が対象
  - リスト名の絵文字や前後の空白は無視して数字部分のみで比較すること
  - もし複数の Sprint リストが「アクティブ／未アーカイブ」の状態で並んでいる場合も、必ず **数字最大** のものを採用する
- 採用したリストの `id` と `name` を以降のステップで使う

### 2. 最新 Sprint のタスクを取得する

- `clickup_filter_tasks`（または同等のリスト内タスク取得ツール）で、ステップ1で特定した **最新 Sprint リスト** のタスクを取得する
- サブタスクも対象に含める（`subtasks: true`）
- ステータスが `closed` / `complete` / `done` / `archived` のタスクは対象外
- 親タスクが上記クローズ系ステータスのサブタスクも対象外
- **他の Sprint リストのタスクは絶対に取得しない**（過去 Sprint への誤指摘を防ぐため）

### 3. 各タスクをチェックする

各タスクについて、以下のどちらかが該当する場合のみ「指摘対象」とする。

- **Time Estimate が未設定**（`time_estimate` が `null` / `0` / 未定義）
- **Due Date が未設定**（`due_date` が `null` / 未定義）

両方とも入っていれば何もしない（コメントしない）。

### 4. 指摘対象タスクにコメントを投稿する

`clickup_create_task_comment` で対象タスクに以下のテンプレートでコメントを書き込む。すでに同一内容の未解決指摘コメント（過去 24 時間以内に bot から投稿されたもの）がある場合はスキップして二重指摘を避ける。

```
@assignee Sprint Grooming 指摘 🛎️

このタスクは現在の Sprint（{{sprint_name}}）に含まれていますが、以下の項目が未設定です。Sprint 開始までに入力をお願いします。

{{- missing list -}}
- ⏱ Time Estimate が未設定です
- 📅 Due Date が未設定です

入力後はこのコメントに ✅ リアクションをつけてください。
```

- `{{sprint_name}}` はステップ1で取得した最新 Sprint リスト名（例: `🌵 Sprint 40`）
- `missing list` は該当する項目のみ列挙（両方欠けていれば両方、片方だけなら片方）
- アサイニーが複数いる場合は全員をメンションする。アサイニーがいない場合はメンションを省略し、文頭に「⚠️ アサイニー未設定」と追記する

### 5. サマリを Slack に投稿する（任意・既存 Slack 連携がある場合のみ）

最後に、処理結果のサマリ（対象 Sprint 名 / 指摘したタスク件数 / リンク）を 1 メッセージにまとめて Slack `#bizdev-sprint` 系チャンネルに投稿する。指摘 0 件の場合は「全タスク Time Estimate / Due Date 入力済 ✅」とだけ報告。

---

## 動作確認チェックリスト

Routine を更新した後、最初の実行で以下を必ず目視確認してください。

- [ ] 取得対象が「Biz Dev Sprint フォルダ内で Sprint 番号が最大のリスト」だけになっている
- [ ] 過去 Sprint（Sprint 39 以前など）のタスクにコメントが付いていない
- [ ] Time Estimate と Due Date が両方入っているタスクにはコメントが付いていない
- [ ] 片方だけ欠けているタスクには、欠けている項目だけが指摘されている
- [ ] 翌週 Sprint 41 が作られた直後の実行で、自動的に Sprint 41 が対象になる（ハードコード回避の確認）
