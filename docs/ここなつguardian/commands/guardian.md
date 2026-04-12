---
id: guardian
sidebar_position: 1
title: /guardian コマンドリファレンス
sidebar_label: /guardian
---

# /guardian コマンドリファレンス

すべての `/guardian` コマンドは **サーバーオーナーのみ** 実行できます。

---

## セットアップ

| コマンド | 説明 |
|---|---|
| `/guardian setup-log <チャンネル>` | セキュリティログチャンネルを設定する |
| `/guardian setup-honeypot <チャンネル>` | Honeypot チャンネルを設定する |
| `/guardian reset-honeypot` | Honeypot チャンネル設定を解除する |

---

## 全体状態確認

| コマンド | 説明 |
|---|---|
| `/guardian status` | 現在のすべての設定と状態を表示する |
| `/guardian help` | コマンド一覧（ヘルプ）を表示する |

---

## 機能トグル

```
/guardian toggle <機能名> <true|false>
```

Guardian の各セキュリティ機能を個別に有効・無効にします。  
一部の機能は親子関係（依存関係）があります。詳しくは [機能依存関係](../features/dependencies.md) を参照してください。

| 機能名（表示） | 対応フラグ | 説明 |
|---|---|---|
| `【一括】全機能変更` | `all_enabled` | すべてのフラグを一括で ON/OFF する |
| `セキュリティ 大元` | `enabled` | Guardian 全体の大元スイッチ。オフにすると全機能停止 |
| `Anti-Raid` | `anti_raid_enabled` | 短時間に大量のメンバーが参加した場合に検知 |
| `招待バースト検知` | `invite_spam_enabled` | 特定の招待コードから短時間に大量参加を検知 |
| `グローバルレイド検知` | `global_raid_enabled` | 同一ユーザーが複数サーバーに短時間で参加を検知 |
| `クロスサーバー検知` | `cross_server_enabled` | 複数サーバー間でスパムメッセージのハッシュを照合 |
| `アバターなしキック` | `kick_no_avatar` | アバター未設定のアカウントを自動キック |
| `Nuke保護` | `nuke_protection_enabled` | チャンネル・ロール大量削除等の破壊行為を検知 |
| `コンテンツフィルタ` | `content_filter_enabled` | メッセージ内容の検査（大元スイッチ） |
| `招待リンクフィルタ` | `invite_filter_enabled` | Discord 招待リンクの送信をブロック |
| `スパム検知 (大元)` | `spam_detection_enabled` | スパム検知全体の大元スイッチ |
| `スパム検知 (単独)` | `solo_spam_enabled` | 1人による高速連投を検知 |
| `スパム検知 (協調)` | `coordinated_spam_enabled` | 複数人による類似メッセージの同時投稿を検知 |
| `リアクションスパム` | `reaction_spam_enabled` | 大量リアクション付与を検知 |
| `Webhookスパム` | `webhook_burst_enabled` | Webhook からの大量送信を検知 |
| `未認証ボットブロック` | `block_unverified_bots` | ホワイトリスト外の未認証ボットの参加をブロック |
| `ボットスキャン` | `scan_bot_messages` | ボットのメッセージも各 Detector で検査 |
| `Webhookスキャン` | `scan_webhook_messages` | Webhook のメッセージも各 Detector で検査 |

全機能を一括でオン/オフする場合:

```
/guardian toggle すべて有効
/guardian toggle すべて無効
```

---

## ボット・Webhook フィルター設定

### `/guardian bot-filter [block_unverified] [scan_messages]`

未認証ボットのブロックおよびボットメッセージのスキャンを設定します。  
引数なしで現在値を表示します。

| 引数 | 説明 |
|---|---|
| `block_unverified` | `true` = ホワイトリスト外の未認証ボット参加をブロック |
| `scan_messages` | `true` = ボットのメッセージをスキャン（違反時キック）|

### `/guardian webhook-filter <scan_messages>`

Webhook メッセージのスキャンを ON/OFF します。違反時は Webhook を削除します。

---

## 閾値・設定コマンド

各検知機能の閾値（何回で検知するか）を個別に設定します。  
**すべてのコマンドは引数なしで実行すると現在の設定値を表示します。**

| コマンド | 説明 | 主な引数 |
|---|---|---|
| `/guardian set-raid` | Anti-Raid 閾値設定（N人がM秒以内に参加で検知） | `threshold`（人数）, `window`（秒）|
| `/guardian set-invite` | 招待バースト検知 閾値設定（特定招待コードからの大量参加） | `threshold`, `window` |
| `/guardian set-spam` | 協調スパム閾値設定（複数人の類似メッセージ検知） | `cluster_size`, `window`, `similarity` |
| `/guardian set-solo-spam` | 単独スパム閾値設定（1人の高速連投検知） | `threshold`, `window` |
| `/guardian set-reaction` | リアクションスパム閾値設定 | `threshold`, `window` |
| `/guardian set-webhook-spam` | Webhook スパム閾値設定 | `threshold`, `window` |
| `/guardian set-bot-spam` | ボットスパム監視閾値設定 | `threshold`, `window` |
| `/guardian set-nuke` | Nuke 保護閾値設定（チャンネル/ロール大量削除の検知） | `threshold`, `window` |
| `/guardian set-screening` | 最低アカウント年齢設定（新規アカウントの自動キック） | `min_age_hours` |

---

## コンテンツフィルタ 詳細設定

### `/guardian set-content`

メッセージの上限値を設定します。

