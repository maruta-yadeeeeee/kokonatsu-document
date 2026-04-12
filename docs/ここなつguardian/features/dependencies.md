---
id: dependencies
sidebar_position: 2
title: 機能依存関係
sidebar_label: 依存関係
---

# 機能依存関係

Guardian のすべての機能はフラグの組み合わせで制御されています。  
**「オンにしたのに動かない！」** という場合、ほとんどの原因は **前提となるフラグがオフになっている** ことです。

このページでは、各機能が動作するために「何をオンにする必要があるのか」を、初めて使う人にもわかるように説明します。

---

## はじめに読んでください: 3つの大原則

### 大原則 1: `enabled`（Guardian 全体）がすべての大元

```
enabled = false → Guardian は何も動きません
```

どんな機能をオンにしても、`enabled` がオフなら **すべて停止** しています。  
最初に必ず以下を実行してください:

```
/guardian toggle セキュリティ 大元 true
```

### 大原則 2: 「大元スイッチ」と「個別スイッチ」の2段構造

Guardian の多くの機能は、**大元のスイッチ**（親）と**個別のスイッチ**（子）の2段構造です。

**例: コンテンツフィルタ**
```
content_filter_enabled（大元） ← これがオフだと ↓ は全部動かない
  ├─ cf_check_lines（行数チェック）
  ├─ cf_check_chars（文字数チェック）
  ├─ cf_check_mentions（メンション数チェック）
  ├─ cf_check_attachments（添付ファイルチェック）
  ├─ cf_check_links（リンクフィルタ）
  └─ cf_check_keywords（キーワードフィルタ）
```

**大元がオフだと、子のスイッチをいくらオンにしても動きません。**

### 大原則 3: 独立した機能は互いに影響しない

Anti-Raid と コンテンツフィルタ は互いに無関係です。  
片方だけオンにしても、もう片方には影響しません。

---

## 「動かない！」原因チェックリスト

以下の表で、あなたが使いたい機能の行を探し、「必要なフラグ」がすべてオンになっているか確認してください。**どれか1つでもオフだと動きません。**

| やりたいこと | 必要なフラグ（すべて `true` にする） | 設定コマンド |
|---|---|---|
| **行数を制限したい** | `enabled` → `content_filter_enabled` → `cf_check_lines` | `/guardian toggle` → `/cf toggle` |
| **文字数を制限したい** | `enabled` → `content_filter_enabled` → `cf_check_chars` | `/guardian toggle` → `/cf toggle` |
| **メンション数を制限したい** | `enabled` → `content_filter_enabled` → `cf_check_mentions` | `/guardian toggle` → `/cf toggle` |
| **添付ファイルを制限したい** | `enabled` → `content_filter_enabled` → `cf_check_attachments` | `/guardian toggle` → `/cf toggle` |
| **リンクをブロックしたい** | `enabled` → `content_filter_enabled` → `cf_check_links` | `/guardian toggle` → `/cf toggle` |
| **NGワードを設定したい** | `enabled` → `content_filter_enabled` → `cf_check_keywords` + ブラックリスト登録 | `/guardian toggle` → `/cf toggle` → `/gs blacklist-keyword add` |
| **外部招待リンクを禁止したい** | `enabled` → `invite_filter_enabled` | `/guardian toggle` |
| **単独スパムを検知したい** | `enabled` → `spam_detection_enabled` → `solo_spam_enabled` | `/guardian toggle` |
| **協調スパムを検知したい** | `enabled` → `spam_detection_enabled` → `coordinated_spam_enabled` | `/guardian toggle` |
| **クロスサーバースパム検知** | `enabled` → `spam_detection_enabled` → `coordinated_spam_enabled` → `cross_server_enabled` | `/guardian toggle` |
| **大量参加を検知したい** | `enabled` → `anti_raid_enabled` | `/guardian toggle` |
| **招待バースト検知** | `enabled` → `invite_spam_enabled` | `/guardian toggle` |
| **グローバルレイド検知** | `enabled` → `global_raid_enabled` | `/guardian toggle` |
| **アバターなしキック** | `enabled` → `kick_no_avatar` | `/guardian toggle` |
| **未認証Botブロック** | `enabled` → `block_unverified_bots` | `/guardian toggle` |
| **Nuke保護** | `enabled` → `nuke_protection_enabled` | `/guardian toggle` |
| **リアクションスパム検知** | `enabled` → `reaction_spam_enabled` | `/guardian toggle` |
| **Webhookバースト検知** | `enabled` → `webhook_burst_enabled` | `/guardian toggle` |
| **Botメッセージもスキャン** | `enabled` → `scan_bot_messages` + 各 Detector が有効 | `/guardian toggle` |
| **Webhookメッセージもスキャン** | `enabled` → `scan_webhook_messages` + 各 Detector が有効 | `/guardian toggle` |

