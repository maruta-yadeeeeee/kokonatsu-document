---
id: nuke-protection
sidebar_position: 8
title: Nuke 保護
sidebar_label: Nuke 保護
---

# Nuke 保護

管理者権限を持つユーザーやボットによる「Nuke」攻撃（短時間での大量チャンネル削除・ロール削除・処罰実行）を検知します。

**必要なフラグ**: `enabled` + `nuke_protection_enabled`

---

## 検知対象

| イベント | 説明 |
|---|---|
| チャンネル削除 | 短時間での大量チャンネル削除 |
| ロール削除 | 短時間での大量ロール削除 |
| 大量処罰 | 短時間でのキック・BAN・タイムアウトの大量実行 |

**注意**: guardian 自身による自動処罰はカウントから除外されます。

---

## 閾値設定

| 設定 | デフォルト | 説明 |
|---|---|---|
| `destruction_threshold` | 3 | 検知件数（N件/ウィンドウ）|
| `destruction_window_sec` | 10 | 検知ウィンドウ（秒）|

```
/guardian set-nuke <threshold> <window>
/guardian toggle Nuke保護
```

---

## 検知後の動作

1. 監査ログから実行者を特定（5 秒以内のエントリを参照）
2. 実行者が特定できた場合 → 自動 BAN
3. 実行者が特定できない場合（監査ログ権限なし等）→ アラートのみ（防衛不可モード）

:::warning 監査ログ権限
Nuke 保護が正しく機能するには、「監査ログを表示」権限が必要です。  
権限がない場合、実行者を特定できず自動処罰が行えません。
:::

---

## Webhook バースト監視

Webhook からの短時間大量送信も別途監視できます。

**フラグ**: `webhook_burst_enabled`

| 設定 | デフォルト | 説明 |
|---|---|---|
| `webhook_burst_threshold` | 10 | 何件で検知するか |
| `webhook_burst_window_sec` | 10 | 検知ウィンドウ（秒）|

検知時は対象 Webhook を削除します。
