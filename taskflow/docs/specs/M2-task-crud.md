# M2: Task CRUD

## 需求描述

本模組提供任務（Task）的完整 CRUD 操作，包含建立、讀取、更新與刪除功能，以及支援分頁查詢、關鍵字搜尋與篩選條件的任務列表端點。所有端點皆需要通過 JWT 身份驗證。

每個任務包含標題、描述、優先級等欄位。建立任務時自動記錄建立者（createdBy）與建立時間（createdAt），更新時自動記錄更新時間（updatedAt）。刪除採用軟刪除策略，設置 deleted 標記而非實際從資料庫移除記錄。

任務列表支援分頁、排序、依標題關鍵字搜尋，以及依優先級篩選，確保大量資料時的查詢效能與使用者體驗。

## 驗收標準

- AC1: 攜帶有效 Token 呼叫 POST /api/tasks 帶入 title、description、priority，回傳 HTTP 201 且回應包含完整任務物件（含自動產生的 id、createdBy、createdAt）
- AC2: 建立任務時未帶 priority，任務的 priority 預設為 MEDIUM
- AC3: 建立任務時未帶 title 或 title 為空字串，回傳 HTTP 400 Bad Request
- AC4: 攜帶有效 Token 呼叫 GET /api/tasks/{id} 帶入存在的任務 ID，回傳 HTTP 200 且回應包含完整任務物件
- AC5: 呼叫 GET /api/tasks/{id} 帶入不存在的任務 ID，回傳 HTTP 404 Not Found
- AC6: 攜帶有效 Token 呼叫 PUT /api/tasks/{id} 帶入更新欄位，回傳 HTTP 200 且回應反映更新後的內容，updatedAt 欄位有值
- AC7: 攜帶有效 Token 呼叫 DELETE /api/tasks/{id}，回傳 HTTP 204 No Content，該任務被軟刪除
- AC8: 軟刪除的任務不出現在 GET /api/tasks 的列表查詢結果中
- AC9: 呼叫 GET /api/tasks/{id} 存取已軟刪除的任務，回傳 HTTP 404 Not Found
- AC10: 呼叫 GET /api/tasks 不帶任何參數，回傳 HTTP 200 且回應為分頁格式，預設 page=0、size=20、按 createdAt 降序排列
- AC11: 呼叫 GET /api/tasks?keyword=xxx，僅回傳 title 包含該關鍵字的任務（不區分大小寫）
- AC12: 呼叫 GET /api/tasks?priority=HIGH，僅回傳 priority 為 HIGH 的任務
- AC13: 呼叫 GET /api/tasks?page=1&size=5，回傳正確的第二頁資料（跳過前 5 筆）
- AC14: 不攜帶 Token 呼叫任何任務端點，回傳 HTTP 401 Unauthorized
- AC15: title 超過 200 字元時，回傳 HTTP 400 Bad Request
- AC16: description 超過 5000 字元時，回傳 HTTP 400 Bad Request

## API 端點

### POST /api/tasks
- 說明: 建立新任務
- Request Header: `Authorization: Bearer <token>`
- Request Body:
  ```json
  {
    "title": "string (required, 1-200 chars)",
    "description": "string (optional, max 5000 chars)",
    "priority": "string (optional, enum: HIGH|MEDIUM|LOW, default: MEDIUM)"
  }
  ```
- Success Response: 201 Created
  ```json
  {
    "id": "long",
    "title": "string",
    "description": "string",
    "priority": "HIGH|MEDIUM|LOW",
    "status": "TODO",
    "createdBy": "long (user id)",
    "createdAt": "ISO 8601 datetime",
    "updatedAt": "ISO 8601 datetime or null"
  }
  ```
- Error Responses: 400 (validation failed), 401 (unauthorized)

### GET /api/tasks/{id}
- 說明: 取得單一任務詳情
- Request Header: `Authorization: Bearer <token>`
- Path Parameter: id (long) — 任務 ID
- Success Response: 200 OK（回應格式同上）
- Error Responses: 401 (unauthorized), 404 (not found or soft-deleted)

### PUT /api/tasks/{id}
- 說明: 更新任務內容
- Request Header: `Authorization: Bearer <token>`
- Path Parameter: id (long) — 任務 ID
- Request Body:
  ```json
  {
    "title": "string (required, 1-200 chars)",
    "description": "string (optional, max 5000 chars)",
    "priority": "string (optional, enum: HIGH|MEDIUM|LOW)"
  }
  ```
- Success Response: 200 OK（回應格式同建立，updatedAt 更新）
- Error Responses: 400 (validation failed), 401 (unauthorized), 404 (not found)

### DELETE /api/tasks/{id}
- 說明: 軟刪除任務
- Request Header: `Authorization: Bearer <token>`
- Path Parameter: id (long) — 任務 ID
- Success Response: 204 No Content
- Error Responses: 401 (unauthorized), 404 (not found)

### GET /api/tasks
- 說明: 分頁查詢任務列表（支援搜尋與篩選）
- Request Header: `Authorization: Bearer <token>`
- Query Parameters:
  - `page` (int, optional, default: 0) — 頁碼，從 0 開始
  - `size` (int, optional, default: 20) — 每頁筆數
  - `sort` (string, optional, default: "createdAt,desc") — 排序欄位與方向
  - `keyword` (string, optional) — 依標題搜尋的關鍵字（不區分大小寫，模糊比對）
  - `priority` (string, optional, enum: HIGH|MEDIUM|LOW) — 依優先級篩選
- Success Response: 200 OK
  ```json
  {
    "content": [{ "id": 1, "title": "...", "..." : "..." }],
    "page": 0,
    "size": 20,
    "totalElements": 100,
    "totalPages": 5
  }
  ```
- Error Responses: 401 (unauthorized)

## 業務規則

- BR1: title 為必填欄位，長度限制 1-200 字元
- BR2: description 為選填欄位，最大長度 5000 字元
- BR3: priority 為列舉值，僅接受 HIGH、MEDIUM、LOW 三種值，未指定時預設為 MEDIUM
- BR4: 建立任務時自動從 JWT Token 中擷取 createdBy（使用者 ID），不接受前端傳入
- BR5: 建立任務時自動記錄 createdAt 為伺服器當前時間
- BR6: 更新任務時自動記錄 updatedAt 為伺服器當前時間
- BR7: 刪除為軟刪除（設置 deleted=true 或 deletedAt 時間戳），不實際移除資料庫記錄
- BR8: 所有列表查詢自動排除已軟刪除的任務
- BR9: 分頁預設值：page=0、size=20、sort=createdAt,desc
- BR10: 新建任務的初始狀態為 TODO（由 M3 模組定義）

## 相依模組

- 依賴: M1 (Authentication) — 所有端點需要 JWT 身份驗證
- 被依賴: M3 (Task State Machine)、M4 (Tag System)、M6 (Task Filter)、M7 (Dashboard)