### よくある「動かない」パターンと解決方法

**Q. キーワードフィルタをオンにしたのに、NGワードが検知されない**

→ 以下の3つすべてがオンか確認してください:
1. `/guardian toggle セキュリティ 大元 true`（Guardian 全体）
2. `/cf toggle コンテンツフィルタ (大元) true`（コンテンツフィルタの大元）
3. `/cf toggle キーワードフィルタ true`（キーワードフィルタ個別）

さらに、ブラックリストにキーワードが登録されている必要があります:
```
/gs blacklist-keyword add <パターン>
```

**Q. スパム検知をオンにしたのに、連投が検知されない**

→ スパム検知は3段構造です:
1. `/guardian toggle セキュリティ 大元 true`
2. `/guardian toggle スパム検知 (大元) true`
3. `/guardian toggle スパム検知 (単独) true` または `/guardian toggle スパム検知 (協調) true`

大元だけオンにしても、「単独」か「協調」のどちらかを個別にオンにしないと動きません。

**Q. クロスサーバースパム検知をオンにしたのに効かない**

→ クロスサーバーは「協調スパム検知」の拡張です:
1. `enabled` = true
2. `spam_detection_enabled` = true
3. `coordinated_spam_enabled` = true ← これが必要
4. `cross_server_enabled` = true

**Q. 招待リンクフィルタとリンクフィルタ、どっちを使えばいい？**

→ 目的が違います:
- **`invite_filter_enabled`** → Discord の招待リンク（discord.gg/... 等）のみをブロック。コンテンツフィルタ不要で独立動作
- **`cf_check_links`** → すべてのURL/リンクを対象に検査。コンテンツフィルタの一部として動作

---

## 2. 大きな処理系統と依存フロー

Guardian の内部は大きく次の2つの処理に分かれます。

1. メッセージ処理パイプライン
2. 参加者 / Audit Log / Bot 監視パイプライン

### 1) メッセージ処理パイプライン

```
メッセージ受信
  ↓
enabled チェック
  ↓
除外チェック
  ↓
Webhook/ボット判定
  ↓
ホワイトリスト / Honeypot
  ↓
Detector パイプライン
```

#### 1.1 除外チェック

以下に該当するメッセージは処理対象から除外されます。

- `excluded_channel_ids` に含まれるチャンネル
- `allowed_user_ids` に含まれるユーザー
- `allowed_role_ids` のロールを持つユーザー

#### 1.2 Honeypot

- `honeypot_channel_id` が設定されている場合、対象チャンネルのメッセージは Honeypot 処理へと送られます。
- `enabled` が `false` ならこれも動きません。

#### 1.3 Detector パイプライン

Detector は次の順序で評価されます。

1. ContentFilter
2. InviteFilter
3. SpamDetector

各 Detector は別個のフラグを必要とします。

---

## 3. コンテンツフィルタ依存関係

### 3.1 コンテンツフィルタ大元

```
content_filter_enabled = false → 内容検査系は一切動作しない
```

`content_filter_enabled` が `false` なら、以下のサブ機能はすべて無効です。

- `cf_check_lines`
- `cf_check_chars`
- `cf_check_mentions`
- `cf_check_attachments`
- `cf_check_links`
- `cf_check_keywords`

このため、コンテンツフィルタを有効にするだけでは足りず、さらに各サブフラグをオンにする必要があります。

### 3.2 個別サブ機能

