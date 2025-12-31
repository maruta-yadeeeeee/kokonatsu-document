# ここなつ OAuth2 / OpenID Connect ドキュメント

このドキュメントでは、ここなつアカウントシステムが提供するOAuth2およびOpenID Connect (OIDC) プロバイダー機能について説明します。

## 概要

ここなつアカウントは、標準的なOAuth 2.0 (RFC 6749) および OpenID Connect Core 1.0 プロトコルに準拠した認証・認可基盤を提供します。
開発者は、ここなつアカウントを使用してユーザーの認証を行い、プロフィール情報にアクセスするアプリケーションを構築できます。

## ベースURL

すべてのエンドポイントは以下のベースURLからの相対パスとなります。

```
https://kokonatsu.top
```

## ディスカバリ (Discovery)

OIDC互換のライブラリを使用する場合、以下のディスカバリURLから設定情報を自動取得できます。

- **OpenID Configuration**: `https://kokonatsu.top/oauth/.well-known/openid-configuration`
- **JWKS (JSON Web Key Set)**: `https://kokonatsu.top/oauth/.well-known/jwks.json`

## エンドポイント一覧

| エンドポイント | メソッド | パス | 説明 |
|---|---|---|---|
| **Authorization** | GET | `/oauth/authorize` | ユーザーに認可を求める画面を表示します。 |
| **Token** | POST | `/oauth/token` | 認可コードをアクセストークンに交換します。 |
| **User Info** | GET/POST | `/oauth/userinfo` | アクセストークンを使用してユーザー情報を取得します。 |
| **Revocation** | POST | `/oauth/revoke` | アクセストークンまたはリフレッシュトークンを無効化します。 |
| **App Icon** | GET | `/oauth/apps/{client_id}/icon/{filename}` | アプリケーションのアイコン画像を取得します。 |

## サポートするスコープ

| スコープ | 説明 | 含まれる情報 |
|---|---|---|
| `openid` | OIDC認証を行います（ID Token発行）。純粋なOAuth2認可のみの場合は省略可能。 | ユーザーID (`sub`) |
| `profile` | プロフィール情報へのアクセス。 | 表示名 (`name`), ユーザー名 (`preferred_username`), アバターURL (`picture`), 自己紹介 (`bio`) |
| `email` | メールアドレスへのアクセス。 | メールアドレス (`email`), メール確認状態 (`email_verified`) |
| `email:verified` | メール確認状態のみの確認。 | メール確認状態 (`email_verified`) ※アドレスは含まれません |
| `mfa_enabled` | 多要素認証(MFA)の状態確認。 | MFA有効状態 (`mfa_enabled`) |

> **注意**: `openid` スコープは OIDC の場合のみ必要です。純粋な OAuth2 認可（ユーザーデータへのアクセス許可のみ）の場合、`openid` を含めなくても動作します。その場合 `id_token` は発行されません。

## トークン有効期限

| トークン種別 | 有効期限 |
|---|---|
| 認可コード (Authorization Code) | 10分 |
| アクセストークン (Access Token) | 60分（3600秒） |
| リフレッシュトークン (Refresh Token) | 30日 |
| ID Token | 1時間 |

## 認証フロー

### 1. Authorization Code Flow (推奨)

Webアプリケーション向けの標準的なフローです。

#### Step 1: 認可リクエスト

ユーザーを以下のURLにリダイレクトします。

```http
GET /oauth/authorize?
  response_type=code
  &client_id={CLIENT_ID}
  &redirect_uri={REDIRECT_URI}
  &scope=openid profile email
  &state={RANDOM_STATE_STRING}
  &nonce={RANDOM_NONCE_STRING}
```

**パラメータ:**

| パラメータ | 必須 | 説明 |
|---|---|---|
| `response_type` | ✅ | 常に `code` |
| `client_id` | ✅ | アプリ作成時に発行されたクライアントID |
| `redirect_uri` | ✅ | 事前に登録したリダイレクトURI（完全一致が必要） |
| `scope` | ✅ | スペース区切りのスコープリスト |
| `state` | 推奨 | CSRF対策用のランダムな文字列。セッションに保存し、コールバック時に検証してください。 |
| `nonce` | OIDC時推奨 | リプレイ攻撃対策。`openid` スコープ使用時は必須級。ID Tokenに含まれて返却されます。 |

#### Step 2: 認可コードの受け取り

ユーザーが承認すると、`redirect_uri` にリダイレクトされます。

```http
GET {REDIRECT_URI}?code={AUTHORIZATION_CODE}&state={STATE}
```

**重要**: `state` パラメータがリクエスト時と一致することを必ず検証してください。

#### Step 3: トークン交換

取得した `code` をアクセストークンに交換します。

