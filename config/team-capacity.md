# Team Capacity Config（スナップショット）

> このファイルは ClickUp Doc `3ed22-86858`
> https://app.clickup.com/3617858/docs/3ed22-86858
> のミラーです。**正本は ClickUp Doc。** Routine が参照するのは ClickUp 側。
> このリポジトリ内のコピーは履歴管理・レビュー用。

最終確認日: 2026-05-12

## メンバー別 実働可能時間 ＋ メールアドレス対応表

| メンバー | ClickUp メール | Slack メール | 1日稼働 | 役割 |
|---|---|---|---|---|
| Hiroko | overthenoon@gmail.com | hiroko@wasimil.com | 3.0h | CEO |
| Jun Nakatani | jun.nakatani.work@gmail.com | jun@wasimil.com | 6.0h | PR/Marketing |
| Chihiro Kanbe | chihiro@wasimil.com | chihiro@wasimil.com | 6.0h | Sales |
| Asakura Misaki | misaki@wasimil.com | misaki@wasimil.com | 6.0h | CS/Onboarding |
| Kaori Yamada | yamadance66@gmail.com | kaori@wasimil.com | 6.0h | CS |
| Sho Ichie | sho@wasimil.com | sho@wasimil.com | 5.0h | 研修期間中 |
| Junko Matsuyama | junko@wasimil.com | junko@wasimil.com | 5.0h | 経理 |
| Miki Omagari | miki@wasimil.com | miki@wasimil.com | 6.0h | Web/DMO |
| Ogawa Kanako | kanako@wasimil.com | kanako@wasimil.com | 6.0h | HR |
| Marie Kadoya | marie@wasimil.com | marie@wasimil.com | 5.0h | HR |
| 神前元紀 | motonori.kozaki@gmail.com | motonori@wasimil.com | 6.0h | Sales/Admin |

## クリーンアップ事項

- `sho@waasimil.com` (ID: 113492889) はタイポ・休眠アカウント → ClickUp Admin で削除推奨
- 正規アカウント: `sho@wasimil.com` (ID: 113492890)

## 除外メンバー（Capacity 管理対象外）

- 外部パートナー（CHM・future.co.jp などのドメイン）
- インドネシア・フィリピン拠点メンバー
- 退職者・休眠アカウント

## 週次キャパシティ計算式

```
週間キャパ      = 1日稼働時間 × 5日
スプリントキャパ = 1日稼働時間 × スプリント営業日数
稼働率          = タスクEstimate合計 / キャパ × 100
```

判定基準:
- 90% 以上: 過負荷リスク 🔴
- 70〜89%: 適正 🟢
- 70% 未満: 余力あり 🟡