| 機能 | 必要なフラグ |
|---|---|
| 行数チェック | `enabled` + `content_filter_enabled` + `cf_check_lines` |
| 文字数チェック | `enabled` + `content_filter_enabled` + `cf_check_chars` |
| メンションチェック | `enabled` + `content_filter_enabled` + `cf_check_mentions` |
| 添付チェック | `enabled` + `content_filter_enabled` + `cf_check_attachments` |
| リンクフィルタ | `enabled` + `content_filter_enabled` + `cf_check_links` |
| キーワードフィルタ | `enabled` + `content_filter_enabled` + `cf_check_keywords` |

#### 3.3 添付ファイルの詳細

添付チェックが有効でも、以下を使ってさらに挙動を制御できます。

- `max_attachment_size_mb`
- `blocked_extensions`
- `allowed_extensions`

`allowed_extensions` が設定されると、それ以外の拡張子はすべて拒否されます。

#### 3.4 trusted_role_ids の意味

- `trusted_role_ids` のロールを持つユーザーは、
  - `cf_check_lines`
  - `cf_check_chars`
  - `cf_check_mentions`
  - `cf_check_attachments`
  の上限チェックをバイパスします。
- ただし、リンクフィルタやキーワードフィルタは引き続き適用されます。

---

## 4. スパム検知依存関係

### 4.1 スパム検知大元

```
spam_detection_enabled = false → スパム検知はすべて無効
```

`spam_detection_enabled` は以下の2つのサブ機能をくくる大元です。

- `solo_spam_enabled`
- `coordinated_spam_enabled`

### 4.2 単独スパム

- 必要フラグ: `enabled` + `spam_detection_enabled` + `solo_spam_enabled`
- 閾値: `solo_spam_threshold` / `solo_spam_window_sec`
- アクション: `spam_action_solo`

### 4.3 協調スパム

- 必要フラグ: `enabled` + `spam_detection_enabled` + `coordinated_spam_enabled`
- 閾値: `spam_cluster_size` / `spam_window_sec` / `similarity_threshold`
- アクション: `spam_action_cluster`

#### 4.3.1 `cross_server_enabled` の役割

- `coordinated_spam_enabled = true` のときに
  - `cross_server_enabled = false` → 同サーバー内のみの協調検知
  - `cross_server_enabled = true`  → 複数サーバー間のハッシュ照合も追加
- `cross_server_enabled` 自体は、`spam_detection_enabled` が有効でないと意味を持ちません。

---

## 5. 参加者 / Nuke / Bot 監視依存関係

### 5.1 参加処理

`enabled` が前提です。以下の各機能は個別に動作します。

| 機能 | 必要なフラグ | 備考 |
|---|---|---|
| `anti_raid_enabled` | `enabled` + `anti_raid_enabled` | local Raid 検知 |
| `invite_burst_enabled` | `enabled` + `invite_burst_enabled` | 特定招待リンクの急増検知 |
| `global_raid_enabled` | `enabled` + `global_raid_enabled` | 複数サーバー跨ぎ参加検知 |
| `block_unverified_bots` | `enabled` + `block_unverified_bots` | ホワイトリスト外 Bot をブロック |
| `kick_no_avatar` | `enabled` + `kick_no_avatar` | アバターなしアカウントをキック |
| `min_account_age_hours` | `enabled` | アカウント年齢スクリーニング |

### 5.2 Nuke 保護 / Audit Log

| 機能 | 必要なフラグ |
|---|---|
| `nuke_protection_enabled` | `enabled` + `nuke_protection_enabled` |
| `reaction_spam_enabled` | `enabled` + `reaction_spam_enabled` |
| `webhook_burst_enabled` | `enabled` + `webhook_burst_enabled` |

### 5.3 Bot / Webhook スキャン

- `scan_bot_messages` は `enabled` 前提で Bot メッセージを監視する
- `scan_webhook_messages` は `enabled` 前提で Webhook メッセージを監視する
- これらは `content_filter_enabled` や `spam_detection_enabled` に依存せず、別途動作できます

---

## 6. 重要な誤解を防ぐためのまとめ

### 6.1 すべての機能は `enabled = true` が前提

