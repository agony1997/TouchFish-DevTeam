# M8: Dashboard (Statistics & Charts)

## 需求描述

本模組實現統計儀表板功能，包含後端統計 API 與前端圖表頁面。後端提供三個聚合查詢端點：任務狀態統計摘要、優先級分佈統計與近七日完成趨勢統計。前端使用 Chart.js（透過 vue-chartjs 封裝）繪製圖表。

儀表板頁面頂部顯示四張統計卡片，分別呈現 TODO、IN_PROGRESS、DONE、CLOSED 各狀態的任務數量。下方區域顯示兩個圖表：優先級分佈圓餅圖與近七日完成趨勢折線圖。

後端統計 API 使用 JPA 聚合查詢（aggregate query）從資料庫直接計算統計數據，確保資料即時性與查詢效率。所有統計端點皆需要 JWT 身份驗證。

## 驗收標準

### 後端 API

- AC1: 攜帶有效 Token 呼叫 GET /api/dashboard/summary，回傳 HTTP 200 且回應包含 todo、inProgress、done、closed 四個欄位，各為對應狀態的任務數量（整數）
- AC2: GET /api/dashboard/summary 回傳的數量僅計算未軟刪除的任務
- AC3: 系統中沒有任何任務時，GET /api/dashboard/summary 回傳所有欄位皆為 0
- AC4: 攜帶有效 Token 呼叫 GET /api/dashboard/priority-distribution，回傳 HTTP 200 且回應為陣列，每個元素包含 priority 與 count 欄位
- AC5: GET /api/dashboard/priority-distribution 回傳的陣列包含 HIGH、MEDIUM、LOW 三個項目，各自統計對應優先級的任務數量
- AC6: 某優先級無任何任務時，該項目的 count 為 0（仍須回傳該項目）
- AC7: 攜帶有效 Token 呼叫 GET /api/dashboard/completion-trend，回傳 HTTP 200 且回應為陣列，每個元素包含 date 與 count 欄位
- AC8: GET /api/dashboard/completion-trend 回傳最近 7 天（含今日）的資料，每天一筆，date 格式為 yyyy-MM-dd，count 為該日轉換為 DONE 狀態的任務數量
- AC9: 某天沒有任何任務完成時，該日的 count 為 0（仍須回傳該日期項目）
- AC10: 不攜帶 Token 呼叫任何統計端點，回傳 HTTP 401 Unauthorized
- AC11: 所有統計端點僅計算未軟刪除的任務

### 前端頁面

- AC12: 進入 /dashboard 路由時，頁面自動載入並顯示四張統計卡片：TODO 數量、IN_PROGRESS 數量、DONE 數量、CLOSED 數量
- AC13: 每張統計卡片顯示狀態名稱、對應的數量數值與代表色（TODO 灰色、IN_PROGRESS 藍色、DONE 綠色、CLOSED 紅色）
- AC14: 頁面下方區域顯示優先級分佈圓餅圖（Pie Chart），圖表包含 HIGH、MEDIUM、LOW 三個扇區與對應數量標籤
- AC15: 圓餅圖扇區顏色：HIGH=#EF4444（紅）、MEDIUM=#F59E0B（橙）、LOW=#10B981（綠）
- AC16: 頁面下方區域顯示近七日完成趨勢折線圖（Line Chart），X 軸為日期（最近 7 天），Y 軸為完成數量
- AC17: 折線圖的 Y 軸起始值為 0，刻度為整數
- AC18: API 請求進行中時，頁面顯示載入中狀態（loading indicator）
- AC19: API 請求失敗時，頁面顯示錯誤提示訊息

## API 端點

### GET /api/dashboard/summary
- 說明: 取得各狀態任務數量統計摘要
- Request Header: `Authorization: Bearer <token>`
- Success Response: 200 OK
  ```json
  {
    "todo": "int",
    "inProgress": "int",
    "done": "int",
    "closed": "int"
  }
  ```
- Error Responses: 401 (unauthorized)

### GET /api/dashboard/priority-distribution
- 說明: 取得各優先級任務數量分佈
- Request Header: `Authorization: Bearer <token>`
- Success Response: 200 OK
  ```json
  [
    { "priority": "HIGH", "count": "int" },
    { "priority": "MEDIUM", "count": "int" },
    { "priority": "LOW", "count": "int" }
  ]
  ```
- Error Responses: 401 (unauthorized)

