# KokoIP API

KokoIP APIは、シンプル、高速、かつ軽量なパブリックIPアドレス取得APIです。
世界中で利用されている `ipify` との完全な互換性を持ち、あらゆる環境からパブリックIPを素早く特定するために設計されています。

## 特徴

- **プロトコル別のエンドポイント**: IPv4とIPv6を確実に使い分け可能。
- **ipify 互換**: 既存の `ipify` 向けライブラリやスクリプトをそのまま利用可能。
- **マルチフォーマット対応**: プレーンテキスト、JSON、JSONP形式をサポート。
- **CORS (Cross-Origin Resource Sharing) 対応**: WebサイトのJavaScriptから直接呼び出し可能。
- **SSL/TLS 暗号化**: すべてのエンドポイントはHTTPSで保護されています。

---

## API エンドポイント

| エンドポイント | プロトコル | 応答形式 |
| :--- | :--- | :--- |
| `https://ipv4.kokonatsu.top` | **IPv4のみ** | Text, JSON, JSONP |
| `https://ipv6.kokonatsu.top` | **IPv6優先** | Text, JSON, JSONP |

---

## 使い方

### 1. プレーンテキスト形式
デフォルトのレスポンスです。IPアドレスのみを返します。

```bash
# IPv4アドレスの取得
curl https://ipv4.kokonatsu.top

# IPv6アドレスの取得
curl https://ipv6.kokonatsu.top
```

### 2. JSON形式
`format=json` パラメータを付与します。

```bash
curl "https://ipv4.kokonatsu.top?format=json"
```
**Response:**
```json
{"ip":"123.456.78.9"}
```

### 3. JSONP形式
`format=jsonp` パラメータを使用します。JavaScriptでのクロスドメイン呼び出しに利用できます。

```bash
# デフォルト (callback)
curl "https://ipv4.kokonatsu.top?format=jsonp"

# コールバック名の指定
curl "https://ipv4.kokonatsu.top?format=jsonp&callback=getMyIP"
```
**Response:**
```javascript
getMyIP({"ip":"123.456.78.9"});
```

---

## 開発者向け実装例

### JavaScript (Fetch API)

Webサイトに訪問者のIPアドレスを表示する例です。

```javascript
async function fetchMyIP() {
    try {
        // IPv4アドレスの取得
        const response = await fetch('https://ipv4.kokonatsu.top?format=json');
        const data = await response.json();
        console.log('Your Public IPv4:', data.ip);
    } catch (error) {
        console.error('Error fetching IP:', error);
    }
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

- **Dual-Stack Connectivity**: 接続環境に応じて最適なプロトコルを自動選択。
- **High Availability**: 高速な応答速度を維持するための最適化されたバックエンド・アーキテクチャ。
- **Privacy Focus**: 本APIは接続元のIPアドレスを確認する目的以外にデータを保持しません。
