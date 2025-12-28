# KokoIP API ドキュメント

KokoIP APIは、シンプル・高速・軽量なパブリックIPアドレス取得APIです。世界中で利用されている `ipify` との完全な互換性を持ち、さらに地理情報やASN情報の取得など多機能な拡張も備えています。

---

## 特徴

- **プロトコル別のエンドポイント**: IPv4/IPv6を明確に使い分け可能
- **ipify互換**: 既存の `ipify` ライブラリやスクリプトがそのまま利用可能
- **マルチフォーマット対応**: Text, JSON, JSONP, 詳細GeoJSON, 詳細GeoJSONP
- **CORS対応**: WebサイトのJavaScriptから直接呼び出し可能
- **SSL/TLS暗号化**: すべてのエンドポイントはHTTPSで保護
- **詳細な地理情報・ASN情報の取得**: KokoIP独自拡張
- **任意IPアドレス指定取得**: クエリで任意のIP情報取得
- **主要なHTTPヘッダー対応**: Cloudflare/Nginx等のリバースプロキシ環境下でも正しいIP判別
- **エラーハンドリング**: 不正なIP指定時はJSONでエラー返却

---

## API エンドポイント

| エンドポイント | プロトコル | 応答形式 |
| :--- | :--- | :--- |
| `https://ipv4.kokonatsu.top` | **IPv4のみ** | Text, JSON, JSONP, jsongeo, jsonpgeo |
| `https://ipv6.kokonatsu.top` | **IPv6優先** | Text, JSON, JSONP, jsongeo, jsonpgeo |

---

## パラメータ一覧

| パラメータ | 説明 | 例 |
| :--- | :--- | :--- |
| `format` | レスポンス形式 (`text`, `json`, `jsonp`, `jsongeo`, `jsonpgeo`) | `format=jsongeo` |
| `callback` | JSONP時のコールバック関数名 | `callback=getMyIP` |
| `ip` | 任意のIPアドレス指定 | `ip=8.8.8.8` |

---

## 使い方

### 1. プレーンテキスト形式（ipify互換）

```
curl https://ipv4.kokonatsu.top
```

### 2. JSON形式（ipify互換）

```
curl "https://ipv4.kokonatsu.top?format=json"
```
**Response:**
```json
{"ip":"123.456.78.9"}
```

### 3. JSONP形式（ipify互換）

```
curl "https://ipv4.kokonatsu.top?format=jsonp&callback=getMyIP"
```
**Response:**
```javascript
getMyIP({"ip":"123.456.78.9"});
```

### 4. 詳細な地理情報・ASN情報の取得（KokoIP拡張）

#### JSON形式
```
curl "https://ipv4.kokonatsu.top?format=jsongeo"
```
**Response:**
```json
{
  "ip": "8.8.8.8",
  "continent": "North America",
  "country": "United States",
  "country_code": "US",
  "region": null,
  "city": null,
  "postal_code": null,
  "latitude": 37.751,
  "longitude": -97.822,
  "timezone": "America/Chicago",
  "asn": 15169,
  "organization": "GOOGLE"
}
```

#### JSONP形式
```
curl "https://ipv4.kokonatsu.top?format=jsonpgeo&callback=cb"
```
**Response:**
```javascript
cb({
  "ip": "8.8.8.8",
  ...
});
```

### 5. 任意IPアドレスの指定

```
curl "https://ipv4.kokonatsu.top?ip=1.1.1.1&format=jsongeo"
```

---

## エラーハンドリング

不正なIPアドレスが指定された場合、エラーをJSONで返します。

```
curl "https://ipv4.kokonatsu.top?ip=invalid_ip&format=json"
```
**Response:**
```json
{"error":"Invalid IP address format"}
```

---

## 開発者向け実装例

### JavaScript (Fetch API)
```javascript
async function fetchMyIP() {
    const response = await fetch('https://ipv4.kokonatsu.top?format=json');
    const data = await response.json();
    console.log('Your Public IPv4:', data.ip);
}
fetchMyIP();
```

### Python
```python
import requests
ip = requests.get('https://ipv4.kokonatsu.top').text
print(f'My Public IP is: {ip}')
```

---

## 技術仕様

- **Dual-Stack Connectivity**: IPv4/IPv6両対応
- **High Availability**: 高速な応答速度を維持する最適化バックエンド
- **Privacy Focus**: IP確認以外の目的でデータを保持しません
- **MaxMind GeoLite2 DB利用**: 最新の地理情報・ASN情報を提供
- **主要なHTTPヘッダー対応**: `cf-connecting-ip`, `x-forwarded-for` などからクライアントIPを自動判別

---

## ライセンス

- 本APIはMaxMind GeoLite2データベースを利用しています。
- GeoLite2データベースのライセンス: https://www.maxmind.com/en/geolite2/eula

---

## お問い合わせ

ご質問・ご要望は mailto:support@kokonatsu.top までご連絡ください。
