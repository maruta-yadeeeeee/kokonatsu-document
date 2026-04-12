---
id: points-punishment
sidebar_position: 9
title: ポイント・処罰システム
sidebar_label: ポイント・処罰
---

# ポイント・処罰システム

guardian は違反ごとにポイントを蓄積し、設定した閾値に達すると自動処罰を実行します。

---

## 仕組み

```
違反発生
  ↓
違反タイプに応じたポイントを加算
  ↓
減衰設定がある場合、前回違反からの経過時間でポイントを差し引き
  ↓
処罰ルールを参照し、該当閾値を超えた処罰を実行
```

---

## 違反タイプとポイント

デフォルトのポイントはサーバーごとにカスタマイズできます。

| 違反タイプ | 内部キー |
|---|---|
| Honeypot 踏み込み | `honeypot` |
| スパム | `spam` |
| リアクションスパム | `reaction` |
| 招待リンク違反 | `invite_spam` |
| コンテンツ上限違反 | `content_limit` |
| リンクブロック | `link_blocked` |
| キーワードブロック | `keyword_blocked` |

```
/gs violation-points                    # 現在のポイント設定を表示
/gs violation-points <タイプ> <ポイント>  # ポイントを変更
```

---

## 処罰ルール

累積ポイントがルールの閾値を超えた場合、指定された処罰が自動実行されます。

| 処罰タイプ | 説明 |
|---|---|
| `timeout` | 指定時間タイムアウト |
| `kick` | サーバーからキック |
| `ban` | サーバーから BAN |

```
/gs punishment-rules              # 現在のルールを表示
/gs punishment-rules <閾値> <タイプ> [duration_minutes]
```

---

## ポイント減衰

時間経過によってポイントを自動的に減らす設定です。

| 設定 | 説明 |
|---|---|
| `points_per_interval` | 減衰するポイント量 |
| `interval_minutes` | 減衰間隔（分）|

```
/gs decay          # 現在の設定を表示
/gs decay <points_per_interval> <interval_minutes>
```

---

## 手動ポイント操作

```
/gs add-points <ユーザー> <ポイント>     # ポイントを手動加算（処罰は発動しない）
/gs remove-points <ユーザー> <ポイント>  # ポイントを手動減算
```

---

## Passive Mode（ロール階層不足）

guardian のロールが対象ユーザーのロールより下位にある場合、処罰は保留されます。  
アラートには「[Passive Mode] 保留」と表示されます。

guardian のロールをサーバー設定で上位に移動すると自動解消されます。
