# Kokonatsu PureSight API ドキュメント

Kokonatsu PureSight APIは、最新のAI技術を活用した高精度なNSFW画像検知APIです。Webアプリケーション、チャットボット、コミュニティサイトなどのコンテンツモデレーションを、わずか数行のコードで自動化します。

---

## 特徴

- **高精度AIモデル**: 用途に合わせて選べる2つのモデル（V1: 高速, V2: 高精度）を提供
- **RESTful API**: シンプルで扱いやすい標準的なHTTPインターフェース
- **バッチ処理対応**: 複数の画像を一度のリクエストで効率的に解析
- **セキュアな認証**: OAuth2連携およびAPIキーによる厳格なアクセス制御
- **ユーザー管理**: 管理画面からユーザーごとのレート制限を設定可能
- **日本語対応**: エラーメッセージからドキュメントまで完全日本語サポート
- **開発者ファースト**: 詳細なドキュメントと管理ダッシュボード完備

---

## ベースURL

```
https://puresight.kokonatsu.top
```

---

## 認証

すべてのAPIリクエストには、ヘッダーにAPIキーを含める必要があります。

```http
X-API-Key: nsfw_xxxxxxxxxxxxxxxxxxxxxx
```

APIキーは [管理ダッシュボード](/dashboard) から生成できます。

---

## エンドポイント一覧

| エンドポイント | メソッド | 説明 |
| :--- | :--- | :--- |
| `/api/predict` | POST | 画像を1枚アップロードしてNSFW判定を行います。 |
| `/api/predict/batch` | POST | 複数の画像を一度にアップロードして一括判定を行います。 |
| `/api/rate-limit` | GET | 現在のレート制限の使用状況と残りの割り当て回数を確認します。 |

---

## 使い方

### 1. 画像解析 (単体)

**POST** `/api/predict`

単一の画像ファイルを送信し、NSFWスコアと判定結果を取得します。

**パラメータ:**

| パラメータ | 必須 | 説明 |
| :--- | :--- | :--- |
| `model` | いいえ | `v2` (高精度, デフォルト) または `v1` (高速) |
| `threshold` | いいえ | 判定しきい値。`0.0` - `1.0` (デフォルト: `0.5`) |
| `file` | **はい** | 解析する画像ファイル (multipart/form-data) |

**リクエスト例:**

```bash
curl -X POST "https://puresight.kokonatsu.top/api/predict?model=v2" \
  -H "X-API-Key: YOUR_KEY" \
  -F "file=@/path/to/image.jpg"
```

**レスポンス例:**

```json
{
  "model": "v2",
  "filename": "image.jpg",
  "is_nsfw": false,
  "nsfw_score": 0.05,
  "sfw_score": 0.95,
  "threshold": 0.5
}
```

### 2. 画像解析 (一括)

**POST** `/api/predict/batch`

複数の画像をまとめて解析します。レート制限は画像の枚数分消費されます。

**パラメータ:**

| パラメータ | 必須 | 説明 |
| :--- | :--- | :--- |
| `files` | **はい** | 解析する画像ファイル (複数指定可) |

**リクエスト例:**

```bash
curl -X POST "https://puresight.kokonatsu.top/api/predict/batch" \
  -H "X-API-Key: YOUR_KEY" \
  -F "files=@image1.jpg" \
  -F "files=@image2.png"
```

### 3. レート制限の確認

**GET** `/api/rate-limit`

自身のAPIキーに紐付いたレート制限の状態を確認します。

**リクエスト例:**

```bash
curl "https://puresight.kokonatsu.top/api/rate-limit" \
  -H "X-API-Key: YOUR_KEY"
```

**レスポンス例:**

```json
{
  "used": 15,
  "limit": 100,
  "remaining": 85,
  "is_authenticated": true
}
```

---

## エラーハンドリング

APIは標準的なHTTPステータスコードを返します。

- **200 OK**: 成功
- **400 Bad Request**: リクエストパラメータの不備
- **401 Unauthorized**: APIキーが無効、または不足している
- **403 Forbidden**: アクセス権限がない
- **413 Content Too Large**: ファイルサイズが大きすぎる
- **429 Too Many Requests**: レート制限を超過した
- **500 Internal Server Error**: サーバー内部エラー

---

## 運営・お問い合わせ

Kokonatsu PureSightは、ここなつグループによって運営されています。
ご質問やサポートが必要な場合は、公式Discordまたはサポートメールまでお問い合わせください。