- `enabled = false` にすると、スパムもコンテンツも招待リンクも復元もすべて“停止”します。
- まず真っ先に `/guardian toggle セキュリティ 大元 true` を行ってください。

### 6.2 `content_filter_enabled` はコンテンツフィルタの大元

- `content_filter_enabled = false` なら、行数・文字数・メンション・添付・リンク・キーワードすべて無効
- それぞれのサブフラグは `content_filter_enabled = true` のうえで有効化する必要あり

### 6.3 `spam_detection_enabled` はスパム検知の大元

- `spam_detection_enabled = false` だと、`solo_spam_enabled` も `coordinated_spam_enabled` も動作しない
- `cross_server_enabled` は協調スパムの拡張であり、単独で動作しない

---

## 7. コマンド対応表

どのフラグをどのコマンドで変更できるかの一覧です。

| 機能 | 必要フラグ（すべて必要） | 設定コマンド | 説明 |
|---|---|---|---|
| セキュリティ大元 | `enabled` | `/guardian toggle セキュリティ 大元` | Guardian 全機能の大元スイッチ |
| コンテンツフィルタ | `enabled` + `content_filter_enabled` | `/guardian toggle コンテンツフィルタ` または `/cf toggle コンテンツフィルタ (大元)` | メッセージ内容検査の大元 |
| 行数チェック | `enabled` + `content_filter_enabled` + `cf_check_lines` | `/cf toggle 行数チェック` | メッセージの行数を上限で制限 |
| 文字数チェック | `enabled` + `content_filter_enabled` + `cf_check_chars` | `/cf toggle 文字数チェック` | メッセージの文字数を上限で制限 |
| メンションチェック | `enabled` + `content_filter_enabled` + `cf_check_mentions` | `/cf toggle メンション数チェック` | メンション数を上限で制限 |
| 添付チェック | `enabled` + `content_filter_enabled` + `cf_check_attachments` | `/cf toggle 添付ファイルチェック` | 添付ファイルの数・サイズ・拡張子を検査 |
| リンクフィルタ | `enabled` + `content_filter_enabled` + `cf_check_links` | `/cf toggle リンクフィルタ` | リンクをブラックリスト・全ブロックで検査 |
| キーワードフィルタ | `enabled` + `content_filter_enabled` + `cf_check_keywords` | `/cf toggle キーワードフィルタ` | NGキーワードを検知 |
| 外部招待リンク禁止 | `enabled` + `invite_filter_enabled` | `/guardian toggle 招待リンクフィルタ` | Discord 招待リンクをブロック（独立動作） |
| スパム検知（大元） | `enabled` + `spam_detection_enabled` | `/guardian toggle スパム検知 (大元)` | スパム検知全体の大元 |
| 単独スパム | `enabled` + `spam_detection_enabled` + `solo_spam_enabled` | `/guardian toggle スパム検知 (単独)` | 1人による高速連投を検知 |
| 協調スパム | `enabled` + `spam_detection_enabled` + `coordinated_spam_enabled` | `/guardian toggle スパム検知 (協調)` | 複数人による類似メッセージを検知 |
| クロスサーバー相関 | 上記 + `cross_server_enabled` | `/guardian toggle クロスサーバー検知` | 複数サーバー間のスパムハッシュ照合 |
| Anti-Raid | `enabled` + `anti_raid_enabled` | `/guardian toggle Anti-Raid` | 短時間の大量参加を検知 |
| 招待バースト | `enabled` + `invite_spam_enabled` | `/guardian toggle 招待バースト検知` | 特定招待コードからの大量参加を検知 |
| グローバルレイド | `enabled` + `global_raid_enabled` | `/guardian toggle グローバルレイド検知` | 同一ユーザーの複数サーバー参加を検知 |
| アバターなしキック | `enabled` + `kick_no_avatar` | `/guardian toggle アバターなしキック` | アバター未設定アカウントを自動キック |
| 未認証Botブロック | `enabled` + `block_unverified_bots` | `/guardian toggle 未認証ボットブロック` | ホワイトリスト外 Bot をブロック |
| Nuke 保護 | `enabled` + `nuke_protection_enabled` | `/guardian toggle Nuke保護` | 大量削除等の破壊行為を検知 |
| リアクションスパム | `enabled` + `reaction_spam_enabled` | `/guardian toggle リアクションスパム` | 大量リアクションを検知 |
| Webhookバースト | `enabled` + `webhook_burst_enabled` | `/guardian toggle Webhookスパム` | Webhook の大量送信を検知 |
| Botメッセージスキャン | `enabled` + `scan_bot_messages` | `/guardian toggle ボットスキャン` | Bot メッセージを Detector に通す |
| Webhookメッセージスキャン | `enabled` + `scan_webhook_messages` | `/guardian toggle Webhookスキャン` | Webhook メッセージを Detector に通す |