```http
POST /oauth/token
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code
&code={AUTHORIZATION_CODE}
&redirect_uri={REDIRECT_URI}
&client_id={CLIENT_ID}
&client_secret={CLIENT_SECRET}
```

#### レスポンス

```json
{
  "access_token": "dG9rZW5fZXhhbXBsZV...",
  "token_type": "Bearer",
  "expires_in": 3600,
  "refresh_token": "cmVmcmVzaF90b2tlbl...",
  "scope": "openid profile email",
  "id_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCIsImtpZCI6Imtva29uYXRzdS1vaWRjIn0..."
}
```

> **注意**: `id_token` は `openid` スコープを要求した場合のみ返却されます。

### 2. PKCE (Proof Key for Code Exchange)

モバイルアプリやSPA（シングルページアプリケーション）など、Client Secretを安全に保持できない環境向けのフローです。

#### PKCEの実装手順

1. **code_verifier** を生成（43〜128文字のランダム文字列、`A-Z`, `a-z`, `0-9`, `-`, `.`, `_`, `~`）
2. **code_challenge** を計算:
   - `S256`: `BASE64URL(SHA256(code_verifier))`
   - `plain`: `code_verifier` そのまま（非推奨）

#### 認可リクエスト（PKCE）

```http
GET /oauth/authorize?
  response_type=code
  &client_id={CLIENT_ID}
  &redirect_uri={REDIRECT_URI}
  &scope=openid profile
  &state={STATE}
  &code_challenge={CODE_CHALLENGE}
  &code_challenge_method=S256
```

#### トークン交換（PKCE）

```http
POST /oauth/token
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code
&code={AUTHORIZATION_CODE}
&redirect_uri={REDIRECT_URI}
&client_id={CLIENT_ID}
&code_verifier={CODE_VERIFIER}
```

> **注意**: PKCE使用時は `client_secret` は不要です。

### 3. Refresh Token Flow

アクセストークンの有効期限が切れた場合、リフレッシュトークンを使用して新しいアクセストークンを取得できます。

```http
POST /oauth/token
Content-Type: application/x-www-form-urlencoded

grant_type=refresh_token
&refresh_token={REFRESH_TOKEN}
&client_id={CLIENT_ID}
&client_secret={CLIENT_SECRET}
```

## ユーザー情報の取得

アクセストークンを使用してユーザー情報を取得します。

```http
GET /oauth/userinfo
Authorization: Bearer {ACCESS_TOKEN}
```

または POST:

```http
POST /oauth/userinfo
Authorization: Bearer {ACCESS_TOKEN}
```

### レスポンス例

```json
{
  "sub": "550e8400-e29b-41d4-a716-446655440000",
  "name": "Kokonatsu User",
  "preferred_username": "kokonatsu",
  "picture": "https://kokonatsu.top/avatars/550e8400-e29b-41d4-a716-446655440000/abcdef.png",
  "bio": "Hello, World!",
  "email": "user@example.com",
  "email_verified": true,
  "mfa_enabled": true
}
```

> **注意**: 返却されるフィールドは要求したスコープによって異なります。

## ID Token の検証

`openid` スコープ使用時に返却される `id_token` は JWT (JSON Web Token) 形式です。

### 検証手順

1. **署名検証**: JWKS (`/.well-known/jwks.json`) から公開鍵を取得し、RS256で署名を検証
2. **発行者検証**: `iss` クレームが `https://kokonatsu.top` であることを確認
3. **対象者検証**: `aud` クレームがあなたの `client_id` と一致することを確認
4. **有効期限検証**: `exp` クレームが現在時刻より未来であることを確認
5. **nonce検証** (該当する場合): `nonce` クレームがリクエスト時に送信した値と一致することを確認

### ID Token クレーム

| クレーム | 説明 |
|---|---|
| `iss` | 発行者 (`https://kokonatsu.top`) |
| `sub` | ユーザーID（UUID形式） |
| `aud` | 対象者（Client ID） |
| `exp` | 有効期限（Unix timestamp） |
| `iat` | 発行日時（Unix timestamp） |
| `auth_time` | 認証日時（Unix timestamp） |
| `nonce` | リクエスト時に指定した nonce |

## アプリケーション管理

開発者は以下のURLからOAuth2アプリケーションを作成・管理できます。

- **管理ダッシュボード**: `https://kokonatsu.top/oauth/manage/apps`
- **認可済みアプリ管理**: `https://kokonatsu.top/oauth/manage/authorized`

### アプリ作成・編集機能

- **アプリ名**: ユーザーに表示されるアプリケーション名（3〜100文字）
- **説明**: アプリケーションの簡単な説明
- **アプリアイコン**: ユーザー認可画面に表示されるアイコン画像（PNG/JPEG, 最大2MB）
- **リダイレクトURI**: 認証後に戻るURL（複数指定可、改行区切り）
  - `https` 必須（`localhost` / `127.0.0.1` は例外的に許可）

