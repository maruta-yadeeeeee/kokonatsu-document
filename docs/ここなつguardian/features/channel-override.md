---
id: channel-override
sidebar_position: 11
title: チャンネルオーバーライド
sidebar_label: チャンネルオーバーライド
---

# チャンネルオーバーライド

特定のチャンネルに対して、サーバー全体の設定を上書きする追加設定を適用できます。

---

## 使い方

```
/guardian channel-override <チャンネル> <設定キー> <値>
```

---

## 設定可能なキー

コンテンツフィルタ関連の `SecurityConfig` フィールドが設定可能です。  
主な用途例:

| キー | 説明 | 例 |
|---|---|---|
| `max_message_lines` | そのチャンネルの行数上限を変更 | `100` |
| `max_message_chars` | 文字数上限を変更 | `5000` |
| `max_mentions` | メンション数上限を変更 | `20` |
| `max_attachments` | 添付数上限を変更 | `10` |
| `cf_check_links` | そのチャンネルでリンクチェックを無効 | `false` |
| `cf_check_keywords` | そのチャンネルでキーワードチェックを無効 | `false` |
| `cf_action_keywords` | そのチャンネルのキーワードアクションを変更 | `log_only` |
| `link_block_all` | そのチャンネルで全リンクブロックを有効 | `true` |

---

## オーバーライドの解除

```
/guardian channel-override <チャンネル> <設定キー> reset
```

---

## 動作の仕組み

オーバーライドはデータベースに保存され、メッセージ処理時にベース設定にマージされます。  
チャンネルオーバーライドが存在しない場合は、サーバー全体の設定がそのまま適用されます。

:::note
チャンネルオーバーライドは **コンテンツフィルタ（ContentFilter）** にのみ適用されます。  
スパム検知・レイド対策・Nuke 保護はサーバー全体の設定が適用されます。
:::