---

## 8. よくある設定ミスまとめ

| ミス | 結果 | 解決方法 |
|---|---|---|
| `enabled` がオフ | **すべて**の機能が停止 | `/guardian toggle セキュリティ 大元 true` |
| `content_filter_enabled` がオフで `cf_check_*` をオン | コンテンツフィルタが動かない | `/cf toggle コンテンツフィルタ (大元) true` |
| `spam_detection_enabled` がオフで `solo_spam_enabled` をオン | スパム検知が動かない | `/guardian toggle スパム検知 (大元) true` |
| `spam_detection_enabled` だけオンで子が両方オフ | スパム検知が何も動かない | `/guardian toggle スパム検知 (単独) true` |
| `cf_check_keywords` をオンにしたがブラックリスト未登録 | キーワードフィルタが何も検知しない | `/gs blacklist-keyword add <パターン>` |
| `cf_check_links` をオンにしたがドメイン未登録かつ `link_block_all` がオフ | リンクフィルタが何も検知しない | `/gs blacklist-domain add <ドメイン>` または `/cf toggle リンク全ブロック true` |
| `cross_server_enabled` だけオンにした | 何も動かない（協調スパムの拡張だから） | `spam_detection_enabled` + `coordinated_spam_enabled` もオンにする |
| `global_raid_enabled` を使うのに `cross_server_enabled` が必要だと思った | 実は不要（別機能） | `global_raid_enabled` は独立して動作する |

