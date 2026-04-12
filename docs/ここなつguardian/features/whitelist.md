---
id: whitelist
sidebar_position: 10
title: ホワイトリスト・除外設定
sidebar_label: ホワイトリスト
---

# ホワイトリスト・除外設定

guardian のスキャンから特定のユーザー・ロール・チャンネル・Bot を除外できます。

---

## メッセージスキャン除外

### チャンネル除外

指定したチャンネルのメッセージはすべてのスキャンをスキップします。

```
/guardian exclude-channel <チャンネル>    # 除外に追加
/guardian exclude-channel remove <チャンネル>
```

### ユーザーホワイトリスト

```
/guardian whitelist-add <ユーザーID>     # allowed_user_ids
/guardian whitelist-remove <ユーザーID>
```

### ロールホワイトリスト

```
/guardian whitelist-add role <ロールID>  # allowed_role_ids
/guardian whitelist-remove role <ロールID>
```

---

## Bot・Webhook スキャン

デフォルトでは Bot と Webhook のメッセージはスキャンされません。  
有効化した場合、違反 Bot は全 Detector パイプラインで検査され、違反時にキックされます。

```
/guardian toggle ボットスキャン    # scan_bot_messages
/guardian toggle Webhookスキャン  # scan_webhook_messages
```

### Bot ホワイトリスト

`scan_bot_messages` 有効時でも、許可リスト登録済み Bot はスキャンをスキップします。  
また、`block_unverified_bots` 有効時はこのリストに登録されていない Bot の参加がブロックされます。

```
/guardian whitelist-add bot <BotID>
/guardian whitelist-remove bot <BotID>
```

---

## 信頼済みロール（コンテンツフィルタ上限バイパス）

信頼済みロールのユーザーはコンテンツ上限チェック（行数・文字数・メンション・添付）をバイパスします。  
ただし、リンクフィルタとキーワードフィルタは引き続き適用されます。

```
/guardian trusted-roles-add <ロールID>
/guardian trusted-roles-remove <ロールID>
```

---

## ブラックリスト管理

ホワイトリストとは逆に、特定のキーワードやドメインを禁止します。

```
/gs blacklist-keyword <add|remove|list> [pattern] [is_regex]
/gs blacklist-domain <add|remove|list> [domain]
```

キーワードは正規表現（`is_regex=True`）にも対応しています。
