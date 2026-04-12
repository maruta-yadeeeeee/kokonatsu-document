---
id: content-filter
sidebar_position: 4
title: コンテンツフィルタ
sidebar_label: コンテンツフィルタ
---

# コンテンツフィルタ

メッセージの内容を多角的にフィルタリングします。

**必要なフラグ**: `enabled` + `content_filter_enabled`

---

## サブ機能

### メッセージ上限チェック

| フラグ | 設定値 | 説明 |
|---|---|---|
| `cf_check_lines` | boolean | 行数制限を有効化 |
| `max_message_lines` | 整数 (default: 30) | 1メッセージあたりの最大行数 |
| `cf_check_chars` | boolean | 文字数制限を有効化 |
| `max_message_chars` | 整数 (default: 2000) | 1メッセージあたりの最大文字数 |
| `cf_check_mentions` | boolean | メンション数制限を有効化 |
| `max_mentions` | 整数 (default: 10) | 1メッセージあたりの最大メンション数 |

上限超過時のアクションモード: `cf_action_content_limit`

### 添付ファイルフィルタ

| フラグ | 設定値 | 説明 |
|---|---|---|
| `cf_check_attachments` | boolean | 添付ファイルフィルタを有効化 |
| `max_attachments` | 整数 (default: 5) | 1メッセージあたりの最大添付数 |
| `max_attachment_size_mb` | 小数 (default: 0.0) | 1ファイルあたりのMB上限（0=制限なし）|
| `blocked_extensions` | リスト | ブロックする拡張子 例: `["exe", "bat", "sh"]` |
| `allowed_extensions` | リスト | 許可のみ（設定時はリスト外を全拒否） |

`blocked_extensions` と `allowed_extensions` は排他的に使用してください。  
両方設定した場合、`allowed_extensions`（許可リスト）が優先されます。

違反時のアクションモード: `cf_action_attachments`

### リンクフィルタ

| フラグ | 設定値 | 説明 |
|---|---|---|
| `cf_check_links` | boolean | リンクフィルタを有効化 |
| `link_block_all` | boolean | 全リンクをブロック（`allowed_domains` 以外）|
| `allowed_domains` | リスト | 許可ドメイン（`link_block_all` 時でも許可）|

ブラックリストドメインは `/gs blacklist-domain` で管理します。

違反時のアクションモード: `cf_action_links`

### キーワードフィルタ

| フラグ | 設定値 | 説明 |
|---|---|---|
| `cf_check_keywords` | boolean | キーワードフィルタを有効化 |
| `keyword_match_whole_word` | boolean | 単語境界マッチ（部分一致を除外）|
| `keyword_case_sensitive` | boolean | 大文字小文字を区別する |

ブラックリストキーワード（通常文字列・正規表現対応）は `/gs blacklist-keyword` で管理します。

違反時のアクションモード: `cf_action_keywords`

---

## trusted_role_ids（信頼済みロール）

`/guardian trusted-roles-add <ロールID>` で登録したロールのユーザーは、以下のチェックをバイパスします。

- ✅ バイパス: 行数・文字数・メンション・添付ファイルの上限チェック
- ❌ バイパス不可: リンクフィルタ・キーワードフィルタ（常に適用）

---

## チャンネルオーバーライド

特定のチャンネルだけ設定を上書きできます。

```
/guardian channel-override <チャンネル> <設定キー> <値>
```

例: `#general` チャンネルのみ `max_message_lines` を 50 に設定する

```
/guardian channel-override #general max_message_lines 50
```

---

## 設定コマンド

```
/guardian toggle コンテンツフィルタ        # content_filter_enabled ON/OFF
/guardian set-content-limit <行数> <文字数> <メンション数> <添付数>
/gs blacklist-keyword <add|remove|list> [pattern] [is_regex]
/gs blacklist-domain <add|remove|list> [domain]
```