### GET /api/dashboard/completion-trend
- 說明: 取得近七日任務完成趨勢（每日轉為 DONE 的數量）
- Request Header: `Authorization: Bearer <token>`
- Success Response: 200 OK
  ```json
  [
    { "date": "2026-02-25", "count": "int" },
    { "date": "2026-02-26", "count": "int" },
    { "date": "2026-02-27", "count": "int" },
    { "date": "2026-02-28", "count": "int" },
    { "date": "2026-03-01", "count": "int" },
    { "date": "2026-03-02", "count": "int" },
    { "date": "2026-03-03", "count": "int" }
  ]
  ```
- Error Responses: 401 (unauthorized)

## 元件結構

### DashboardPage
- 路由: /dashboard
- 說明: 儀表板頁面主元件，負責載入統計資料並傳遞給子元件
- 職責: 平行呼叫三個統計 API（Promise.all），管理頁面層級狀態（loading、error）

### StatCard
- 說明: 單一統計卡片元件，顯示狀態名稱與任務數量
- Props:
  - `label` (String, required) — 狀態名稱（如「TODO」、「IN_PROGRESS」）
  - `count` (Number, required) — 任務數量
  - `color` (String, required) — 卡片代表色（十六進位色碼）
- 顯示: 狀態名稱、數量數值（大字體）、左側或頂部色條裝飾

### PieChart
- 說明: 優先級分佈圓餅圖元件，使用 vue-chartjs 的 Pie 元件
- Props:
  - `data` (Array, required) — 優先級分佈資料，格式為 [{ priority, count }]
- 圖表配置:
  - 扇區顏色：HIGH=#EF4444、MEDIUM=#F59E0B、LOW=#10B981
  - 顯示圖例（legend）於圖表右側
  - 滑鼠懸停顯示 tooltip（優先級名稱與數量）

### LineChart
- 說明: 完成趨勢折線圖元件，使用 vue-chartjs 的 Line 元件
- Props:
  - `data` (Array, required) — 趨勢資料，格式為 [{ date, count }]
- 圖表配置:
  - X 軸：日期標籤（yyyy-MM-dd 格式）
  - Y 軸：起始值 0，刻度為整數（stepSize: 1 或 ticks.precision: 0）
  - 線條顏色：#3B82F6（藍色）
  - 資料點標記：實心圓點
  - 填充區域：線條下方半透明填充（backgroundColor: rgba(59,130,246,0.1)）

## 後端實作規格

### JPA 聚合查詢

- Summary 查詢: 使用 `SELECT status, COUNT(*) FROM task WHERE deleted = false GROUP BY status` 或等效 JPA @Query
- Priority Distribution 查詢: 使用 `SELECT priority, COUNT(*) FROM task WHERE deleted = false GROUP BY priority` 或等效 JPA @Query
- Completion Trend 查詢: 使用 `SELECT DATE(updated_at), COUNT(*) FROM task WHERE status = 'DONE' AND deleted = false AND updated_at >= :sevenDaysAgo GROUP BY DATE(updated_at)` 或等效 JPA @Query，程式端補齊無資料的日期（count=0）

### 前端圖表函式庫

- 使用 `chart.js`（^4.x）與 `vue-chartjs`（^5.x）
- 在 DashboardPage 或各圖表元件中按需引入 Chart.js 模組（tree-shaking 友善）

## 業務規則

- BR1: 所有統計數據僅計算未軟刪除（deleted=false）的任務
- BR2: Summary 端點固定回傳四個狀態的數量，即使某狀態數量為 0 也必須包含
- BR3: Priority Distribution 端點固定回傳三個優先級的數量，即使某優先級數量為 0 也必須包含
- BR4: Completion Trend 端點固定回傳最近 7 天（含今日）的資料，每天一筆，無資料的日期 count 為 0
- BR5: Completion Trend 統計的「完成」定義為任務狀態轉換為 DONE 的時間點（以 updatedAt 欄位判斷）
- BR6: 統計卡片顏色對應：TODO=#6B7280（灰）、IN_PROGRESS=#3B82F6（藍）、DONE=#10B981（綠）、CLOSED=#EF4444（紅）
- BR7: 前端同時發送三個統計 API 請求（Promise.all），提升頁面載入速度
- BR8: 圓餅圖扇區顏色對應：HIGH=#EF4444（紅）、MEDIUM=#F59E0B（橙）、LOW=#10B981（綠）

## 相依模組

- 依賴: M2 (Task CRUD) — 任務資料為統計來源、M3 (Task State Machine) — 狀態定義與完成趨勢統計
- 被依賴: 無
