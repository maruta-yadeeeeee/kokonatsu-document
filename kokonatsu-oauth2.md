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

- **OpenID Configuration**: `https://kokonatsu.top/.well-known/openid-configuration`
- **JWKS (JSON Web Key Set)**: `https://kokonatsu.top/.well-known/jwks.json`

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
| `openid` | **必須**。OIDC認証を行います。 | ユーザーID (`sub`) |
| `profile` | プロフィール情報へのアクセス。 | 表示名 (`name`), ユーザー名 (`preferred_username`), アバターURL (`picture`), 自己紹介 (`bio`) |
| `email` | メールアドレスへのアクセス。 | メールアドレス (`email`), メール確認状態 (`email_verified`) |
| `email:verified` | メール確認状態のみの確認。 | メール確認状態 (`email_verified`) ※アドレスは含まれません |
| `mfa_enabled` | 多要素認証(MFA)の状態確認。 | MFA有効状態 (`mfa_enabled`) |

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
```

- `client_id`: アプリ作成時に発行されたクライアントID
- `redirect_uri`: 事前に登録したリダイレクトURI（完全一致が必要）
- `scope`: スペース区切りのスコープリスト
- `state`: CSRF対策用のランダムな文字列。**推奨:** 推測不可能な文字列を生成してセッションに保存し、コールバック時に検証してください。

#### Step 2: 認可コードの受け取り

ユーザーが承認すると、`redirect_uri` にリダイレクトされます。

```http
GET {REDIRECT_URI}?code={AUTHORIZATION_CODE}&state={STATE}
```

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
  "access_token": "eyJhbGciOi...",
  "token_type": "bearer",
  "expires_in": 3600,
  "refresh_token": "v1.r.kT...",
  "scope": "openid profile email",
  "id_token": "eyJhbGciOi..."
}
```

### 2. PKCE (Proof Key for Code Exchange)

モバイルアプリやSPA（シングルページアプリケーション）など、Client Secretを安全に保持できない環境向けのフローです。
`code_challenge` と `code_verifier` を使用します。

- `code_challenge_method`: `S256` (推奨) または `plain`

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

### レスポンス例

```json
{
  "sub": "1234567890",
  "name": "Kokonatsu User",
  "preferred_username": "kokonatsu",
  "picture": "https://kokonatsu.top/avatars/1234567890/abcdef.png",
  "bio": "Hello, World!",
  "email": "user@example.com",
  "email_verified": true,
  "mfa_enabled": true
}
```

## アプリケーション管理

開発者は以下のURLからOAuth2アプリケーションを作成・管理できます。

- **管理ダッシュボード**: `https://kokonatsu.top/oauth/manage/apps`

### アプリ作成・編集機能
- **アプリ名**: ユーザーに表示されるアプリケーション名
- **説明**: アプリケーションの簡単な説明
- **アプリアイコン**: ユーザー認可画面に表示されるアイコン画像（PNG/JPEG, 最大2MB）
- **リダイレクトURI**: 認証後に戻るURL（複数指定可、改行区切り）
  - `https` 必須（`localhost` は例外的に許可）

### 公式バッジ
運営により信頼されたアプリケーションには「公式」バッジが付与され、認可画面等で表示されます。

## エラーレスポンス

エラーが発生した場合、以下の形式でJSONが返されます。

```json
{
  "error": "invalid_request",
  "error_description": "The request is missing a required parameter..."
}
```

主なエラーコード:
- `invalid_request`: リクエストパラメータが不正
- `invalid_client`: クライアント認証に失敗
- `invalid_grant`: 認可コードまたはリフレッシュトークンが無効・期限切れ
- `unauthorized_client`: クライアントがこのフローを許可されていない
- `unsupported_grant_type`: 指定された grant_type がサポートされていない
- `invalid_scope`: 要求されたスコープが無効
