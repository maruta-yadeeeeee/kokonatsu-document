---
id: cf
sidebar_position: 3
title: /cf コマンドリファレンス
sidebar_label: /cf
---

# /cf コマンドリファレンス

`/cf`（content filter）コマンド群は、コンテンツフィルタの機能トグルとサブ機能の個別設定を管理します。  
すべてのコマンドは **サーバーオーナーのみ** 実行できます。

---

## 前提条件

`/cf` コマンドで設定した内容が実際に動作するには、以下の条件を満たす必要があります。

1. **Guardian 全体**が有効であること（`/guardian toggle セキュリティ 大元 true`）
2. **コンテンツフィルタ（大元）**が有効であること（`/cf toggle コンテンツフィルタ (大元) true`）

大元が無効の場合、個別のサブ機能をオンにしても動作しません。  
依存関係の詳細は [機能依存関係](../features/dependencies.md) を参照してください。

---

## コマンド一覧

| コマンド | 説明 |
|---|---|
| `/cf toggle <機能名> <true\|false>` | コンテンツフィルタ機能の ON/OFF を切り替える |
| `/cf content-subflags [引数...]` | コンテンツフィルタのサブ機能を個別に ON/OFF する（引数なしで現在値表示） |

---

## `/cf toggle`

コンテンツフィルタに関連する機能の有効・無効を切り替えます。

### 使い方

```
/cf toggle <機能名> <true|false>
```

### 切り替え可能な機能一覧

| 選択肢（表示名） | 内部フラグ | 説明 |
|---|---|---|
| **【一括】コンテンツ全機能変更** | `cf_all_enabled` | コンテンツフィルタの全サブ機能をまとめて ON/OFF する |
| **コンテンツフィルタ (大元)** | `content_filter_enabled` | コンテンツフィルタ全体の有効・無効（これがオフだと下記すべて無効） |
| **行数チェック** | `cf_check_lines` | メッセージの行数が上限を超えた場合に検知 |
| **文字数チェック** | `cf_check_chars` | メッセージの文字数が上限を超えた場合に検知 |
| **メンション数チェック** | `cf_check_mentions` | メッセージのメンション数が上限を超えた場合に検知 |
| **添付ファイルチェック** | `cf_check_attachments` | 添付ファイルの数・サイズ・拡張子を検査 |
| **リンクフィルタ** | `cf_check_links` | メッセージ内のリンクを検査（ブラックリスト・全ブロック） |
| **キーワードフィルタ** | `cf_check_keywords` | メッセージ内のキーワードをブラックリストと照合 |
| **リンク全ブロック** | `link_block_all` | 許可ドメイン以外のすべてのリンクをブロック |
| **キーワード:ワード単位** | `keyword_match_whole_word` | キーワードを単語単位でマッチさせる（部分一致ではなく完全一致） |
| **キーワード:大小区別** | `keyword_case_sensitive` | キーワードマッチで大文字・小文字を区別する |

### 一括変更について

- `【一括】コンテンツ全機能変更` を `true` にすると、上記のすべてのサブ機能が一括で有効になります。
- `false` にすると、`content_filter_enabled`（大元）が無効になり、サブ機能もすべて動作停止します。

### 依存関係の警告

サブ機能を有効にした際、前提条件が満たされていない場合は警告メッセージが表示されます。

- `⚠️ コンテンツフィルタ（大元）が無効です` → `/cf toggle コンテンツフィルタ (大元) true` を先に実行
- `⚠️ Guardian 全体が無効です` → `/guardian toggle セキュリティ 大元 true` を先に実行

### 使用例

```
# コンテンツフィルタ全体を有効にする
/cf toggle コンテンツフィルタ (大元) true

# リンクフィルタだけ有効にする
/cf toggle リンクフィルタ true

# すべてのコンテンツフィルタ機能を一括有効化
/cf toggle 【一括】コンテンツ全機能変更 true

# キーワードフィルタを無効にする
/cf toggle キーワードフィルタ false
```

---

## `/cf content-subflags`

コンテンツフィルタの 6 つのサブ機能を個別に ON/OFF します。  
**引数なしで実行すると、現在の設定値を一覧表示します。**

### 使い方

```
/cf content-subflags [check_lines] [check_chars] [check_mentions] [check_attachments] [check_links] [check_keywords]
```

### 引数

| 引数 | 型 | 説明 |
|---|---|---|
| `check_lines` | True/False | 行数チェックの有効化 |
| `check_chars` | True/False | 文字数チェックの有効化 |
| `check_mentions` | True/False | メンション数チェックの有効化 |
| `check_attachments` | True/False | 添付ファイルチェックの有効化 |
| `check_links` | True/False | リンクフィルタの有効化 |
| `check_keywords` | True/False | キーワードフィルタの有効化 |

すべての引数は任意です。指定した項目のみが更新されます。

### 使用例

```
# 現在の設定を確認する
/cf content-subflags

# 行数チェックとリンクフィルタだけ有効にする
/cf content-subflags check_lines:true check_links:true

# キーワードフィルタを無効にする
/cf content-subflags check_keywords:false
```

### `/cf toggle` との違い

`/cf toggle` と `/cf content-subflags` はどちらもサブ機能の ON/OFF ができますが、用途が異なります。

| 項目 | `/cf toggle` | `/cf content-subflags` |
|---|---|---|
| **操作対象** | 1つの機能を指定して切替 | 複数の機能をまとめて設定可能 |
| **一括変更** | 「一括」選択肢あり | なし（指定した引数のみ変更） |
| **対象範囲** | サブ機能 + `link_block_all` + キーワードオプション | 6つのサブ機能のみ |
| **現在値確認** | なし | 引数なしで現在値を表示 |

---

## 関連コマンド

コンテンツフィルタの閾値やブラックリストなどの詳細設定は `/gs` コマンドで行います。

| 設定内容 | コマンド |
|---|---|
| 行数・文字数・メンション・添付数の上限値 | `/gs set-content` |
| 添付ファイルのサイズ・拡張子フィルタ | `/gs set-attachment` |
| リンクフィルタの許可ドメイン管理 | `/gs set-link` |
| キーワードフィルタのマッチング設定 | `/gs set-keyword` |
| キーワードブラックリストの追加・削除 | `/gs blacklist-keyword` |
| ドメインブラックリストの追加・削除 | `/gs blacklist-domain` |
| コンテンツ違反時のアクション設定 | `/gs content-action` |
| 信頼済みロール管理（上限バイパス） | `/gs trusted-role` |
| チャンネル別オーバーライド | `/gs channel-override` |

詳しくは [/gs コマンドリファレンス](gs.md) を参照してください。
