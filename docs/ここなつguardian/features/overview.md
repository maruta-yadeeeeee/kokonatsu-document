---
id: overview
sidebar_position: 1
title: 機能一覧
sidebar_label: 機能一覧
---

# 機能一覧

guardian の全機能を一覧します。各機能の有効化フラグと、その機能が動作するために必要な前提条件を示します。

---

## 大元スイッチ

| フラグ | 説明 |
|---|---|
| `enabled` | **すべての機能の共通前提条件**。`false` の場合、いかなる検知・処罰も実行されません。 |

---

## コンテンツフィルタ

`enabled` + `content_filter_enabled` が必要です。

| フラグ | 説明 |
|---|---|
| `cf_check_lines` | メッセージの行数制限（`max_message_lines`）|
| `cf_check_chars` | 文字数制限（`max_message_chars`）|
| `cf_check_mentions` | メンション数制限（`max_mentions`）|
| `cf_check_attachments` | 添付ファイル制限（数・サイズ・拡張子）|
| `cf_check_links` | リンクフィルタ（ブラックリストドメイン・全リンクブロック）|
| `cf_check_keywords` | キーワードフィルタ（ブラックリストワード）|

:::info trusted_role_ids について
`trusted_role_ids` に登録されたロールのユーザーは、行数・文字数・メンション・添付の上限チェックをスキップします。  
ただし **リンクフィルタとキーワードフィルタは trusted_role_ids があっても常に適用されます**。
:::

---

## スパム検知

`enabled` + `spam_detection_enabled` が必要です。

| フラグ | 説明 |
|---|---|
| `solo_spam_enabled` | 1人による高速連投の検知（`solo_spam_threshold` / `solo_spam_window_sec`）|
| `coordinated_spam_enabled` | 複数ユーザーによる類似メッセージの協調スパム検知 |
| `cross_server_enabled` | クロスサーバー間のスパム相関検知（`coordinated_spam_enabled` と組み合わせ）|

---

## レイド・参加者対策

`enabled` が必要です。各機能は **互いに独立** しています。

| フラグ | 説明 |
|---|---|
| `anti_raid_enabled` | ローカル Anti-Raid（短時間での大量参加を検知）|
| `invite_burst_enabled` | 招待バースト検知（特定の招待コードから短時間に大量参加）**Anti-Raid とは独立** |
| `global_raid_enabled` | グローバルレイド検知（同一ユーザーが複数サーバーに連続参加）**cross_server_enabled 不要** |
| `kick_no_avatar` | アバター未設定メンバーの自動キック（`anti_raid_enabled` 不要）|
| `min_account_age_hours` | アカウント年齢が不足しているメンバーの自動キック（`anti_raid_enabled` 不要）|
| `block_unverified_bots` | ホワイトリスト未登録 Bot の参加ブロック（`allowed_bot_ids` でホワイトリスト管理）|

---

## 招待リンクフィルタ

| フラグ | 説明 |
|---|---|
| `invite_filter_enabled` | 外部サーバーへの招待リンクの送信を禁止する（`content_filter_enabled` 不要・独立）|

---

## Nuke 保護

| フラグ | 説明 |
|---|---|
| `enabled` + `nuke_protection_enabled` | 大量チャンネル削除・ロール削除・処罰操作の検知 |

---

## Reaction スパム / Webhook バースト

| フラグ | 説明 |
|---|---|
| `enabled` + `reaction_spam_enabled` | リアクションスパムの検知 |
| `enabled` + `webhook_burst_enabled` | Webhook からの大量送信の検知 |

---

## Honeypot

| フラグ | 説明 |
|---|---|
| `honeypot_channel_id` 設定済み | 隠しチャンネルへの書き込みを即時検知 |

---

## メッセージ処理の除外設定

以下の設定が当てはまるメッセージは、Detector パイプライン（コンテンツフィルタ・スパム等）に到達しません。

| 設定 | 効果 |
|---|---|
| `excluded_channel_ids` | そのチャンネルのメッセージ全体をスキャン対象外にする |
| `allowed_user_ids` | 指定ユーザーのメッセージをスキャン対象外にする |
| `allowed_role_ids` | 指定ロール所持ユーザーのメッセージをスキャン対象外にする |
| `allowed_bot_ids` | 指定 Bot をスキャン対象外にする（`scan_bot_messages` 有効時）|
