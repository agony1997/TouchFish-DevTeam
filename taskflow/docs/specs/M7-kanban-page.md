# M7: Kanban Page (Vue Frontend)

## 需求描述

本模組實現看板（Kanban）頁面，以四欄式看板佈局呈現任務狀態分佈。四個欄位分別對應 TODO、IN_PROGRESS、DONE、CLOSED 四種任務狀態，每個任務以卡片形式顯示於其所屬狀態欄位中。

使用者可透過 HTML5 原生 Drag and Drop API 將任務卡片從一個欄位拖曳至另一個欄位，系統自動呼叫後端 PATCH /api/tasks/{id}/status 端點執行狀態轉換。若轉換不合法（違反狀態機規則），頁面顯示錯誤提示（error toast），且卡片自動回彈至原欄位。

本模組不使用任何外部拖曳套件，完全基於瀏覽器原生 HTML5 Drag and Drop API 實現。

## 驗收標準

- AC1: 進入 /kanban 路由時，頁面顯示四個欄位，從左至右依序為 TODO、IN_PROGRESS、DONE、CLOSED
- AC2: 頁面載入時自動從後端取得所有任務（GET /api/tasks?size=1000），並依狀態分組顯示於對應欄位中
- AC3: 每個欄位標題顯示狀態名稱與該欄位中的任務數量（例如「TODO (5)」）
- AC4: 每張任務卡片顯示：任務標題（title）、優先級標籤（priority badge）、負責人名稱（assignee displayName，若無則顯示「未指派」）
- AC5: 使用者可拖曳任務卡片至其他欄位，拖曳過程中目標欄位顯示視覺提示（如邊框高亮或背景色變化）
- AC6: 拖曳放下後，系統呼叫 PATCH /api/tasks/{id}/status 帶入目標欄位對應的狀態值
- AC7: 狀態轉換成功（HTTP 200）時，卡片留在目標欄位，原欄位與目標欄位的任務數量即時更新
- AC8: 狀態轉換失敗（HTTP 400，不合法轉換）時，頁面顯示錯誤 toast 提示訊息（包含後端回傳的錯誤描述），卡片自動回彈至原欄位
- AC9: 將 TODO 任務拖曳至 DONE 欄位（非法轉換），顯示錯誤 toast 且卡片回彈至 TODO 欄位
- AC10: 將 CLOSED 任務拖曳至任何其他欄位（終止狀態不可轉換），顯示錯誤 toast 且卡片回彈至 CLOSED 欄位
- AC11: 將卡片拖曳放下至同一欄位（狀態未變），不呼叫 API、不顯示任何提示
- AC12: 優先級標籤顏色與 M6 一致：HIGH 紅色、MEDIUM 橙色、LOW 綠色
- AC13: API 請求進行中時，被拖曳的卡片顯示載入狀態（如半透明或 loading spinner）
- AC14: 不攜帶 Token 存取 /kanban 路由，重定向至登入頁面
- AC15: API 請求失敗（非 400 錯誤，如 500 伺服器錯誤）時，顯示通用錯誤 toast 且卡片回彈

## 元件結構

### KanbanPage
- 路由: /kanban
- 說明: 看板頁面主元件，負責載入任務資料並將任務依狀態分組傳遞給各欄位元件
- 職責: 管理頁面層級狀態（loading、error），處理拖曳事件的 API 呼叫與結果處理

### KanbanColumn
- 說明: 單一看板欄位元件，代表一種任務狀態
- Props:
  - `status` (String, required) — 該欄位對應的狀態值（TODO|IN_PROGRESS|DONE|CLOSED）
  - `tasks` (Array, required) — 該欄位中的任務列表
- Events:
  - `task-dropped` (Object: { taskId, fromStatus, toStatus }) — 任務卡片被拖曳放入此欄位時觸發
- 職責: 實作 HTML5 Drop Zone（dragover、dragenter、dragleave、drop 事件），管理拖曳進入時的視覺提示

### KanbanCard
- 說明: 單一任務卡片元件，可被拖曳
- Props:
  - `task` (Object, required) — 任務物件，包含 id、title、priority、assignee、status
- 職責: 實作 HTML5 Draggable（設定 draggable="true"，處理 dragstart、dragend 事件），透過 dataTransfer 傳遞任務 ID 與來源狀態

## 拖曳實作規格

### HTML5 Drag and Drop API 使用方式
- KanbanCard: 設定 `draggable="true"`，於 `dragstart` 事件中透過 `event.dataTransfer.setData()` 傳遞 taskId 與 fromStatus
- KanbanColumn: 於 `dragover` 事件中呼叫 `event.preventDefault()` 允許放置，於 `drop` 事件中透過 `event.dataTransfer.getData()` 取得 taskId 與 fromStatus
- 拖曳過程中，來源卡片加上 `opacity: 0.5` 樣式，目標欄位加上高亮邊框（`border: 2px dashed #3B82F6`）
- 拖曳結束（dragend）時恢復所有視覺樣式

### 狀態轉換流程
1. 使用者拖曳卡片至目標欄位並放下
2. 若 fromStatus === toStatus，忽略操作
3. 前端立即將卡片移至目標欄位（樂觀更新, optimistic update）
4. 呼叫 PATCH /api/tasks/{id}/status { status: toStatus }
5. 若 API 回傳 200，保留更新後狀態
6. 若 API 回傳錯誤，將卡片移回原欄位並顯示 error toast

## 業務規則

- BR1: 看板僅使用 HTML5 原生 Drag and Drop API，不引入任何外部拖曳套件（如 vuedraggable、SortableJS 等）
- BR2: 四個欄位固定順序為 TODO、IN_PROGRESS、DONE、CLOSED，不可自訂或重新排序
- BR3: 拖曳採用樂觀更新（optimistic update）策略：先更新前端 UI，再等待 API 回應，失敗時回滾
- BR4: 合法的狀態轉換路徑遵循 M3 狀態機定義：TODO->IN_PROGRESS、IN_PROGRESS->TODO、IN_PROGRESS->DONE、DONE->CLOSED
- BR5: 優先級 badge 顏色對應：HIGH=#EF4444（紅）、MEDIUM=#F59E0B（橙）、LOW=#10B981（綠）
- BR6: 頁面載入時一次性取得所有任務（size=1000），不使用分頁，以確保看板完整呈現
- BR7: Error toast 顯示時間為 3 秒後自動消失

## 相依模組

- 依賴: M2 (Task CRUD) — 取得任務列表資料、M3 (Task State Machine) — 狀態轉換 API 與規則
- 被依賴: 無
