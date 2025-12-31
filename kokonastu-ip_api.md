# KokoIP API ドキュメント

KokoIP APIは、シンプル・高速・軽量なパブリックIPアドレス取得APIです。世界中で利用されている `ipify` との完全な互換性を持ち、さらに地理情報、ASN情報、VPN/プロキシ検知など多機能な拡張も備えています。

---

## 特徴

- **プロトコル別のエンドポイント**: IPv4/IPv6を明確に使い分け可能
- **ipify互換**: 既存の `ipify` ライブラリやスクリプトがそのまま利用可能
- **柔軟なincludeパラメータ**: 必要な情報のみを取得可能
- **VPN/プロキシ検知**: データセンターIP、VPN、ホスティングプロバイダーを自動検出
- **マルチフォーマット対応**: Text, JSON, JSONP
- **CORS対応**: WebサイトのJavaScriptから直接呼び出し可能
- **SSL/TLS暗号化**: すべてのエンドポイントはHTTPSで保護
- **任意IPアドレス指定取得**: クエリで任意のIP情報取得
- **主要なHTTPヘッダー対応**: Cloudflare/Nginx等のリバースプロキシ環境下でも正しいIP判別
- **エラーハンドリング**: 不正なIP指定時はJSONでエラー返却
- **一貫性のあるレスポンス**: 指定した情報は存在しない場合でもnullとして返却

---

## API エンドポイント

| エンドポイント | プロトコル |
| :--- | :--- |
| `https://ipv4.kokonatsu.top` | **IPv4のみ** |
| `https://ipv6.kokonatsu.top` | **IPv6優先** |

---

## パラメータ一覧

| パラメータ | 説明 | 例 |
| :--- | :--- | :--- |
| `format` | レスポンス形式 (`text`, `json`, `jsonp`) | `format=json` |
| `include` | 追加情報の指定 (`geo`, `detection`, `geo,detection`) | `include=geo,detection` |
| `callback` | JSONP時のコールバック関数名 | `callback=getMyIP` |
| `ip` | 任意のIPアドレス指定 | `ip=8.8.8.8` |

### includeパラメータの詳細

- **`geo`**: 地理情報とASN情報を含める（continent, country, city, asn, organizationなど）
- **`detection`**: VPN/プロキシ検知情報を含める（proxy_detection）
- **`geo,detection`**: 両方の情報を含める

---

## 使い方

### 1. プレーンテキスト形式（ipify互換）

```bash
curl https://ipv4.kokonatsu.top
```
**Response:**
```
123.456.78.9
```

### 2. JSON形式（ipify互換）

```bash
curl "https://ipv4.kokonatsu.top?format=json"
```
**Response:**
```json
{"ip":"123.456.78.9"}
```

### 3. JSONP形式（ipify互換）

```bash
curl "https://ipv4.kokonatsu.top?format=jsonp&callback=getMyIP"
```
**Response:**
```javascript
getMyIP({"ip":"123.456.78.9"});
```

### 4. 地理情報・ASN情報の取得

```bash
curl "https://ipv4.kokonatsu.top?format=json&include=geo"
```
**Response:**
```json
{
  "ip": "8.8.8.8",
  "continent": "North America",
  "country": "United States",
  "country_code": "US",
  "latitude": 37.751,
  "longitude": -97.822,
  "timezone": "America/Chicago",
  "asn": 15169,
  "organization": "GOOGLE"
}
```

> **Note**: `include`パラメータで指定されたフィールドは、値が取得できない場合でも `null` としてレスポンスに含まれます。これによりクライアント側での型処理が一貫します。

### 5. VPN/プロキシ検知

**VPN検出ありの例:**
```bash
curl "https://ipv4.kokonatsu.top?format=json&include=detection&ip=8.8.8.8"
```
**Response:**
```json
{
  "ip": "8.8.8.8",
  "proxy_detection": {
    "is_vpn": false,
    "is_proxy": false,
    "is_datacenter": true,
    "is_hosting": true,
    "risk_score": 90,
    "provider_name": "Google Cloud"
  }
}
```

**検出なし（家庭用IPなど）の例:**
```bash
curl "https://ipv4.kokonatsu.top?format=json&include=detection&ip=1.2.3.4"
```
**Response:**
```json
{
  "ip": "1.2.3.4",
  "proxy_detection": {
    "is_vpn": false,
    "is_proxy": false,
    "is_datacenter": false,
    "is_hosting": false,
    "risk_score": 0,
    "provider_name": null
  }
}
```

#### proxy_detectionフィールドの詳細

| フィールド | 型 | 説明 |
| :--- | :--- | :--- |
| `is_vpn` | boolean | VPNプロバイダーとして検出された場合true |
| `is_proxy` | boolean | プロキシとして検出された場合true |
| `is_datacenter` | boolean | データセンターIPとして検出された場合true |
| `is_hosting` | boolean | ホスティングプロバイダーとして検出された場合true |
| `risk_score` | 0-100 | リスクスコア（高いほど疑わしい） |
| `provider_name` | string \| null | 検出されたプロバイダー名 |