```
enabled  ━━ すべての機能の大元スイッチ
│
├──【メッセージ処理パイプライン】──────────────────────────────────
│   ※ 以下の除外設定を通過したメッセージのみ Detector に到達する
│
│   [除外チェック（Detector に届かない）]
│   ├─ excluded_channel_ids  → 除外チャンネルのメッセージはスキップ
│   ├─ allowed_user_ids      → 許可ユーザーのメッセージはスキップ
│   ├─ allowed_role_ids      → 許可ロールのメッセージはスキップ
│   └─ honeypot_channel_id   → そのチャンネルのメッセージは即 Honeypot 処理
│                               （以降の Detector は呼ばれない）
│
│   [Detector パイプライン（優先順位順に評価）]
│   │
│   ├─ content_filter_enabled ──────────── コンテンツフィルタ（大元）
│   │   │  ※ false なら以下のサブ機能はすべて無効
│   │   │
│   │   ├─ cf_check_lines        → 行数超過チェック（max_message_lines）
│   │   ├─ cf_check_chars        → 文字数超過チェック（max_message_chars）
│   │   ├─ cf_check_mentions     → メンション数超過チェック（max_mentions）
│   │   ├─ cf_check_attachments  → 添付ファイルチェック
│   │   │     └─ max_attachments / max_attachment_size_mb
│   │   │        blocked_extensions / allowed_extensions
│   │   ├─ cf_check_links        → リンクフィルタ
│   │   │     └─ link_block_all / allowed_domains / ブラックリストドメイン
│   │   └─ cf_check_keywords     → キーワードフィルタ
│   │         └─ keyword_match_whole_word / keyword_case_sensitive
│   │            ブラックリストキーワード（正規表現対応）
│   │
│   │   ⚠️ trusted_role_ids について：
│   │      該当ロール所持者は lines/chars/mentions/attachments をバイパスする
│   │      ただし cf_check_links と cf_check_keywords は trusted_role でも適用される
│   │
│   ├─ invite_filter_enabled ────────────── 外部招待リンク禁止（完全独立）
│   │   content_filter_enabled がなくても単独で動作する
│   │
│   └─ spam_detection_enabled ──────────── スパム検知（大元）
│       │  ※ false なら以下のサブ機能はすべて無効
│       │
│       ├─ solo_spam_enabled     → 単独スパム（1人による高速連投）
│       │     └─ solo_spam_threshold / solo_spam_window_sec
│       │        アクション: spam_action_solo
│       │
│       └─ coordinated_spam_enabled → 協調スパム（複数人による類似メッセージ）
│             └─ spam_cluster_size / spam_window_sec / similarity_threshold
│                アクション: spam_action_cluster
│                │
│                └─ cross_server_enabled（オプション・独立）
│                     有効時: クロスサーバー間のスパムハッシュ照合も行う
│                     無効時: 同サーバー内の類似度チェックのみ（協調スパムは動作する）
│
├──【参加者処理パイプライン】──────────────────────────────────────
│   ※ すべて enabled = true が前提。各機能は互いに独立
│
│   ├─ block_unverified_bots  → Bot の参加をホワイトリストで制御
│   │     └─ allowed_bot_ids（ホワイトリスト登録済み Bot は通過）
│   │
│   ├─ global_raid_enabled    → グローバルレイド検知（cross_server_enabled 不要）
│   │     同一ユーザーが 60 秒以内に 3+ サーバーに参加 → 自動キック
│   │     ※ cross_server_enabled は「スパム相関」専用。レイドとは無関係
│   │
│   ├─ anti_raid_enabled      → ローカル Anti-Raid（大量参加検知）
│   │     └─ raid_join_threshold / raid_join_window_sec
│   │
│   ├─ kick_no_avatar         → アバターなしキック（anti_raid_enabled 不要）
│   ├─ min_account_age_hours  → アカウント年齢スクリーニング（anti_raid_enabled 不要）
│   │
│   └─ invite_burst_enabled   → 招待バースト検知（anti_raid_enabled 不要・完全独立）
│         特定の招待コードから短時間に大量参加 → 招待リンク削除
│         └─ invite_burst_threshold / invite_burst_window_sec
│
├──【Audit Log 監視】──────────────────────────────────────────────
│   ├─ nuke_protection_enabled → Nuke 保護
│   │     └─ destruction_threshold / destruction_window_sec
│   ├─ reaction_spam_enabled   → リアクションスパム
│   │     └─ reaction_burst_threshold / reaction_burst_window_sec
│   └─ webhook_burst_enabled   → Webhook バースト監視
│         └─ webhook_burst_threshold / webhook_burst_window_sec
│
└──【Bot・Webhook スキャン】（enabled が前提）─────────────────────
    ├─ scan_bot_messages      → Bot メッセージを Detector パイプラインに通す
    └─ scan_webhook_messages  → Webhook メッセージを Detector パイプラインに通す
```

---

## 機能ごとの必要フラグ一覧

