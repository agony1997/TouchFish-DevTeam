# M5: Assignment (Task Assignee)

## 需求描述

本模組實現任務指派功能，支援將單一使用者指派為任務的負責人（assignee）。每個任務最多只能有一位負責人，可透過 API 進行指派、取消指派與查詢指定使用者的所有指派任務。

指派操作透過 PUT 端點執行，帶入目標使用者 ID 即可完成指派。若任務已有負責人，再次指派將覆蓋原負責人（冪等式重新指派）。取消指派透過 DELETE 端點執行，移除任務的負責人欄位。

任務詳情 API 回應中應包含負責人資訊（assignee），任務列表查詢亦支援依 assigneeId 參數篩選特定使用者負責的任務。此外提供依使用者 ID 查詢其所有指派任務的端點。

## 驗收標準

- AC1: 攜帶有效 Token 呼叫 PUT /api/tasks/{taskId}/assignee 帶入合法的 userId，回傳 HTTP 200 且回應包含更新後的任務物件，assignee 欄位顯示該使用者資訊
- AC2: 攜帶有效 Token 呼叫 PUT /api/tasks/{taskId}/assignee 帶入不存在的 userId，回傳 HTTP 404 Not Found 且錯誤訊息說明使用者不存在
- AC3: 攜帶有效 Token 呼叫 PUT /api/tasks/{taskId}/assignee 帶入不存在的 taskId，回傳 HTTP 404 Not Found 且錯誤訊息說明任務不存在
- AC4: 對已有負責人的任務再次呼叫 PUT /api/tasks/{taskId}/assignee 帶入相同的 userId，回傳 HTTP 200（冪等操作，負責人不變）
- AC5: 對已有負責人的任務呼叫 PUT /api/tasks/{taskId}/assignee 帶入不同的 userId，回傳 HTTP 200 且負責人更新為新使用者
- AC6: 攜帶有效 Token 呼叫 DELETE /api/tasks/{taskId}/assignee，回傳 HTTP 204 No Content 且任務的 assignee 欄位被清除
- AC7: 對沒有負責人的任務呼叫 DELETE /api/tasks/{taskId}/assignee，回傳 HTTP 204 No Content（冪等操作）
- AC8: 攜帶有效 Token 呼叫 GET /api/users/{userId}/tasks，回傳 HTTP 200 且回應包含該使用者所有被指派的任務列表
- AC9: 呼叫 GET /api/users/{userId}/tasks 帶入不存在的 userId，回傳 HTTP 404 Not Found
- AC10: 呼叫 GET /api/tasks/{id} 取得任務詳情時，回應包含 assignee 欄位，內含負責人的 id、username、displayName（若無負責人則為 null）
- AC11: 呼叫 GET /api/tasks?assigneeId=xxx 篩選，僅回傳指派給該使用者的任務
- AC12: 指派或取消指派成功後，任務的 updatedAt 欄位更新為伺服器當前時間
- AC13: 不攜帶 Token 呼叫任何指派端點，回傳 HTTP 401 Unauthorized

## API 端點

### PUT /api/tasks/{taskId}/assignee
- 說明: 指派使用者為任務負責人（若已有負責人則覆蓋）
- Request Header: `Authorization: Bearer <token>`
- Path Parameter: taskId (long) — 任務 ID
- Request Body:
  ```json
  {
    "userId": "long (required)"
  }
  ```
- Success Response: 200 OK
  ```json
  {
    "id": "long",
    "title": "string",
    "description": "string",
    "priority": "HIGH|MEDIUM|LOW",
    "status": "TODO|IN_PROGRESS|DONE|CLOSED",
    "assignee": {
      "id": "long",
      "username": "string",
      "displayName": "string"
    },
    "createdBy": "long",
    "createdAt": "ISO 8601 datetime",
    "updatedAt": "ISO 8601 datetime"
  }
  ```
- Error Responses: 401 (unauthorized), 404 (task or user not found)

### DELETE /api/tasks/{taskId}/assignee
- 說明: 取消任務的負責人指派
- Request Header: `Authorization: Bearer <token>`
- Path Parameter: taskId (long) — 任務 ID
- Success Response: 204 No Content
- Error Responses: 401 (unauthorized), 404 (task not found)

### GET /api/users/{userId}/tasks
- 說明: 查詢指定使用者被指派的所有任務
- Request Header: `Authorization: Bearer <token>`
- Path Parameter: userId (long) — 使用者 ID
- Success Response: 200 OK
  ```json
  [
    {
      "id": "long",
      "title": "string",
      "description": "string",
      "priority": "HIGH|MEDIUM|LOW",
      "status": "TODO|IN_PROGRESS|DONE|CLOSED",
      "assignee": {
        "id": "long",
        "username": "string",
        "displayName": "string"
      },
      "createdBy": "long",
      "createdAt": "ISO 8601 datetime",
      "updatedAt": "ISO 8601 datetime"
    }
  ]
  ```
- Error Responses: 401 (unauthorized), 404 (user not found)

## 業務規則

- BR1: 每個任務最多只能有一位負責人（assignee），不支援多人指派
- BR2: 任何已認證的使用者皆可執行指派操作，不限於任務建立者
- BR3: 重新指派（對已有負責人的任務指派新使用者）會直接覆蓋原負責人，無需先取消指派
- BR4: 重複指派相同使用者為冪等操作，回傳 200 且不產生副作用
- BR5: 取消無負責人任務的指派為冪等操作，回傳 204 且不產生副作用
- BR6: 任務詳情回應中必須包含 assignee 欄位，若無負責人則該欄位值為 null
- BR7: 任務列表查詢（GET /api/tasks）支援 assigneeId 查詢參數，僅回傳指派給該使用者的任務
- BR8: 已軟刪除的任務不應出現在使用者的指派任務列表中
- BR9: 指派與取消指派操作皆須更新任務的 updatedAt 欄位

## 相依模組

- 依賴: M1 (Authentication) — 所有端點需要 JWT 身份驗證、M2 (Task CRUD) — 基於既有任務資料進行指派管理
- 被依賴: M6 (Task List Page) — 任務列表顯示負責人資訊與篩選
