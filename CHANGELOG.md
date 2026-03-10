# 變更紀錄

此檔案將記錄本專案的所有重大變更。

格式基於 [Keep a Changelog](https://keepachangelog.com/zh-CN/1.0.0/)，
並且本專案遵循 [語意化版本 (Semantic Versioning)](https://semver.org/lang/zh-TW/)。

> **版本重設說明**：本專案前身為 [TouchFish-Skills](https://github.com/agony1997/TouchFish-Skills) 的 `plugins/dev-team`。
> 原先版本號為隨手分配（v1.0→v1.1→v1.2→v1.3→v2.0→v2.1→v2.2→v3.0），不符合語意化版本原則。
> 現已根據 git 歷史重新分配版本號：0.x 為原型快速迭代期，1.0.0 為首個正式發布。
> 設計文件中出現的「v3.0」即為本版本號重設後的 v1.0.0。

## [1.5.0] - 2026-03-09

### 新增

- **Complexity estimation**：P0 為每個 task 估算複雜度（LOW/MED/HIGH），記錄 `complexity` + `reason` 於 PLAN `[TASK]` 行
- **Wave sub-batching**：異質 wave（混合 LOW/MED + HIGH）自動拆為子批次（Na=快, Nb=慢），同質 wave 不拆
- **Sub-batch execution rule**：P2 按子批次順序執行，批次內平行、批次間循序，避免慢 agent 阻塞快 agent 的後續 pipeline

### 驗證

- **E3 測試**（2026-03-09）：8 tasks, 272/272 tests PASS, ~50min, 0 compaction, 0 TL 介入, all first-pass QA PASS
- E2-I6（slow agent blocking）完全消除：critical path 提早 15 min，pipeline idle 從 20% 降至 13%
- 每任務效率 6.25 min/task（E2: 8.6, E1: 11.3）

## [1.4.0] - 2026-03-06

### 新增

- **Context budget rule**：TL 當 router 不當 relay，傳路徑不傳內容，防止 TL context 在 Phase 3 前爆滿
- **Minimal return**：test-agent / Worker / qa-task / qa-global 只返回摘要，詳細內容寫檔
- **tl-state.md 恢復機制**：TL 在每次 phase 轉換、agent spawn/shutdown 時更新 `temp/tl-state.md`，context compaction 後 re-read 恢復狀態
- **test-agent impl-notes**：test-agent 完成測試後寫 `temp/impl-notes-N.md`，提供框架陷阱和 mock 策略提示給 Worker
- **Worker PARALLEL AWARENESS**：Worker 只跑自己的測試檔案，不跑全套、不修改其他 Worker 的檔案
- **P0 Cross-cutting concerns**：Phase 0 新增步驟識別橫切關注點（error handling、auth、logging），記錄在 PLAN `[CROSS-CUTTING]`
- **P1 AC cross-check**：CONTRACT 撰寫前逐條比對 acceptance criteria，確保每個 AC 有對應 endpoint

### 變更

- **TL 禁止直接修復**：Phase 3 qa-global 發現的問題必須走 fix task → spawn fix Worker，TL 不得自己 Edit 程式碼
- **test-agent 語法驗證提升**：OUTPUT 段落新增必要步驟 — 返回前必須跑每個測試檔確認無 syntax/compile error
- **預設輸出目錄**：從 `docs/dev-team/<feature>/` 改為 `docs/dev-team/YYYY-MM-DD-HHmmss/`
- **qa-global 自讀檔案**：TL 不轉發內容，qa-global 從磁碟自行讀取所有檔案

## [1.3.0] - 2026-03-06

### 新增

- **Skill Isolation Directive**：`<EXTREMELY_IMPORTANT>` 區塊明確禁止 12 個 superpowers 技能，防止 superpowers:brainstorming 等攔截 dev-team 流程
- **Description 前綴**：加入「⚠️ SELF-CONTAINED SKILL」標記

### 修復

- **E0 skill 衝突問題**：E0 實驗證實 superpowers 技能會完全取代 dev-team 的 Phase 0-4 流程，v1.3.0 的反截斷指令解決此問題

## [1.2.0] - 2026-03-03

### 新增

- **Token 成本透明度**：Phase 0 確認 PLAN 時輸出預估 spawn 數量和模型分佈，讓使用者知道規模後再決定是否執行
- **Fix Task context 傳遞**：QA FAIL 時 TL 將失敗原因摘要寫入 fix task description，新 Worker 有完整 context
- **qa-global 可選整合測試**：CHECK 6 — 如果專案有現成測試框架，跑一次全部測試確認無回歸
- **部分交付支援**：DELIVERY.md 支援「部分完成」標記，列出成功和失敗的任務
- **test-agent RULES 補完**：需求模糊時停止回報 TL、同一 criterion 允許多斷言
- **改版驗證 Checklist**：`docs/checklist-release.md`
- **docs/ 文件索引**：`docs/INDEX.md`，統一檔名格式 `{date}-v{version}-{category}-{descriptor}.md`

### 變更

- **qa-task 改用 Opus**：品質閘門（6 步驟複雜推理）改用最強模型
- **METRICS 改由 TL 記錄**：Sub-agent 移除 [METRICS] 自報，TL 從 Agent tool 回傳取得 usage 後寫入 tl.log
- **Worker 併發上限標暫定**：`≤ 3 Workers concurrent` 標註為暫定經驗值，待實測調整
- 移除未經驗證的具體數字（~40% token savings → 待驗證、10 倍成本差 → 模糊描述）

### 移除

- 過時的 `GUIDE.zh-TW.md`（與 v1.1.0 嚴重矛盾）
- Sub-agent prompts 中的 [METRICS] 自報行（test-agent、qa-task、qa-global）

## [1.1.0] - 2026-03-03

### 破壞性變更

- **Phase 重新編號**：P0-P5 → P0-P4。舊 P0（Reconnaissance）+ P1（Requirements）合併為新 P0（Project Understanding + Task Planning）
- **移除所有外部 plugin 偵測邏輯**：不再偵測 explorer、superpowers-tdd、reviewer、OpenSpec。選擇「完全獨立」方案，消除 2^4 = 16 種組合的測試矩陣爆炸問題

### 新增

- **需求澄清步驟**：Phase 0 新增理解摘要輸出 + AskUserQuestion 確認理解，避免基於錯誤理解拆任務
- **TDD 紀律內建**：test-agent prompt 新增 TDD DISCIPLINE 段落（RED-GREEN 語義、失敗驗證、最小化測試）
- **Worker scope 自驗**：COMPLETION SEQUENCE 新增步驟 1.5（`git diff --name-only` + ALLOWED list 比對 + 自動 revert）
- **規範資料夾詢問**：從自動偵測 `.standards/` 改為 AskUserQuestion 主動詢問路徑
- **Execution strategy 註記**：預設 `parallel-per-task`，設計空間留給未來 `batch-test-first` 等替代方案
- **設計決策紀錄**：`docs/2026-03-03-v1.1-design-rationale.md`

### 移除

- **DELIVERY Agent Metrics section**：8 section → 7 section。Agent 層級 log 中 `[METRICS]` 行保留作 debug 用途
- **PLAN Integrations section**：移除 `[DETECT]` 行，不再追蹤外部 plugin 狀態
- **英文 README**：快速迭代期間僅維護中文版

### 變更

- plan-template.md 的 Phase 引用從 Phase 1 → Phase 0
- delivery-sub prompt 從 8-section → 7-section

## [1.0.0] - 2026-03-03

### 拆分

- **從 TouchFish-Skills 拆分為獨立 repo**：dev-team 是「編制框架」（完整開發系統），與其他 6 個「管線單元」在本質上不同，故獨立管理。

### 破壞性變更

- **完全重設計**：從 0.5.0（task pool + Challenger）地面重寫
    - 移除 Challenger 角色，品質保證改由 per-task 三方交叉驗證 QA + Phase 4 全域審查
    - Worker 從任務池自取改為 **1-task-per-worker**（TL 指派，完成即 shutdown）
    - 新增 **分離測試**：test-agent（Opus sub-agent）先寫測試 → Worker 寫 code → QA 審查
    - 文件架構改為 **LLM-native 格式**（PLAN + CONTRACT）+ Markdown（DELIVERY）+ 分散式 logs/
    - Phase 重組為 **6 個**（P0 偵察→P1 規劃→P2 Contract→P3 開發→P4 全域審查→P5 交付）
    - 移除 TRACE / PROCESS_LOG / ISSUES 文件，改用 TaskList + 分散式 append-only log

### 新增

- **新 prompts**：`test-agent.md`、`qa-task.md`、`qa-global.md`、`delivery-sub.md`
- **新 references**：`plan-template.md`、`log-templates.md`
- **DELIVERY 升級**：從簡要報告升級為完整開發回歸文件
- **獨立 repo 基礎**：README、LICENSE、CHANGELOG、使用指南

### 移除

- `prompts/challenger.md`
- `references/trace-template.md`、`process-log-template.md`、`issues-template.md`、`qa-review-template.md`

## [0.5.0] - 2026-02-26

### 新增

- Agent Metrics 功能：追蹤每個 agent 的 token 用量和耗時

### 修復

- batch 處理邏輯修正
- 誠實 metrics 報告（不偽造數據）
- 早期契約衝突警告
- Worker race condition 修復

## [0.4.0] - 2026-02-26

### 破壞性變更

- **架構重寫為 task pool 模式**：Worker 從任務池自取（取代 pipeline 指派）
- 移除 explore-leader、pg-leader、qa-leader 三個子角色
- 新增 **Challenger** 角色：對抗式 QA 審查
- 新增 `qa-review-template.md`

## [0.3.0] - 2026-02-26

### 新增

- 文件追蹤系統：TRACE、PROCESS_LOG、ISSUES 模板
- API 契約模板（`api-contract-template.md`）
- 交付報告模板（`delivery-report-template.md`）
- Lightweight mode：小型任務快速通道
- Stop rule：任務中止規則
- Scope check：範圍檢查機制
- Batch report：批次報告

## [0.2.0] - 2026-02-26

### 變更

- **統一架構重構**：英文 SKILL.md（始終載入）+ prompts/references/（按需載入）
- 新增 explore-leader、pg-leader、qa-leader、worker 四個 prompt 模板
- 新增繁體中文使用指南（`docs/GUIDE.zh-TW.md`）

## [0.1.0] - 2026-02-25

### 新增

- dev-team 插件初始版本
- Pipeline 架構：TL（Opus）編排 explore-leader、pg-leader、qa-leader + Workers（Sonnet）
- 混合 agent 模式（teammates + sub-agents）
- 基本的多角色協作流程
