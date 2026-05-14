# VC月次報告 Notion自動投稿ルーティン（完成版 / Routine 貼り付け用）

毎月 5 日 09:00 JST にリモートクラウド環境で自動実行される、AZOO（WASIMIL）VC 月次報告の Notion ページ生成 + Slack 通知ルーティン。

**このリポジトリの位置づけ：** Routine のプロンプト本体をバージョン管理する場所。Claude Code の Routine 設定（`trig_01UA4wahnz5ZPY5jC9rNyHuu`）に貼り付ける本文は、下記「Routine プロンプト本体」セクション以下をそのままコピーする。

ClickUp プロダクト集計部分（STEP 5）は、`prompts/product-monthly-update-for-vc.md` で定義したロジックを統合済み。

---

## Routine プロンプト本体（ここから下を Routine に貼り付ける）

これはリモートクラウド環境で実行されます。ローカルスクリプト（`python3 gw.py` 等）は使えません。**Notion / Supabase / ClickUp / Slack / Google-Drive / Firecrawl MCP のみ**使用してください。

### 前提情報

- Notion VC Monthly Meeting database `data_source_id`: `deb5f01e-7d35-8285-8ca2-075d87700da9`
- 財務ダッシュボード URL: `https://wasi-financial-dashboard.vercel.app/embed/vc-report?key=azoo-vc-2026&y={YEAR}&m={MONTH}&lang=ja&theme=light`
- Slack 通知先: `washimo.slack.com` チャンネル ID `C06U8SWQT0Q`
- Sales Pipeline Supabase Project ID: `bnqejljedwkmlkykdhcr`
- ATS Supabase Project ID: `jrzrdcxbuypumgukvlok`
- freee 会計 Google Sheets ID: `1dZqJ1ijbbJxXwk9DjRsJKa047BrNUHBPMm5Bip7ib_E`
  - シート名: 「ダッシュボード | Metrics」、「月次商談・リード状況表」
- ClickUp Product フォルダ ID: `3747129`
- ClickUp All Tasks フォルダ ID: `19531798`

### STEP 1: 報告対象月の計算

「N 月の VC 報告 = N-1 月に実行した活動の記録」

- 例: 実行日が 2026/5/5 → 報告対象月 = **2026 年 4 月**
- レポートタイトル: `April 2026 VC - AZOO月報`
- 集計期間（JST = `Asia/Tokyo`）:
  - 開始: 報告対象月 1 日 00:00:00 JST
  - 終了: 報告対象月 末日 23:59:59 JST
- 月初 1 日 / 末日 / 翌月 1 日のエポックミリ秒（JST）を変数として保持し、以降の各 STEP で使い回す

### STEP 2: Financial Analysis

Google-Drive MCP で Sheets ID `1dZqJ1ijbbJxXwk9DjRsJKa047BrNUHBPMm5Bip7ib_E` の以下範囲を読み取り、KPI を抽出する。

- `ダッシュボード | Metrics!A1:Q20`
- `月次商談・リード状況表!A1:Z20`

抽出 KPI:

- MRR / Committed ARR / 施設数 / ARPA / 解約率

財務ダッシュボード URL を報告対象月で生成（`{YEAR}`/`{MONTH}` を埋める。`{MONTH}` はゼロ埋めなし）。

### STEP 3: Sales Pipeline (Supabase project: `bnqejljedwkmlkykdhcr`)

以下 SQL を Supabase MCP の `execute_sql` で実行（日付は JST ベース、上で計算した境界値を使用）:

- 当月の新規 deals 数（ステージ別）
- 当月の成約 deals 詳細
- パイプライン現況（全アクティブ商談）
- 当月の新規顧客
- 顧客全体サマリ
- 前月比較用（先々月の新規 deals 数）

### STEP 4: ATS (Supabase project: `jrzrdcxbuypumgukvlok`)

以下 SQL を実行:

- 当月の新規候補者数（ソース別）
- 当月の候補者ステータス別集計
- 当月の候補者 × 求人別
- オープン求人一覧
- 当月の採用実績（`hiring_records`）
- 選考パイプライン全体
- 前月比較用

### STEP 5: ClickUp Product 集計（フォルダ ID `3747129` と `19531798`）

