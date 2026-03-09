# M4: Tag System

## 需求描述

本模組實現標籤（Tag）管理系統，支援標籤的 CRUD 操作，以及標籤與任務之間的多對多（many-to-many）關聯管理。每個標籤具有名稱與顏色屬性，可被關聯至多個任務，每個任務也可擁有多個標籤。

標籤管理包含建立、查詢全部、更新與刪除功能。刪除標籤時自動移除該標籤與所有任務的關聯。任務與標籤的關聯透過專用端點進行新增與移除操作。

任務詳情 API 應包含該任務關聯的所有標籤資訊，任務列表查詢亦支援依標籤 ID 篩選。

## 驗收標準

- AC1: 攜帶有效 Token 呼叫 POST /api/tags 帶入 name 與 color，回傳 HTTP 201 且回應包含完整標籤物件（含自動產生的 id）
- AC2: 建立標籤時未帶 color，標籤的 color 預設為 #808080
- AC3: 建立標籤時帶入已存在的 name，回傳 HTTP 409 Conflict
- AC4: 攜帶有效 Token 呼叫 GET /api/tags，回傳 HTTP 200 且回應包含所有標籤的陣列
- AC5: 攜帶有效 Token 呼叫 PUT /api/tags/{id} 帶入更新的 name 或 color，回傳 HTTP 200 且回應反映更新後的標籤內容
- AC6: 攜帶有效 Token 呼叫 DELETE /api/tags/{id}，回傳 HTTP 204 No Content 且該標籤被刪除
- AC7: 刪除標籤後，原本關聯該標籤的任務不再包含該標籤（關聯自動移除）
- AC8: 攜帶有效 Token 呼叫 POST /api/tasks/{taskId}/tags 帶入 tagIds 陣列，回傳 HTTP 200 且任務成功關聯指定的標籤
- AC9: 攜帶有效 Token 呼叫 DELETE /api/tasks/{taskId}/tags/{tagId}，回傳 HTTP 204 No Content 且該任務與標籤的關聯被移除
- AC10: 呼叫 GET /api/tasks/{id} 取得任務詳情時，回應包含 tags 欄位，列出該任務關聯的所有標籤（含 id、name、color）
- AC11: 呼叫 GET /api/tasks?tagId=xxx 篩選，僅回傳關聯了該標籤的任務
- AC12: 建立標籤時 name 為空字串或超過 50 字元，回傳 HTTP 400 Bad Request
- AC13: 建立標籤時 color 格式不符合十六進位色碼格式，回傳 HTTP 400 Bad Request
- AC14: 更新標籤 name 為已被其他標籤使用的名稱，回傳 HTTP 409 Conflict
- AC15: 不攜帶 Token 呼叫任何標籤端點，回傳 HTTP 401 Unauthorized
- AC16: 對同一任務重複關聯相同標籤，不應產生重複關聯（冪等操作，回傳 200）

## API 端點

### POST /api/tags
- 說明: 建立新標籤
- Request Header: `Authorization: Bearer <token>`
- Request Body:
  ```json
  {
    "name": "string (required, 1-50 chars, unique)",
    "color": "string (optional, hex color, default: #808080)"
  }
  ```
- Success Response: 201 Created
  ```json
  {
    "id": "long",
    "name": "string",
    "color": "string (hex color)"
  }
  ```
- Error Responses: 400 (validation failed), 401 (unauthorized), 409 (duplicate name)

### GET /api/tags
- 說明: 取得所有標籤列表
- Request Header: `Authorization: Bearer <token>`
- Success Response: 200 OK
  ```json
  [
    {
      "id": "long",
      "name": "string",
      "color": "string (hex color)"
    }
  ]
  ```
- Error Responses: 401 (unauthorized)

### PUT /api/tags/{id}
- 說明: 更新標籤資訊
- Request Header: `Authorization: Bearer <token>`
- Path Parameter: id (long) — 標籤 ID
- Request Body:
  ```json
  {
    "name": "string (required, 1-50 chars, unique)",
    "color": "string (optional, hex color)"
  }
  ```
- Success Response: 200 OK（回應格式同建立）
- Error Responses: 400 (validation failed), 401 (unauthorized), 404 (not found), 409 (duplicate name)

### DELETE /api/tags/{id}
- 說明: 刪除標籤（連帶移除所有任務關聯）
- Request Header: `Authorization: Bearer <token>`
- Path Parameter: id (long) — 標籤 ID
- Success Response: 204 No Content
- Error Responses: 401 (unauthorized), 404 (not found)

### POST /api/tasks/{taskId}/tags
- 說明: 為任務新增標籤關聯（支援批次）
- Request Header: `Authorization: Bearer <token>`
- Path Parameter: taskId (long) — 任務 ID
- Request Body:
  ```json
  {
    "tagIds": [1, 2, 3]
  }
  ```
- Success Response: 200 OK
  ```json
  {
    "id": "long",
    "title": "string",
    "tags": [
      { "id": "long", "name": "string", "color": "string" }
    ]
  }
  ```
- Error Responses: 401 (unauthorized), 404 (task or tag not found)

### DELETE /api/tasks/{taskId}/tags/{tagId}
- 說明: 移除任務與標籤的關聯
- Request Header: `Authorization: Bearer <token>`
- Path Parameter: taskId (long) — 任務 ID, tagId (long) — 標籤 ID
- Success Response: 204 No Content
- Error Responses: 401 (unauthorized), 404 (task or tag not found)

## 業務規則

- BR1: 標籤名稱（name）為必填，長度限制 1-50 字元
- BR2: 標籤名稱在系統中必須唯一，重複名稱回傳 409 Conflict
- BR3: 標籤顏色（color）採用十六進位色碼格式（正則: `^#[0-9A-Fa-f]{6}$`），未指定時預設為 #808080
- BR4: 刪除標籤時，自動級聯刪除（cascade delete）該標籤與所有任務的關聯記錄
- BR5: 任務與標籤為多對多關係，同一組合不可重複（重複新增視為冪等操作，不報錯）
- BR6: 任務詳情回應中必須包含 tags 欄位，列出所有關聯標籤的 id、name、color
- BR7: 任務列表查詢支援 tagId 參數篩選，僅回傳關聯了指定標籤的任務

## 相依模組

- 依賴: M1 (Authentication)、M2 (Task CRUD) — 標籤需關聯至既有任務，且所有端點需要身份驗證
- 被依賴: M6 (Task Filter) — 進階篩選需使用標籤條件
