---
id: raid-protection
sidebar_position: 6
title: レイド・参加者対策
sidebar_label: レイド・参加者対策
---

# レイド・参加者対策

参加者スクリーニングとレイド（大量参加攻撃）の検知・対策を行います。

**必要なフラグ**: `enabled`  
各機能は **互いに独立** しており、個別に有効化できます。

---

## ローカル Anti-Raid

**フラグ**: `anti_raid_enabled`

サーバーへの大量参加を検知します。

| 設定 | デフォルト | 説明 |
|---|---|---|
| `raid_join_threshold` | 5 | 何人以上の参加で検知するか |
| `raid_join_window_sec` | 10 | 検知ウィンドウ（秒）|

レイド検知時は、サーバーオーナーへのアラートと認証レベル引き上げの推奨が通知されます。  
自動キックは行いません（手動対応を推奨）。

```
/guardian set-raid <threshold> <window>
/guardian toggle Anti-Raid
```

---

## グローバルレイド検知

**フラグ**: `global_raid_enabled`  
**`cross_server_enabled` は不要です（完全独立）**

同一ユーザーが 60 秒以内に 3 つ以上の関連サーバーに参加した場合に検知します。  
検知時はそのユーザーを自動キックします。

| パラメータ | 値 |
|---|---|
| ウィンドウ | 60 秒（固定）|
| 検知閾値 | 3 サーバー（固定）|

:::warning
ボット側のロール階層が不足している場合は Passive Mode で動作し、キックを保留してアラートのみ送信します。
:::

```
/guardian toggle グローバルレイド検知
```

---

## 招待バースト検知

**フラグ**: `invite_burst_enabled`  
**`anti_raid_enabled` は不要です（完全独立）**

特定の招待コード 1 つから短時間に大量のメンバーが参加した場合、その招待リンクを自動削除します。

| 設定 | デフォルト | 説明 |
|---|---|---|
| `invite_burst_threshold` | 5 | 何人以上の参加で検知するか |
| `invite_burst_window_sec` | 60 | 検知ウィンドウ（秒）|

:::note 必要な権限
「招待リンクを管理」権限が必要です。権限がない場合、このサーバーでの招待バースト検知は自動的に無効化されます。
:::

```
/guardian set-invite <threshold> <window>
/guardian toggle 招待バースト検知
```

---

## アカウント年齢スクリーニング

**フラグ**: `min_account_age_hours`（0 に設定で無効）

設定した時間よりも新しいアカウントの参加を自動キックします。

```
/guardian set-screening <min_age_hours>
```

---

## アバターなしキック

**フラグ**: `kick_no_avatar`

アバターが設定されていないアカウントの参加を自動キックします。

```
/guardian toggle アバターなしキック
```

---

## 未認証 Bot ブロック

**フラグ**: `block_unverified_bots`

`allowed_bot_ids` に登録されていない Bot の参加を自動キックします。

```
/guardian toggle 未認証ボットブロック
/guardian whitelist-add <ID>   # Bot を許可リストに追加
/guardian whitelist-remove <ID>
```

---

## Passive Mode（ロール階層不足時）

guardian のロールが操作対象のロールより下位にある場合、自動処罰は実行されず **Passive Mode** で動作します。  
この場合、セキュリティアラートのみ送信し、処罰は保留されます。

guardian のロールをサーバー設定で上位に移動してください。
