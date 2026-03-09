# M3: Task State Machine

## 需求描述

本模組實現任務的狀態機管理，定義任務生命週期中的合法狀態與狀態轉換規則。任務具有四種狀態：TODO、IN_PROGRESS、DONE、CLOSED，每種狀態之間僅允許特定的轉換路徑。

狀態轉換透過專用的 PATCH 端點執行，系統在轉換前驗證當前狀態是否允許轉換至目標狀態。此外提供查詢端點，讓前端可取得指定任務當前可用的合法轉換列表，用於 UI 按鈕的啟用/停用控制。

CLOSED 為終止狀態，一旦進入 CLOSED 後不可再轉換至其他任何狀態。

## 驗收標準

- AC1: 新建任務的初始狀態為 TODO
- AC2: 呼叫 PATCH /api/tasks/{id}/status 將 TODO 任務轉換為 IN_PROGRESS，回傳 HTTP 200 且任務狀態更新為 IN_PROGRESS
- AC3: 呼叫 PATCH /api/tasks/{id}/status 將 IN_PROGRESS 任務轉換為 TODO，回傳 HTTP 200 且任務狀態更新為 TODO
- AC4: 呼叫 PATCH /api/tasks/{id}/status 將 IN_PROGRESS 任務轉換為 DONE，回傳 HTTP 200 且任務狀態更新為 DONE
- AC5: 呼叫 PATCH /api/tasks/{id}/status 將 DONE 任務轉換為 CLOSED，回傳 HTTP 200 且任務狀態更新為 CLOSED
- AC6: 呼叫 PATCH /api/tasks/{id}/status 嘗試將 TODO 直接轉換為 DONE，回傳 HTTP 400 Bad Request 且包含錯誤訊息說明不允許該轉換
- AC7: 呼叫 PATCH /api/tasks/{id}/status 嘗試將 CLOSED 任務轉換為任何其他狀態，回傳 HTTP 400 Bad Request
- AC8: 呼叫 PATCH /api/tasks/{id}/status 嘗試將 TODO 直接轉換為 CLOSED，回傳 HTTP 400 Bad Request
- AC9: 呼叫 PATCH /api/tasks/{id}/status 嘗試將 DONE 轉換為 TODO，回傳 HTTP 400 Bad Request
- AC10: 狀態轉換成功後，任務的 updatedAt 欄位更新為伺服器當前時間
- AC11: 呼叫 GET /api/tasks/{id}/transitions 查詢 TODO 任務，回傳 HTTP 200 且回應包含 ["IN_PROGRESS"]
- AC12: 呼叫 GET /api/tasks/{id}/transitions 查詢 IN_PROGRESS 任務，回傳 HTTP 200 且回應包含 ["TODO", "DONE"]
- AC13: 呼叫 GET /api/tasks/{id}/transitions 查詢 DONE 任務，回傳 HTTP 200 且回應包含 ["CLOSED"]
- AC14: 呼叫 GET /api/tasks/{id}/transitions 查詢 CLOSED 任務，回傳 HTTP 200 且回應包含空陣列 []
- AC15: 不攜帶 Token 呼叫狀態轉換或查詢端點，回傳 HTTP 401 Unauthorized
- AC16: 呼叫 PATCH /api/tasks/{id}/status 帶入不存在的任務 ID，回傳 HTTP 404 Not Found

## API 端點

### PATCH /api/tasks/{id}/status
- 說明: 執行任務狀態轉換
- Request Header: `Authorization: Bearer <token>`
- Path Parameter: id (long) — 任務 ID
- Request Body:
  ```json
  {
    "status": "string (enum: TODO|IN_PROGRESS|DONE|CLOSED)"
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
    "createdBy": "long",
    "createdAt": "ISO 8601 datetime",
    "updatedAt": "ISO 8601 datetime"
  }
  ```
- Error Responses: 400 (invalid transition), 401 (unauthorized), 404 (not found)

### GET /api/tasks/{id}/transitions
- 說明: 查詢任務當前可用的狀態轉換列表
- Request Header: `Authorization: Bearer <token>`
- Path Parameter: id (long) — 任務 ID
- Success Response: 200 OK
  ```json
  {
    "currentStatus": "TODO",
    "availableTransitions": ["IN_PROGRESS"]
  }
  ```
- Error Responses: 401 (unauthorized), 404 (not found)

## 業務規則

- BR1: 任務狀態列舉值共四種：TODO、IN_PROGRESS、DONE、CLOSED
- BR2: 新建任務初始狀態固定為 TODO，不可由使用者指定
- BR3: 合法的狀態轉換路徑如下：
  - TODO → IN_PROGRESS
  - IN_PROGRESS → TODO
  - IN_PROGRESS → DONE
  - DONE → CLOSED
- BR4: 除上述四條路徑外，其餘所有狀態轉換皆為非法，應回傳 400 錯誤
- BR5: CLOSED 為終止狀態（terminal state），進入後不可轉換至任何其他狀態
- BR6: 每次成功的狀態轉換必須更新 updatedAt 欄位為伺服器當前時間
- BR7: 狀態轉換不影響任務的其他欄位（title、description、priority 等）

## 相依模組

- 依賴: M1 (Authentication)、M2 (Task CRUD) — 基於既有任務資料進行狀態管理
- 被依賴: M7 (Dashboard) — 儀表板需依狀態統計任務數量