### 6. すべての情報を取得

```bash
curl "https://ipv4.kokonatsu.top?format=json&include=geo,detection&ip=8.8.8.8"
```
**Response:**
```json
{
  "ip": "8.8.8.8",
  "continent": "North America",
  "country": "United States",
  "country_code": "US",
  "latitude": 37.751,
  "longitude": -97.822,
  "timezone": "America/Chicago",
  "asn": 15169,
  "organization": "GOOGLE",
  "proxy_detection": {
    "is_vpn": false,
    "is_proxy": false,
    "is_datacenter": true,
    "is_hosting": true,
    "risk_score": 90,
    "provider_name": "Google Cloud"
  }
}
```

### 7. JSONP形式で詳細情報を取得

```bash
curl "https://ipv4.kokonatsu.top?format=jsonp&include=geo,detection&callback=cb"
```
**Response:**
```javascript
cb({
  "ip": "123.456.78.9",
  "continent": "Asia",
  "country": "Japan",
  ...
});
```

### 8. 任意IPアドレスの検証

```bash
curl "https://ipv4.kokonatsu.top?ip=1.1.1.1&format=json&include=geo,detection"
```

---

## エラーハンドリング

不正なIPアドレスが指定された場合、エラーをJSONで返します。

```bash
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
// IPアドレスのみ取得
async function fetchMyIP() {
    const response = await fetch('https://ipv4.kokonatsu.top?format=json');
    const data = await response.json();
    console.log('Your Public IPv4:', data.ip);
}

// 地理情報とVPN検知を含む完全な情報取得
async function fetchFullInfo() {
    const response = await fetch('https://ipv4.kokonatsu.top?format=json&include=geo,detection');
    const data = await response.json();
    console.log('IP:', data.ip);
    console.log('Location:', data.city, data.country);
    
    if (data.proxy_detection) {
        console.log('VPN Detected:', data.proxy_detection.is_vpn);
        console.log('Risk Score:', data.proxy_detection.risk_score);
    }
}
```

### Python
```python
import requests

# シンプルなIP取得
ip = requests.get('https://ipv4.kokonatsu.top').text
print(f'My Public IP: {ip}')

# 詳細情報取得
response = requests.get('https://ipv4.kokonatsu.top?format=json&include=geo,detection')
data = response.json()
print(f"IP: {data['ip']}")
print(f"Country: {data.get('country', 'Unknown')}")

if 'proxy_detection' in data:
    print(f"Is VPN: {data['proxy_detection']['is_vpn']}")
    print(f"Risk Score: {data['proxy_detection']['risk_score']}")
```

### cURL (詳細なVPN検証スクリプト)
```bash
#!/bin/bash
IP=$1
RESULT=$(curl -s "https://ipv4.kokonatsu.top?format=json&include=detection&ip=$IP")
IS_VPN=$(echo $RESULT | jq -r '.proxy_detection.is_vpn // false')
RISK_SCORE=$(echo $RESULT | jq -r '.proxy_detection.risk_score // 0')

echo "IP: $IP"
echo "Is VPN: $IS_VPN"
echo "Risk Score: $RISK_SCORE"
```

---

## VPN/プロキシ検知の仕組み

KokoIP APIは以下の3つのアプローチでVPN/プロキシを検知します：

1. **ASNリストマッチング**: 既知のVPN/ホスティングプロバイダーのASNとマッチング
2. **組織名キーワード分析**: Organization名に"vpn", "hosting", "datacenter"等のキーワードが含まれるかチェック
3. **地理情報整合性**: 都市情報の欠落をデータセンターIPの兆候として判定

これらの手法を組み合わせて0-100のリスクスコアを算出します。

> **注意**: この検知機能はGeoLite2データベースのみを使用しているため、商用VPN検知サービスと比較して精度が劣る可能性があります。

---

## 技術仕様

- **Dual-Stack Connectivity**: IPv4/IPv6両対応
- **High Availability**: 高速な応答速度を維持する最適化バックエンド
- **Privacy Focus**: アクセスログやリクエストログを一切保存しません。IP確認以外の目的でデータを保持・収集・記録することはありません
- **MaxMind GeoLite2 DB利用**: 最新の地理情報・ASN情報を提供
- **主要なHTTPヘッダー対応**: `cf-connecting-ip`, `x-forwarded-for` などからクライアントIPを自動判別
- **Consistent Schema**: クライアント側でのパースを容易にするため、要求されたフィールドは常に存在します（値がない場合はnull）

---

## レート制限

現在、レート制限は設定されていませんが、公正な利用をお願いします。過度なリクエストはサービスの品質に影響を与える可能性があります。

---

## ライセンス

- 本APIはMaxMind GeoLite2データベースを利用しています。
- GeoLite2データベースのライセンス: https://www.maxmind.com/en/geolite2/eula

---

## お問い合わせ

ご質問・ご要望は mailto:support@kokonatsu.top までご連絡ください。