### クライアントシークレットの再生成

セキュリティ上の理由からクライアントシークレットが漏洩した場合は、管理画面から再生成できます。
再生成後は古いシークレットは即座に無効化されます。

### 公式バッジ

運営により信頼されたアプリケーションには「公式」バッジが付与され、認可画面等で表示されます。

## エラーレスポンス

### OAuth2 標準エラー

```json
{
  "error": "invalid_request",
  "error_description": "The request is missing a required parameter..."
}
```

| エラーコード | 説明 |
|---|---|
| `invalid_request` | リクエストパラメータが不正または不足 |
| `invalid_client` | クライアント認証に失敗 |
| `invalid_grant` | 認可コードまたはリフレッシュトークンが無効・期限切れ |
| `unauthorized_client` | クライアントがこのフローを許可されていない |
| `unsupported_grant_type` | 指定された grant_type がサポートされていない |
| `invalid_scope` | 要求されたスコープが無効 |
| `access_denied` | ユーザーが認可を拒否した |

## サンプルコード

### Python (requests)

```python
import requests
import secrets

# Step 1: 認可URLの生成
client_id = "YOUR_CLIENT_ID"
redirect_uri = "https://yourapp.example.com/callback"
state = secrets.token_urlsafe(32)
nonce = secrets.token_urlsafe(32)

auth_url = (
    f"https://kokonatsu.top/oauth/authorize"
    f"?response_type=code"
    f"&client_id={client_id}"
    f"&redirect_uri={redirect_uri}"
    f"&scope=openid profile email"
    f"&state={state}"
    f"&nonce={nonce}"
)
# ユーザーを auth_url にリダイレクト

# Step 2: コールバック後、認可コードを取得
# code = request.args.get("code")
# received_state = request.args.get("state")
# assert received_state == state  # CSRF検証

# Step 3: トークン交換
token_response = requests.post(
    "https://kokonatsu.top/oauth/token",
    data={
        "grant_type": "authorization_code",
        "code": code,
        "redirect_uri": redirect_uri,
        "client_id": client_id,
        "client_secret": "YOUR_CLIENT_SECRET"
    }
)
tokens = token_response.json()
access_token = tokens["access_token"]

# Step 4: ユーザー情報取得
userinfo = requests.get(
    "https://kokonatsu.top/oauth/userinfo",
    headers={"Authorization": f"Bearer {access_token}"}
).json()

print(f"ログインユーザー: {userinfo['name']} ({userinfo['email']})")
```

### JavaScript (Node.js)

```javascript
const crypto = require('crypto');

// PKCE用 code_verifier と code_challenge の生成
function generatePKCE() {
    const verifier = crypto.randomBytes(32).toString('base64url');
    const challenge = crypto
        .createHash('sha256')
        .update(verifier)
        .digest('base64url');
    return { verifier, challenge };
}

const { verifier, challenge } = generatePKCE();

// 認可URL生成
const authUrl = new URL('https://kokonatsu.top/oauth/authorize');
authUrl.searchParams.set('response_type', 'code');
authUrl.searchParams.set('client_id', 'YOUR_CLIENT_ID');
authUrl.searchParams.set('redirect_uri', 'https://yourapp.example.com/callback');
authUrl.searchParams.set('scope', 'openid profile');
authUrl.searchParams.set('state', crypto.randomBytes(16).toString('hex'));
authUrl.searchParams.set('code_challenge', challenge);
authUrl.searchParams.set('code_challenge_method', 'S256');

console.log('認可URL:', authUrl.toString());
// verifier はセッションに保存しておき、トークン交換時に使用
```

## よくある質問 (FAQ)

### Q: `openid` スコープは必須ですか？

**A:** いいえ。`openid` スコープはOIDC（ID Token発行）が必要な場合のみ指定してください。
純粋なOAuth2認可（Access Tokenでユーザー情報を取得するだけ）の場合は不要です。

### Q: PKCE と Client Secret は同時に使えますか？

**A:** PKCE を使用する場合、Client Secret は不要です。
どちらか一方で認証されます。

### Q: リフレッシュトークンはどこに保存すべきですか？

**A:** リフレッシュトークンは長期間有効（30日）なため、安全な場所（サーバーサイドのセッションストア、暗号化されたデータベースなど）に保存してください。
クライアントサイド（LocalStorage、Cookie）への保存は避けてください。

### Q: アクセストークンが無効になった場合は？

**A:** アクセストークンは60分で期限切れになります。
リフレッシュトークンを使用して新しいアクセストークンを取得してください。
