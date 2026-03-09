# M6: Task List Page (Vue Frontend)

## 需求描述

本模組實現任務列表頁面，為 TaskFlow 系統的核心前端頁面之一。使用 Vue 3 框架搭配 Pinia 狀態管理與 axios HTTP 客戶端，提供分頁任務列表、關鍵字搜尋、優先級與狀態篩選等功能。

頁面載入時自動從後端 API 取得任務列表，每頁預設顯示 20 筆任務。使用者可透過搜尋列依關鍵字篩選、透過下拉選單依優先級或狀態篩選任務。每個任務項目顯示標題、狀態標籤、優先級標籤、負責人名稱與關聯標籤。

頁面提供「新增任務」按鈕開啟建立表單，以及分頁控制元件供使用者翻頁瀏覽。點擊任務項目可導航至任務詳情頁面。

## 驗收標準

- AC1: 進入 /tasks 路由時，頁面自動載入並顯示任務列表，每頁預設 20 筆，按建立時間降序排列
- AC2: 搜尋列輸入關鍵字後，列表自動篩選僅顯示標題包含該關鍵字的任務（呼叫 GET /api/tasks?keyword=xxx）
- AC3: 優先級下拉選單包含「全部」、HIGH、MEDIUM、LOW 四個選項，選擇後列表篩選僅顯示對應優先級的任務
- AC4: 狀態下拉選單包含「全部」、TODO、IN_PROGRESS、DONE、CLOSED 五個選項，選擇後列表篩選僅顯示對應狀態的任務
- AC5: 多個篩選條件可同時生效（例如 keyword + priority + status 組合篩選）
- AC6: 每個任務項目顯示以下資訊：標題（title）、狀態標籤（status badge）、優先級標籤（priority badge）、負責人名稱（assignee displayName，若無則顯示「未指派」）、關聯標籤（tags，顯示標籤名稱與顏色）
- AC7: 狀態標籤依狀態值顯示不同顏色：TODO 灰色、IN_PROGRESS 藍色、DONE 綠色、CLOSED 紅色
- AC8: 優先級標籤依優先級值顯示不同顏色：HIGH 紅色、MEDIUM 橙色、LOW 綠色
- AC9: 點擊任務項目，頁面導航至 /tasks/:id 任務詳情路由
- AC10: 點擊「新增任務」按鈕，開啟任務建立表單（TaskCreateForm 元件）
- AC11: 任務建立表單包含 title（必填）、description（選填）、priority（下拉選單，預設 MEDIUM）欄位，送出後呼叫 POST /api/tasks 建立任務
- AC12: 任務建立成功後，表單關閉並自動重新載入任務列表以顯示新任務
- AC13: 分頁控制元件顯示當前頁碼、總頁數，提供「上一頁」、「下一頁」按鈕
- AC14: 點擊「上一頁」或「下一頁」按鈕，載入對應頁碼的任務資料
- AC15: 第一頁時「上一頁」按鈕停用（disabled），最後一頁時「下一頁」按鈕停用
- AC16: API 請求進行中時，頁面顯示載入中狀態（loading indicator）
- AC17: API 請求失敗時，頁面顯示錯誤提示訊息

## 元件結構

### TaskListPage
- 路由: /tasks
- 說明: 任務列表頁面主元件，負責組合篩選區塊、任務列表與分頁控制元件
- 職責: 管理頁面層級狀態（loading、error），協調子元件互動

### TaskFilters
- 說明: 篩選區塊元件，包含搜尋列與篩選下拉選單
- Props: 無（從 Pinia store 讀取當前篩選狀態）
- Events: 篩選條件變更時觸發 store action 重新載入任務列表
- 內容:
  - 搜尋輸入框（keyword）
  - 優先級下拉選單（priority filter）
  - 狀態下拉選單（status filter）

### TaskListItem
- 說明: 單一任務項目元件，顯示任務摘要資訊
- Props:
  - `task` (Object, required) — 任務物件，包含 id、title、status、priority、assignee、tags
- Events:
  - `click` — 點擊時觸發，由父元件處理路由導航
- 顯示內容: 標題、狀態 badge、優先級 badge、負責人名稱、標籤列表

### TaskCreateForm
- 說明: 新增任務的彈出表單元件
- Props:
  - `visible` (Boolean, required) — 控制表單顯示/隱藏
- Events:
  - `created` — 任務建立成功後觸發
  - `cancel` — 使用者取消操作時觸發
- 表單欄位: title（必填，1-200 字元）、description（選填，最大 5000 字元）、priority（下拉選單，預設 MEDIUM）

### Pagination
- 說明: 分頁控制元件
- Props:
  - `currentPage` (Number, required) — 當前頁碼（從 0 開始）
  - `totalPages` (Number, required) — 總頁數
- Events:
  - `page-change` (Number) — 頁碼變更時觸發，攜帶目標頁碼

## 狀態管理 (Pinia Store)

### taskStore
- State:
  - `tasks` (Array) — 當前頁任務列表
  - `page` (Number) — 當前頁碼，預設 0
  - `size` (Number) — 每頁筆數，預設 20
  - `totalElements` (Number) — 總任務數
  - `totalPages` (Number) — 總頁數
  - `keyword` (String) — 搜尋關鍵字
  - `priorityFilter` (String|null) — 優先級篩選值
  - `statusFilter` (String|null) — 狀態篩選值
  - `loading` (Boolean) — 載入中狀態
  - `error` (String|null) — 錯誤訊息
- Actions:
  - `fetchTasks()` — 依據當前篩選條件與分頁參數呼叫 GET /api/tasks 取得任務列表
  - `setKeyword(keyword)` — 設定搜尋關鍵字並重新載入（重置頁碼為 0）
  - `setPriorityFilter(priority)` — 設定優先級篩選並重新載入（重置頁碼為 0）
  - `setStatusFilter(status)` — 設定狀態篩選並重新載入（重置頁碼為 0）
  - `setPage(page)` — 設定頁碼並重新載入
  - `createTask(taskData)` — 呼叫 POST /api/tasks 建立任務，成功後重新載入列表

## 業務規則

- BR1: 任務列表使用 axios 呼叫後端 API，所有請求自動附帶 JWT Token（透過 axios interceptor）
- BR2: 篩選條件變更時自動重置頁碼為第 0 頁，避免超出範圍
- BR3: 搜尋為即時篩選（可加入 debounce 300ms 延遲以減少 API 呼叫頻率）
- BR4: 任務列表狀態統一由 Pinia taskStore 管理，元件透過 store 讀取與更新資料
- BR5: 空列表時顯示「目前沒有任務」提示文字
- BR6: 狀態 badge 顏色對應：TODO=#6B7280（灰）、IN_PROGRESS=#3B82F6（藍）、DONE=#10B981（綠）、CLOSED=#EF4444（紅）
- BR7: 優先級 badge 顏色對應：HIGH=#EF4444（紅）、MEDIUM=#F59E0B（橙）、LOW=#10B981（綠）

## 相依模組

- 依賴: M2 (Task CRUD) — 任務列表與建立 API、M3 (Task State Machine) — 狀態篩選與顯示、M5 (Assignment) — 負責人資訊顯示與篩選
- 被依賴: 無