#### 5-1. 対象リストの動的解決（ハードコード禁止）

- `clickup_get_folder(folder_id="3747129")` → Product フォルダ配下のリストを **すべて対象に含める**
  - 例: `🔎 User Feedback` / `🐞 Functional QA (BUGS)` / `🐞 Sentry QA` / `Mark` / `Swati` / `Nath` / `Nursandy`
- `clickup_get_folder(folder_id="19531798")` → All Tasks フォルダ配下のリストのうち、**名前が `🎯Sprint` で始まるリストのみ**対象に含める
  - `📝Sprint XXX Backlog` は **除外**（未着手バックログのため）
  - 旧アーカイブ系フォルダ（Sprints 1-84 / Sprints 90+ / Design）は別フォルダ ID なので元から対象外
- 採用したリスト ID 配列を `target_list_ids` として保持

#### 5-2. 前月完了タスクの取得

`target_list_ids` の各リストに対して `clickup_filter_tasks` を実行:

- `include_closed: true`（クローズドステータスを含む）
- `subtasks: false`（仕様どおりサブタスクは含めない）
- `date_done_gt = 報告対象月 1日 00:00:00 JST のエポックミリ秒 - 1`
- `date_done_lt = 報告対象月 末日 23:59:59 JST のエポックミリ秒 + 1`
- `order_by: "updated"`
- ステータスが `closed` / `complete` / `done` / `archived` / `resolved` のもののみ残す
- ページネーション: `page=0` から順に、`last_page` または件数 0 になるまで全件取得
- 取得後、ローカルでも `date_done` が報告対象月内であることを再フィルタ（API のフィルタが緩い場合の保険）
- 全タスクをひとつの配列にまとめ、**`id` で重複排除**

#### 5-3. カテゴリ分類

上から優先で 1 タスクに 1 カテゴリを割り当てる:

1. **🚀 新機能・機能改善** — `🎯Sprint` リスト配下のタスク、または `tags` に `feature` / `enhancement` / `release` を含む、または名前先頭が `[feat]` `[feature]` `[新機能]`
2. **🛠 インフラ・技術改善（リファクタ含む）** — `tags` に `improvement` / `refactor` / `chore` / `tech-debt` / `infra` を含む、または名前先頭が `[improve]` `[refactor]` `[chore]` `[infra]`
3. **🐞 バグ修正** — リストが `🐞 Functional QA (BUGS)` または `🐞 Sentry QA`、もしくは `tags` に `bug` を含む
4. **🔬 QA・テスト** — `tags` に `qa` / `test` を含む、または名前先頭が `[qa]` `[test]`
5. **🔎 ユーザーフィードバック対応** — リストが `🔎 User Feedback`、または `tags` に `user-feedback` / `customer-request`
6. **その他（個別メンバーリストの単発タスク等）** — 上記いずれにも該当しないもの

タグやプレフィックスが無くても、`🎯Sprint` 配下のタスクは原則「新機能・機能改善」または「インフラ・技術改善」に寄せる（迷ったら 1 へ）。

#### 5-4. 集計と整形

- **カテゴリ別**: 各カテゴリの件数 + 主要タスク名を箱条書きで **最大 10 件**。各行末に ClickUp タスク URL を `[link]` として付記
- **担当者別**: 完了タスク数の上位を集計
- **Sprint 番号別**: タスクのリスト名から Sprint 番号を正規表現で抽出（`/(?:Sprint|🎯Sprint)\s*(\d+)/` 等）し、Sprint 別の件数も集計
- **ハイライト**: 「事業インパクトのある進捗」を 1〜3 行で要約（VC が知りたい内容を優先）

### STEP 6: Hotel Tech 動向（Firecrawl MCP）

`firecrawl_search` で以下 3 クエリを実行:

1. `hotel technology trends 2026`
2. `hospitality tech news latest`
3. `hotel PMS innovation AI 2026`

3〜5 件に要約。各項目 1〜2 文 + ソース URL を付記。WASIMIL 事業（PMS / 宿泊 DX / Cosmos / Keylock 等）に関連度が高いものを優先。

**直接引用は 15 語以内に厳守。** それ以外は自分の言葉で要約する。