| 引数 | デフォルト | 説明 |
|---|---|---|
| `max_lines` | 30 | 1メッセージあたりの最大行数 |
| `max_chars` | 2000 | 1メッセージあたりの最大文字数 |
| `max_mentions` | 10 | 1メッセージあたりの最大メンション数 |
| `max_attachments` | 5 | 1メッセージあたりの最大添付ファイル数 |

### `/guardian content-subflags`

コンテンツフィルタのどのチェックを有効にするかを個別に設定します。

| 引数 | 対応フラグ |
|---|---|
| `check_lines` | 行数チェック |
| `check_chars` | 文字数チェック |
| `check_mentions` | メンション数チェック |
| `check_attachments` | 添付ファイルチェック |
| `check_links` | リンクフィルタ |
| `check_keywords` | キーワードフィルタ |

引数なしで現在値を表示します。

### `/guardian set-attachment`

添付ファイルの詳細フィルタを設定します。

| 引数 | 説明 |
|---|---|
| `max_size_mb` | ファイルサイズ上限（MB）。`0` で無制限 |
| `block_ext` | ブロックする拡張子をカンマ区切りで指定（例: `exe,bat,sh`）|
| `allow_ext` | 許可する拡張子のみをカンマ区切りで指定（例: `png,jpg,gif`）。設定すると他はすべて拒否。空文字で解除 |

### `/guardian set-link`

リンクフィルタの詳細設定です。

| 引数 | 説明 |
|---|---|
| `block_all` | `true` = 許可ドメイン以外の外部リンクをすべてブロック |
| `add_allowed` | 許可ドメインを追加（例: `discord.com`）|
| `remove_allowed` | 許可ドメインから削除 |

引数なしで現在の設定（全リンクブロックモードと許可ドメイン一覧）を表示します。

### `/guardian set-keyword`

キーワードマッチングの詳細設定です。

| 引数 | 説明 |
|---|---|
| `whole_word` | `true` = 単語境界単位でマッチ（部分一致から変更）|
| `case_sensitive` | `true` = 大文字小文字を区別する |

---

## アクションモード設定

各モジュールの違反時アクションは専用コマンドで個別に設定します。

### `/guardian content-action [check_type] [action]`

コンテンツフィルタ違反時のアクションモードを設定します。

**check_type の選択肢：**

| 値 | 対象 |
|---|---|
| `コンテンツ上限（行数・文字数・メンション）` | `cf_action_content_limit` |
| `添付ファイル違反` | `cf_action_attachments` |
| `リンク違反` | `cf_action_links` |
| `キーワード違反` | `cf_action_keywords` |

### `/guardian spam-action [check_type] [action]`

スパム検知違反時のアクションモードを設定します。

| 値 | 対象 |
|---|---|
| `単独スパム` | `spam_action_solo` |
| `協調スパム（クラスター）` | `spam_action_cluster` |

### `/guardian invite-action [action]`

招待リンクフィルタ違反時のアクションモードを設定します。

**action の共通選択肢（全コマンド共通）：**

| 値 | 説明 |
|---|---|
| `delete_notify` | 削除 + チャンネル通知（デフォルト）|
| `delete_silent` | 削除のみ（通知なし）|
| `warn_only` | 削除せず警告メッセージのみ |
| `log_only` | ログのみ（削除・通知・ポイント加算なし）|

引数なしで現在の設定を表示します。

---

## ホワイトリスト・信頼済みロール・除外チャンネル

```
/guardian whitelist-add [user] [role] [bot_id]
/guardian whitelist-remove [user] [role] [bot_id]
/guardian whitelist-list
```

ホワイトリストに登録されたユーザー・ロール・ボットはすべての検知から除外されます。

```
/guardian trusted-role [action] [role]
```

信頼済みロールを管理します。`action` は `add` または `remove` を選択。引数なしで一覧表示。  
信頼済みロールは行数・文字数・メンション・添付ファイルの**上限チェックをバイパス**します。  
キーワード・リンクフィルタは引き続き適用されます。

```
/guardian exclude-channel <チャンネル>
```

チャンネルをセキュリティスキャン対象から除外します。再実行で解除できます。

---

## チャンネルオーバーライド

チャンネルごとにコンテンツフィルタ設定を上書きできます。

```
/guardian channel-override [channel] [max_lines] [max_chars] [max_mentions] [max_attachments] [content_filter] [action] [clear]
```

| 引数 | 説明 |
|---|---|
| `channel` | 対象チャンネル（省略時は全上書き設定を一覧表示）|
| `max_lines` / `max_chars` / `max_mentions` / `max_attachments` | 該当チャンネル専用の上限値（`0` で該当設定を解除）|
| `content_filter` | このチャンネルのコンテンツフィルタを有効/無効に上書き |
| `action` | このチャンネルのすべての違反アクションを上書き |
| `clear` | `true` でこのチャンネルのすべての上書き設定をリセット |

引数なし（チャンネルなし）で全チャンネルの上書き設定一覧を表示します。

---

## 関連コマンドグループ

| グループ | 説明 | リファレンス |
|---|---|---|
| `/cf` | コンテンツフィルタの個別トグル・サブ機能設定 | [/cf コマンドリファレンス](cf.md) |
| `/gs` | ポイント・処罰・ブラックリスト等の詳細設定 | [/gs コマンドリファレンス](gs.md) |
| `/kb` | サーバーのバックアップ・復元 | [/kb コマンドリファレンス](kb.md) |

---

## 機能依存関係について

Guardian の多くの機能は「大元スイッチ」と「個別スイッチ」の2段構造です。  
「オンにしたのに動かない」場合は、前提となるフラグがオフになっている可能性があります。  
詳しくは [機能依存関係](../features/dependencies.md) を参照してください。
