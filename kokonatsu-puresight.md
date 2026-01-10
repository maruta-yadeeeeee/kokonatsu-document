# Kokonatsu PureSight API ドキュメント

Kokonatsu PureSight APIは、最新のAI技術を活用した高精度なコンテンツモデレーションAPIです。画像（NSFW検知）およびテキスト（有毒性検知）に対応し、Webアプリケーションやコミュニティサイトの健全性を守ります。

---

## 特徴

- **マルチモーダル対応**: 画像のNSFW判定に加え、テキストの不適切コンテンツ（Toxic）判定も可能。
- **柔軟なモデル選択**:
  - **NSFW V1 (画像)**: 軽量・高速なモデル。
  - **NSFW V2 (画像)**: 一般的な検知に適した高精度なモデル。
  - **Toxic V1 (テキスト)**: 日本語に対応した不適切発言検知モデル。
- **一元化された制限管理**: Webダッシュボードからの利用とAPI経由の利用は、同一アカウントの「使用回数」として同期・共有されます。

---

## ベースURL

```
https://puresight.kokonatsu.top
```

---

## 認証と制限

### 1. APIキー認証 (推奨)
ヘッダーにAPIキーを含めることで、アカウントに紐付いたレート制限で利用できます。

```http
X-API-Key: YOUR_API_KEY
```

---

## エンドポイントと使用例

### 1. テキスト解析

**POST** `/api/predict/text/{model_name}`

テキストに含まれる有害性（Toxic/NSFW）を判定します。

**URLパラメータ:**
- `model_name`: `toxic_v1` など

**使用例 (cURL):**

```bash
curl -X POST "https://puresight.kokonatsu.top/api/predict/text/toxic_v1" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: YOUR_API_KEY" \
  -d '{
    "text": "判定したいテキスト"
  }'
```

**使用例 (Python):**

```python
import urllib.request, json
req = urllib.request.Request(
    "https://puresight.kokonatsu.top/api/predict/text/toxic_v1",
    data=json.dumps({"text": "判定したいテキスト"}).encode(),
    headers={"Content-Type": "application/json", "X-API-Key": "YOUR_KEY"},
    method="POST"
)
with urllib.request.urlopen(req) as res:
    print(json.load(res))
```

**レスポンス例:**

```json
{
  "score": 0.98,
  "text_classification_result": "Toxic"
}
```

### 2. 画像解析

**POST** `/api/predict/picture/{model_name}`

画像をアップロードしてNSFW判定を行います。

**URLパラメータ:**
- `model_name`: `nsfw_v1` または `nsfw_v2`

**使用例 (cURL):**

```bash
curl -X POST "https://puresight.kokonatsu.top/api/predict/picture/nsfw_v2" \
  -H "X-API-Key: YOUR_API_KEY" \
  -F "file=@/path/to/image.jpg"
```

**使用例 (Python):**

```python
import urllib.request, json
boundary = "boundary"
with open("image.png", "rb") as f: data = f.read()
body = (
    f"--{boundary}\r\nContent-Disposition: form-data; name=\"file\"; filename=\"image.png\"\r\n"
    f"Content-Type: image/png\r\n\r\n"
).encode() + data + f"\r\n--{boundary}--\r\n".encode()

req = urllib.request.Request(
    "https://puresight.kokonatsu.top/api/predict/picture/nsfw_v2",
    data=body,
    headers={"Content-Type": f"multipart/form-data; boundary={boundary}", "X-API-Key": "YOUR_KEY"},
    method="POST"
)
with urllib.request.urlopen(req) as res:
    print(json.load(res))
```

**レスポンス例:**

```json
{
  "nsfw_score": 0.05,
  "sfw_score": 0.95,
  "is_NSFW": false
}
```

**※ 注: `nsfw_v1` モデルの場合**
V1モデルでは、`details` フィールドに追加の詳細判定スコアが含まれます。

```json
{
  "nsfw_score": 0.1,
  "sfw_score": 0.8,
  "is_NSFW": false,
  "details": {
    "questionable": 0.05,
    "explicit": 0.05,
    "general": 0.8,
    "sensitive": 0.1
  }
}
```

---

## ダッシュボード

ユーザーはダッシュボードから以下の機能を利用できます。

- **APIキー管理**: キーの作成・削除
- **デモ機能**: ブラウザ上で直接テキスト・画像をアップロードして解析テスト
- **使用量確認**: APIの残り使用回数を可視化
- **設定**: テーマ（ダーク/ライト）切り替え、言語（日本語/英語）切り替え

**注**: アカウントの使用回数は、ウェブ上のデモ機能とAPIキーによるアクセスの両方で共通して消費されます。

---

## エラーハンドリング

- **200 OK**: 成功
- **400 Bad Request**: パラメータ不正
- **401 Unauthorized**: 認証が必要
- **403 Forbidden**: 管理者権限が必要
- **404 Not Found**: 指定されたモデルが存在しない
- **429 Too Many Requests**: レート制限超過
- **500 Internal Server Error**: サーバー内部エラー

---

## 運営・お問い合わせ

Kokonatsu PureSightは、ここなつグループによって運営されています。
ご質問やサポートが必要な場合は、公式Discordまたはサポートメールまでお問い合わせください。
