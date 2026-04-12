---
id: spam-detection
sidebar_position: 5
title: スパム検知
sidebar_label: スパム検知
---

# スパム検知

**必要なフラグ**: `enabled` + `spam_detection_enabled`

---

## 単独スパム（solo spam）

1人のユーザーが短時間に大量のメッセージを送信した場合に検知します。

**追加フラグ**: `solo_spam_enabled`

| 設定 | デフォルト | 説明 |
|---|---|---|
| `solo_spam_threshold` | 5 | 何件送信で検知するか |
| `solo_spam_window_sec` | 10 | 検知ウィンドウ（秒）|

アクションモード: `spam_action_solo`

---

## 協調スパム（cluster spam）

複数のユーザーが類似したメッセージを短時間に送信した場合に検知します。

**追加フラグ**: `coordinated_spam_enabled`

| 設定 | デフォルト | 説明 |
|---|---|---|
| `spam_cluster_size` | 3 | 何人以上で協調スパムと判定するか |
| `spam_window_sec` | 60 | 検知ウィンドウ（秒）|
| `similarity_threshold` | 0.90 | メッセージ類似度の閾値 (0.0〜1.0) |
| `spam_min_cjk_len` | 10 | CJK文字（日本語等）の最小判定文字数 |
| `spam_min_latin_len` | 15 | ラテン文字の最小判定文字数 |

アクションモード: `spam_action_cluster`

### クロスサーバー相関（cross_server_enabled）

`cross_server_enabled` を有効にすると、複数サーバーにまたがる完全一致メッセージのハッシュ比較を行います。

- **無効時**: 同サーバー内のメッセージのみで類似度チェック
- **有効時**: 複数サーバーのメッセージハッシュを共有プールで比較（クロスギルド判定の閾値は固定値 3）

:::note
`cross_server_enabled` はスパム相関にのみ影響します。グローバルレイド検知（`global_raid_enabled`）とは別個のフラグです。
:::

---

## アルゴリズム

### 単独スパム
トークンバケットアルゴリズムを使用。容量 = `solo_spam_threshold`、補充レート = `threshold / window_sec`。

### 協調スパム（ローカル類似度）
1. SHA-256 ハッシュによる完全一致チェック（O(1)）
2. Levenshtein 比率（`similarity_threshold`）による類似度チェック
3. CJK/Latin 混在コンテンツの最小長フィルタリング（ノイズ除去）

---

## 設定コマンド

```
/guardian toggle スパム検知 (大元)      # spam_detection_enabled
/guardian toggle スパム検知 (単独)      # solo_spam_enabled
/guardian toggle スパム検知 (協調)      # coordinated_spam_enabled
/guardian toggle クロスサーバー検知     # cross_server_enabled
/guardian set-spam <cluster_size> <window> <similarity>
/guardian set-solo-spam <threshold> <window>
```
