---
id: invite-filter
sidebar_position: 7
title: 招待リンクフィルタ
sidebar_label: 招待リンクフィルタ
---

# 招待リンクフィルタ

外部サーバーへの招待リンクの送信を禁止します。

**必要なフラグ**: `enabled` + `invite_filter_enabled`  
`content_filter_enabled` は不要です（独立機能）。

---

## 動作

1. メッセージ内の Discord 招待リンク（`discord.gg/...` 等）を検出します
2. リンク先のサーバー ID を解決します（最大 3 リンク/メッセージを並行処理）
3. **自サーバー以外へ**の招待リンクであれば違反として処理します

自サーバー自身の招待リンクは常に許可されます。

---

## 外部招待キャッシュ

招待コードの解決結果はキャッシュされ、同じコードへの再チェックを省略します。

| 設定 | デフォルト | 説明 |
|---|---|---|
| `external_invite_ttl_days` | 3 | キャッシュの有効期限（日）|

```
/gs set-invite-ttl <days>
```

---

## アクションモード

`invite_action` で設定します。詳細は [アクションモード](./action-modes) を参照してください。

---

## 設定コマンド

```
/guardian toggle 招待リンクフィルタ    # invite_filter_enabled ON/OFF
/guardian set-action 招待リンク <モード>
```