### STEP 7: Notion ページ作成

`notion-create-pages` で以下属性のページを作成:

- parent `data_source_id`: `deb5f01e-7d35-8285-8ca2-075d87700da9`
- Name: `{Month英語} {Year} VC - AZOO月報`（例: `April 2026 VC - AZOO月報`）
- `date.Date.start`: 報告対象月の **翌月の第 2 木曜日**（`YYYY-MM-DD`）
  - 例: 報告対象月 = 2026/4 → 翌月 = 2026/5 → 第 2 木曜日 = 2026/5/14
- icon: 🦅

ページ内容は以下セクション構成:

- `01_サマリー`
- `02_財務状況`（STEP 2 の KPI + 財務ダッシュボード URL）
- `03_セールス活動`（STEP 3 の集計）
- `04_採用活動`（STEP 4 の集計）
- `05_Hotel Tech動向`（STEP 6）
- `06_プロダクト`（STEP 5：カテゴリ別 + 担当者別 + Sprint 別 + ハイライト）
- `07_課題と対策`（手動記入プレースホルダ）
- `08_次月アクション`（手動記入プレースホルダ）
- `09_ディスカッションポイント`（手動記入プレースホルダ）

Notion ブロックは見出し（heading_2）/ 箇条書き（bulleted_list_item）/ 段落（paragraph）/ コールアウト（callout）を適切に組み合わせる。

### STEP 8: Slack 通知

Notion ページ作成完了後、`slack_send_message` でチャンネル `C06U8SWQT0Q` にサマリと Notion ページ URL を投稿:

- Financial Analysis（MRR / ARR / 施設数）
- Sales Pipeline（新規商談 / 成約 / パイプライン MRR）
- ATS（新規応募 / 採用決定 / オープン求人）
- **Product（完了タスク総数 + カテゴリ別件数 🚀{N1} / 🛠{N2} / 🐞{N3} / 🔬{N4} / 🔎{N5} + 主要リリース 1〜3 件）**
- Hotel Tech（トップ 1〜2 トピック）
- 手動記入項目リスト（07/08/09 セクション）
- Notion ページ URL

### 注意事項

- 金額は `¥` 付きカンマ区切り（例: `¥1,234,567`）
- Supabase の SQL 日付フィルタは **JST ベースで計算（+9 時間オフセット考慮）**
- ClickUp タスク取得はページ 0 から順に取得し、最終ページまで必ず辿る
- API エラーはスキップせず Slack に通知（どの STEP で失敗したかを明記）
- Firecrawl 検索結果は要約のみ（直接引用は 15 語以内）
- Sprint 番号・List ID・Notion ページ ID・Slack チャンネル名はハードコードしない（**ID は前提情報セクションの定数のみ可**）
- 旧アーカイブ Sprint フォルダ（Sprints 1-84 / Sprints 90+）のタスクが混入しないこと

---

## 動作確認チェックリスト（最初の実行で目視確認）

- [ ] STEP 1: 報告対象月が「実行日の前月」になっている
- [ ] STEP 2: 財務ダッシュボード URL の `{YEAR}` / `{MONTH}` が正しく埋まっている
- [ ] STEP 3/4: SQL の日付境界が JST で計算されている
- [ ] STEP 5: All Tasks フォルダの `🎯Sprint XX` は対象、`📝Sprint XXX Backlog` は除外
- [ ] STEP 5: Product フォルダ配下（User Feedback / QA / 個人リスト）が対象に含まれている
- [ ] STEP 5: 同タスクが重複カウントされていない（id で dedupe）
- [ ] STEP 5: カテゴリ分類が優先順位どおりに 1 カテゴリ割当になっている
- [ ] STEP 6: Firecrawl 結果の直接引用が 15 語以内
- [ ] STEP 7: Notion ページタイトル / アイコン / 日付（翌月第 2 木曜日）が正しい
- [ ] STEP 7: 9 セクションすべて存在し、06_プロダクトに STEP 5 の内容が反映されている
- [ ] STEP 8: Slack 投稿に Product カテゴリ別件数と Notion URL が含まれている
- [ ] エラー時に Slack へ STEP 名付きで通知される