| 機能 | 必要なフラグ（すべて必要） | 備考 |
|---|---|---|
| **行数制限** | `enabled` + `content_filter_enabled` + `cf_check_lines` | |
| **文字数制限** | `enabled` + `content_filter_enabled` + `cf_check_chars` | |
| **メンション制限** | `enabled` + `content_filter_enabled` + `cf_check_mentions` | |
| **添付ファイル制限** | `enabled` + `content_filter_enabled` + `cf_check_attachments` | |
| **リンクフィルタ** | `enabled` + `content_filter_enabled` + `cf_check_links` | `link_block_all` or ブラックリスト登録も必要 |
| **キーワードフィルタ** | `enabled` + `content_filter_enabled` + `cf_check_keywords` | ブラックリスト登録も必要 |
| **外部招待リンク禁止** | `enabled` + `invite_filter_enabled` | `content_filter_enabled` 不要 |
| **単独スパム検知** | `enabled` + `spam_detection_enabled` + `solo_spam_enabled` | |
| **協調スパム検知（ローカル）** | `enabled` + `spam_detection_enabled` + `coordinated_spam_enabled` | |
| **協調スパム検知（クロスサーバー）** | 上記 + `cross_server_enabled` | ローカルのみも動作する |
| **ローカル Anti-Raid** | `enabled` + `anti_raid_enabled` | |
| **招待バースト検知** | `enabled` + `invite_burst_enabled` | `anti_raid_enabled` 不要 |
| **グローバルレイド検知** | `enabled` + `global_raid_enabled` | `cross_server_enabled` 不要 |
| **アカウント年齢スクリーニング** | `enabled`（`min_account_age_hours` > 0） | 常時動作・独立 |
| **アバターなしキック** | `enabled` + `kick_no_avatar` | 独立 |
| **未認証 Bot ブロック** | `enabled` + `block_unverified_bots` | |
| **Honeypot** | `enabled` + `honeypot_channel_id` 設定 | |
| **Nuke 保護** | `enabled` + `nuke_protection_enabled` | |
| **リアクションスパム** | `enabled` + `reaction_spam_enabled` | |
| **Webhook バースト** | `enabled` + `webhook_burst_enabled` | |
| **Bot メッセージスキャン** | `enabled` + `scan_bot_messages` | Detector が別途必要 |
| **Webhook メッセージスキャン** | `enabled` + `scan_webhook_messages` | Detector が別途必要 |

---

## よく混同されるフラグの違い

### ① `global_raid_enabled` vs `cross_server_enabled`

これは最も誤解されやすい組み合わせです。

| フラグ | 何を制御するか | 影響する機能 |
|---|---|---|
| `global_raid_enabled` | 同一ユーザーが複数サーバーに短時間で参加することを検知 | **参加者処理のみ** |
| `cross_server_enabled` | 複数サーバーをまたぐスパムメッセージのハッシュ照合 | **スパム検知のみ** |

```
❌ 誤解: global_raid_enabled を使うには cross_server_enabled が必要
✅ 正解: 完全に別機能。どちらか片方だけ有効にしてよい
```

**例: グローバルレイド対策のみ有効にしたい場合**
```
enabled = true
global_raid_enabled = true
cross_server_enabled = false  ← 不要
```

---

### ② `invite_burst_enabled` vs `anti_raid_enabled`

| フラグ | 何を検知するか |
|---|---|
| `anti_raid_enabled` | サーバー全体への一定時間内の大量参加（N人/秒） |
| `invite_burst_enabled` | **特定の招待コード1つ**から一定時間内に大量参加 |

```
❌ 誤解: 招待バースト検知は Anti-Raid の一部
✅ 正解: 独立機能。Anti-Raid なしで招待バースト検知のみ有効にできる
```

**ユースケースの違い**:
- Anti-Raid → 「誰かが大量にユーザーを連れてくる攻撃」全般に対応
- 招待バースト → 「特定の招待リンクをばらまいて大量参加させる手口」に対応

---

### ③ `content_filter_enabled` vs `invite_filter_enabled`

| フラグ | 対象 | 独立性 |
|---|---|---|
| `content_filter_enabled` | メッセージの内容（行数・文字数・リンク・キーワード等）| 独立 |
| `invite_filter_enabled` | Discord 招待リンクの送信 | `content_filter_enabled` 不要 |

```
✅ content_filter をオフにしつつ、招待リンク禁止だけ有効にできる
```

---

### ④ `coordinated_spam_enabled` と `cross_server_enabled`

| 状態 | 動作 |
|---|---|
| `coordinated_spam_enabled = true`、`cross_server_enabled = false` | **同サーバー内**の類似メッセージのみ照合 |
| `coordinated_spam_enabled = true`、`cross_server_enabled = true` | 同サーバー + **複数サーバー間**のハッシュ照合も追加実施 |
| `coordinated_spam_enabled = false` | いずれの場合も協調スパム検知は動作しない |

```
✅ cross_server_enabled なしで、ローカルの協調スパム検知は完全に動作する
```

---

## メッセージ処理の内部フロー

メッセージが guardian に届いてから処理されるまでの流れです。

