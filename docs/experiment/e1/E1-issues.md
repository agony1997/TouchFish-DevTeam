# E1 Baseline Test Issues

Test: 2026-03-06, dev-team v1.3.0, TaskFlow M1+M2+M3+M6
Duration: ~1h19m, 24 agents, 442 tests passed

## Issues

### I1: Context compaction 中斷 qa-global ✅ RESOLVED (v1.4.0)

- **階段**: P3 (Global Review)
- **現象**: TL session 在 P3 觸及 200k token 上限 (161k/200k, 81%)，自動壓縮後遺失正在背景執行的 qa-global agent ID，必須重新啟動 qa-global
- **影響**: 產生 2 個 qa-global（壓縮前/後各一），浪費 token + 時間

**根因鏈分析**:
1. 瞬態 context 累積：agent 返回全量內容，TL 變成胖中繼站
2. TL 直接修復程式碼：SKILL 設計應 create fix task，但 TL 自己 Edit（膨脹 context）
3. Worker-7 偵錯循環：卡在 vue-router click test，8 次 Edit + 7 版 debug script，24 min
4. Compaction 後無恢復機制：agent ID 丟失無法 resume

**Worker-7 深入分析**:
- 4 個 JSONL segment (3.5MB) 是 context compaction 快照，非重複執行
- 工作只做了 1 次，38 Bash + 21 Read + 8 Write + 10 Edit
- 根因：test-agent (Opus) 寫了 vue-router click navigation 測試，Worker (Sonnet) 缺乏實作提示
- Worker 在密集工作時不回應使用者互動（teammate 模式下的行為）

**v1.4.0 修復**:
1. Context budget rule：TL 當 router not relay，傳路徑不傳內容
2. Minimal return：test-agent/worker/qa-task 只返回摘要，詳細寫檔
3. TL 禁止直接修復：P3 issues 必須 create fix task → spawn fix Worker
4. test-agent 寫 impl-notes：框架陷阱、mock 策略提示（不是完整計畫）
5. tl-state.md 恢復機制：persist phase + active agent IDs，compaction 後 re-read

### I2: 跨 Worker 編譯衝突 ✅ RESOLVED (v1.4.0)

- **階段**: P2 (Development)
- **現象**: 並行 Worker 各自寫入同一 repo，互相踩到對方未完成的程式碼導致編譯失敗
- **影響**: Worker 需要暫時移除無關測試檔案才能跑自己的測試
- **根因**: 所有 Worker 共用同一 working tree，沒有隔離
- **決策**: 不做 git worktree 隔離（複雜度高），改用 file scope + worker 自律
- **v1.4.0 修復**: worker.md 加入 PARALLEL AWARENESS 段落 — Worker 只跑自己的測試檔案，不跑全套、不修改其他 Worker 的檔案

### I3: CONTRACT v1 遺漏 status filter ✅ RESOLVED (v1.4.0)

- **階段**: P1→P2→P3
- **現象**: CONTRACT v1 遺漏 GET /api/tasks 的 status query param，P2 期間 qa-task-6 發現後升級 v2，但後端 Worker 已完成實作（基於 v1），導致 P3 qa-global 再次抓到前後端不一致
- **影響**: P3 需要 TL 手動修復 4 個後端檔案
- **根因**: P1 CONTRACT 審查不夠細緻，沒有逐條比對 spec AC
- **決策**: 輕防線 — P1 加交叉比對提醒，接受遺漏可能發生，靠現有 amendment flow + no-self-fix 修正
- **v1.4.0 修復**: P1 step 1 加「Cross-check: walk each AC → verify endpoint coverage」；P3 no-self-fix 確保遺漏修復走 fix task 而非 TL 手動修

### I4: 錯誤處理格式不一致 ✅ RESOLVED (v1.4.0)

- **階段**: P2
- **現象**: Auth 模組用 ResponseStatusException（Spring 預設 JSON），Task 模組用自訂例外 + GlobalExceptionHandler（自訂格式），回傳格式不同
- **影響**: 前端需要處理兩種錯誤格式；qa-global 標記為 medium severity 但未修復
- **根因**: 不同 Worker 各自選擇不同的錯誤處理策略，CONTRACT 只規範了格式但沒強制實作方式
- **決策**: P0 識別橫切關注點，寫入 PLAN，Worker 讀 PLAN 時自然遵循統一策略
- **v1.4.0 修復**: P0 加 step 9「Cross-cutting concerns」— 識別 error handling、auth、logging 等共用策略，記錄在 PLAN `[CROSS-CUTTING]`

### I5: dev-team 透過 Read 載入而非 Skill 工具 ℹ️ NOTED

- **階段**: Session 開始
- **現象**: 模型用 Read 直接讀取 SKILL.md 內容，而非透過 Skill() 工具調用
- **影響**: 功能上無差異（內容完整讀入），但繞過了 Skill 工具的標準載入流程
- **根因**: skill 原始碼與測試目標在同一 repo，模型可直接 Read 檔案
- **決策**: 測試環境副作用，正式使用時 skill 安裝在使用者環境中透過 Skill 工具載入，不需修改

### I6: SecurityConfig 回傳 403 而非 401 ✅ POSITIVE

- **階段**: P2 (qa-task-1)
- **現象**: Spring Security 預設對未認證請求回傳 403，spec 要求 401
- **影響**: T1 QA FAIL，TL 需手動加入 AuthenticationEntryPoint
- **根因**: Spring Security 預設行為，test-agent 寫的測試正確抓到了
- **評價**: separated testing 機制運作良好——test-agent 先寫測試，Worker 實作後 QA 抓到不符。正面驗證

### I7: test-agent 產出的測試有編譯錯誤 ✅ RESOLVED (v1.4.0)

- **階段**: P2
- **現象**: test-agent-1 使用不存在的 status().isNot() API；test-agent-6 的 mock 命名匯出無法被 default import 存取
- **影響**: TL 需要手動修復測試程式碼才能繼續
- **根因**: test-agent (Opus) 對框架 API 細節不夠精確；prompt 中的語法驗證規則存在但未被遵守
- **決策**: 將語法驗證從 TDD DISCIPLINE 規則提升為 OUTPUT 必要步驟，強制返回前執行
- **v1.4.0 修復**: test-agent.md OUTPUT 段落加「BEFORE returning: run every test file once → fix syntax/compile errors」
