# M1: Authentication (JWT)

## 需求描述

本模組提供基於 JWT（JSON Web Token）的使用者身份驗證機制，包含註冊、登入與取得當前使用者資訊三項核心功能。所有需要身份驗證的 API 端點皆透過 JWT Bearer Token 進行授權驗證。

使用者註冊時須提供唯一的使用者名稱、密碼與顯示名稱。密碼必須使用 bcrypt 雜湊後儲存，禁止以明文形式保存。登入成功後回傳 JWT Token，Token 包含使用者 ID 與過期時間資訊。

受保護的端點需在 HTTP Header 中攜帶 `Authorization: Bearer <token>`，伺服器驗證 Token 的有效性與過期狀態後才允許存取。

## 驗收標準

- AC1: 呼叫 POST /api/auth/register 帶入合法的 username、password、displayName，回傳 HTTP 201 且回應包含新建使用者的 id、username、displayName
- AC2: 呼叫 POST /api/auth/register 帶入已存在的 username，回傳 HTTP 409 Conflict 且回應包含錯誤訊息
- AC3: 儲存於資料庫中的 password 欄位為 bcrypt 雜湊值，非明文密碼
- AC4: 呼叫 POST /api/auth/login 帶入正確的 username 與 password，回傳 HTTP 200 且回應包含 token（非空字串）和 expiresIn 欄位
- AC5: 呼叫 POST /api/auth/login 帶入錯誤的 password，回傳 HTTP 401 Unauthorized
- AC6: 呼叫 POST /api/auth/login 帶入不存在的 username，回傳 HTTP 401 Unauthorized
- AC7: 攜帶有效 JWT Token 呼叫 GET /api/auth/me，回傳 HTTP 200 且回應包含 id、username、displayName
- AC8: 不攜帶 Authorization Header 呼叫 GET /api/auth/me，回傳 HTTP 401 Unauthorized
- AC9: 攜帶過期的 JWT Token 呼叫 GET /api/auth/me，回傳 HTTP 401 Unauthorized
- AC10: 攜帶格式錯誤的 JWT Token 呼叫 GET /api/auth/me，回傳 HTTP 401 Unauthorized
- AC11: 註冊時 username 少於 3 字元或超過 50 字元，回傳 HTTP 400 Bad Request
- AC12: 註冊時 password 少於 8 字元，回傳 HTTP 400 Bad Request
- AC13: 註冊時 displayName 為空或超過 100 字元，回傳 HTTP 400 Bad Request
- AC14: 註冊時 username 包含非英數字或底線的字元（如空格、特殊符號），回傳 HTTP 400 Bad Request

## API 端點

### POST /api/auth/register
- 說明: 註冊新使用者
- Request Body:
  ```json
  {
    "username": "string (3-50 chars, alphanumeric + underscore)",
    "password": "string (min 8 chars)",
    "displayName": "string (1-100 chars)"
  }
  ```
- Success Response: 201 Created
  ```json
  {
    "id": "long",
    "username": "string",
    "displayName": "string"
  }
  ```
- Error Responses: 400 (validation failed), 409 (username already exists)

### POST /api/auth/login
- 說明: 使用者登入，取得 JWT Token
- Request Body:
  ```json
  {
    "username": "string",
    "password": "string"
  }
  ```
- Success Response: 200 OK
  ```json
  {
    "token": "string (JWT)",
    "expiresIn": "number (seconds, 86400)"
  }
  ```
- Error Responses: 401 (wrong credentials)

### GET /api/auth/me
- 說明: 取得當前已登入使用者資訊
- Request Header: `Authorization: Bearer <token>`
- Success Response: 200 OK
  ```json
  {
    "id": "long",
    "username": "string",
    "displayName": "string"
  }
  ```
- Error Responses: 401 (missing/invalid/expired token)

## 業務規則

- BR1: username 長度限制 3-50 字元，僅允許英文字母、數字與底線（正則: `^[a-zA-Z0-9_]{3,50}$`）
- BR2: password 最少 8 字元，無上限限制
- BR3: displayName 長度限制 1-100 字元，允許任意 Unicode 字元
- BR4: password 使用 bcrypt 演算法雜湊後儲存，不得以明文儲存
- BR5: JWT Token 有效期為 24 小時（86400 秒）
- BR6: username 在系統中必須唯一（不區分大小寫或區分大小寫皆可，但同一字串不可重複）
- BR7: 登入失敗時不應區分「使用者不存在」與「密碼錯誤」，統一回傳 401 以防止帳號列舉攻擊

## 相依模組

- 依賴: 無
- 被依賴: M2 (Task CRUD)、M3 (Task State Machine)、M4 (Tag System) — 所有需要身份驗證的模組