```
メッセージ受信
    ↓
[1] enabled チェック        → false なら即終了
    ↓
[2] 除外チャンネルチェック   → 対象外なら即終了
    ↓
[3] Webhook メッセージ判定
    ├─ scan_webhook_messages = false → 終了
    └─ true → Detector パイプライン → 違反時 Webhook 削除
    ↓
[4] Bot メッセージ判定
    ├─ scan_bot_messages = false → 終了
    ├─ allowed_bot_ids に含まれる → 終了
    └─ true → Detector パイプライン → 違反時 Bot キック
    ↓
[5] ユーザーホワイトリストチェック
    ├─ allowed_user_ids に含まれる → 終了
    └─ allowed_role_ids のロールを持つ → 終了
    ↓
[6] Honeypot チャンネルチェック
    └─ honeypot_channel = そのチャンネル → Honeypot 処理 → 終了
    ↓
[7] Detector パイプライン（順番に評価・最初にヒットしたもので終了）
    ├─ ContentFilter   (content_filter_enabled が必要)
    ├─ InviteFilter    (invite_filter_enabled が必要)
    └─ SpamDetector    (spam_detection_enabled が必要)
```

---

## 参加者処理の内部フロー

```
メンバー参加
    ↓
[1] enabled チェック             → false なら即終了
    ↓
[2] Bot 判定
    ├─ block_unverified_bots = false → 通過
    └─ allowed_bot_ids に含まれない → キック
    ↓
[3] グローバルレイドチェック     (global_raid_enabled が必要)
    └─ 検知した場合 → キック → 終了（以降処理なし）
    ↓
[4] ローカル Anti-Raid チェック  (anti_raid_enabled が必要)
    └─ 検知した場合 → アラート送信（自動キックなし）
    ↓
[5] 個別スクリーニング（独立・常時動作）
    ├─ アカウント年齢チェック    (min_account_age_hours > 0)
    └─ アバターなしチェック      (kick_no_avatar = true)
    ↓
[6] 招待バーストチェック         (invite_burst_enabled が必要)
    └─ 検知した場合 → 招待リンク削除・アラート送信
```

---

## 推奨セットアップパターン

### パターン A: 最小構成（スパムとコンテンツのみ）

```
enabled                    = true
content_filter_enabled     = true
cf_check_keywords          = true   ← キーワードフィルタのみ有効
cf_check_links             = true   ← リンクフィルタのみ有効
spam_detection_enabled     = true
solo_spam_enabled          = true   ← 単独スパムのみ
```

### パターン B: レイド対策重視

```
enabled                    = true
anti_raid_enabled          = true
invite_burst_enabled       = true   ← バースト招待も個別に対策
global_raid_enabled        = true   ← クロスサーバーレイドも
kick_no_avatar             = true
min_account_age_hours      = 24
```

### パターン C: フル有効化

```
enabled                    = true

# コンテンツ
content_filter_enabled     = true
# cf_check_* はすべて true（デフォルト）

# 招待
invite_filter_enabled      = true

# スパム
spam_detection_enabled     = true
solo_spam_enabled          = true
coordinated_spam_enabled   = true
cross_server_enabled       = true   ← クロスサーバー相関も

# レイド
anti_raid_enabled          = true
invite_burst_enabled       = true
global_raid_enabled        = true
kick_no_avatar             = true
min_account_age_hours      = 24

# Nuke・その他
nuke_protection_enabled    = true
reaction_spam_enabled      = true
webhook_burst_enabled      = true
```

---

## フラグ設定コマンド

```bash
# 現在の設定を確認
/guardian status

# Guardian 全体のフラグを切り替える
/guardian toggle <機能名> <true|false>

# コンテンツフィルタのサブ機能を切り替える
/cf toggle <機能名> <true|false>

# 全部まとめて有効化
/guardian toggle 【一括】全機能変更 true

# コンテンツフィルタだけ全部有効化
/cf toggle 【一括】コンテンツ全機能変更 true
```

各コマンドの詳細は以下を参照してください:
- [/guardian コマンドリファレンス](../commands/guardian.md)
- [/gs コマンドリファレンス](../commands/gs.md)
- [/cf コマンドリファレンス](../commands/cf.md)
- [/kb コマンドリファレンス](../commands/kb.md)
