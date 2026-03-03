# dev-team v2.2.0 質疑報告

> 審查日期：2026-03-02
> 審查方式：以質疑者（challenger）身分對 SKILL.md、prompts、references 全文件進行批判性審查
> 審查範圍：架構設計、實務執行、缺失防護、過度工程、矛盾與模糊、平台限制

---

## 嚴重程度定義

| 等級 | 定義 |
|------|------|
| CRITICAL | 會導致整個流程崩潰或產出品質嚴重低下 |
| HIGH | 在常見情境下會造成明顯問題 |
| MEDIUM | 設計缺陷，長期累積會成為問題 |
| LOW | 可優化項目 |

---

## CRITICAL

### C1: TL 是單點瓶頸 + 單點故障

TL 同時承擔：Phase 0-2 的分析工作、所有 worker 的通訊中樞、QA spawn/結果處理、TRACE 維護、challenger 協調、PROCESS_LOG/ISSUES 更新。

**問題**：到 Phase 4 時，TL 的 context window 已經裝滿了規格分析、API 契約、TRACE、process log 等內容。每次 worker 回報完成，TL 要：Read TRACE → Edit TRACE → Read QA template → spawn QA → 收 QA 結果 → Edit TRACE → 可能 Edit ISSUES → 通知 worker。這是一連串 context 消耗。

**真實風險**：如果有 10+ 任務 + 5 workers 並行回報，TL 的 context 會在 Phase 4 中段爆滿，導致遺忘前期的契約細節或 scope 規則。整個團隊崩潰。

**缺少的**：沒有任何 context 管理策略，沒有 checkpoint-and-resume 機制。

### C2: 沒有測試執行要求

整份 SKILL.md 沒有一行要求 worker 執行測試。QA sub-agent 也只是讀程式碼 — 不能跑測試、不能編譯、不能驗證 runtime 行為。

對比 superpowers 強制 TDD，dev-team 的品質保證本質上是「讀 code + 對照 checklist」— 一個 glorified linting pass。一個 worker 可以寫出語法正確但邏輯完全錯誤的程式碼，QA 可能看不出來。

### C3: 多 Worker 同時修改 working tree，沒有 git 隔離

所有 workers 都在同一個 working tree 中操作。File Scope 的 ALLOWED/FORBIDDEN 是靠 prompt 「請求」worker 遵守，沒有實際強制機制。

**災難場景**：
- Worker-1 修改 `shared/types.ts`（在其 ALLOWED 中）
- Worker-2 也需要修改同一檔案（TL 疏忽分配）
- 兩者同時 Edit → 後者覆蓋前者的修改
- 沒有 git branch 隔離，沒有 merge conflict 偵測

File Scope 只是文字聲明，不是技術強制。

---

## HIGH

### H1: QA sub-agent 缺乏足夠 context

QA sub-agent 是一次性的、收到的 context 只有：file_list + contract + standards + task description + "relevant existing code context"。

**問題**：
- 不了解整個專案架構
- 不知道其他 task 的實作方式（跨 task 一致性無法驗證）
- 看不到 runtime 行為、DB schema、環境設定
- 被要求做 "API Contract Compliance" 檢查但可能連 contract 的完整上下文都不夠

**結果**：QA 結果的 false negative 率（漏放問題）很高。

### H2: Challenger 的投資報酬率可疑

Challenger 是持續存在的 teammate（Sonnet），在 checkpoints 才被激活。

**問題**：
- 大部分時間 idle，但持續佔用 API 資源
- 不能跑 code、不能修改、只能 Read + 分析
- 對 <5 任務的小專案，challenger 的開銷 > 價值
- checkpoint review 的時機由 TL 主觀決定，可能太晚才觸發

**缺少退場條件**：如果 challenger 連續 N 次 review 都 "No issues found"，是否應該提前 shutdown？

### H3: Metrics 數據多半不可用

| 數據 | 真實性 |
|------|--------|
| QA sub-agent tokens | 精確 (from Task return) |
| Worker tokens | `n/a` — 無法取得 |
| Challenger tokens | `n/a` — 無法取得 |
| TL tokens | `n/a` — 無法取得 |
| Duration (wall-clock) | spawn_time → shutdown_time，包含 idle 時間，不代表工作時間 |
| Cost estimate | 只有 QA 部分精確，其他全靠估算 |

Delivery Report 的 "Agent Metrics" 給使用者一個精確報告的假象，但實際上 ~70% 的資源消耗是 `n/a`。這不是 "metrics"，是 "partial metrics with gaps"。

### H4: Worker 自取任務的 race condition

```
Worker-1: TaskList → 找到 T-05 → TaskUpdate owner=worker-1
Worker-2: TaskList → 找到 T-05 → TaskUpdate owner=worker-2
```

Step 5 的 "verify claim" 是事後檢查：`TaskGet → confirm owner is yourself`。但在分散式環境下，兩個 worker 的 TaskUpdate 可能都成功（後者覆蓋前者），然後 worker-1 的 verify 失敗。

**更糟的情況**：worker-1 已經開始工作了才 verify 失敗 — 浪費了工作。Verify 應該在 TaskUpdate 之前，但 TaskList API 不支援 atomic claim。

---

## MEDIUM

### M1: 5 個追蹤文件的維護負擔不成比例

TRACE.md + API_CONTRACT.md + PROCESS_LOG.md + ISSUES.md + DELIVERY_REPORT.md

對 3-5 個任務的小功能，TL 花在維護這 5 個文件的 context 可能比實際協調工作還多。TaskList 本身已經追蹤了大部分狀態（status, owner, blockedBy）。TRACE 與 TaskList 有大量重複。

**沒有輕量模式**：SKILL.md 沒有根據任務規模自動簡化流程的機制。3 個任務和 30 個任務走同樣的完整流程。

### M2: Phase 2 skip 後 Phase 5 的處理不明確

Phase 2 skip condition 說：「if requirements contain no API interactions, skip Phase 2 entirely. Note 'Phase 2: N/A'」

但 Phase 5 是 "Contract Consistency Check" — 如果 Phase 2 被 skip 了：
- Phase 5 要 spawn 什麼？contract verification without contract?
- SKILL.md 沒有 Phase 5 的 skip condition 對應 Phase 2 N/A 的情況
- TL 需要自己判斷跳過，但沒有明確指示

### M3: Communication Discipline 的 STOP RULE 邊界模糊

```
STOP RULE: Do NOT reply to pure acknowledgments ("received", "noted", "got it").
```

**問題**：TL 的回覆常常是 acknowledge + 隱含指示（例如「收到，繼續下一個任務」）。Worker 可能把「繼續下一個任務」當成 acknowledgment 而不回覆。邊界模糊。

### M4: 沒有成本預估 / 預算上限

7 個 agents × 多小時 session = 潛在數百美元的 API 成本。

- Phase 1 沒有成本預估給使用者
- 沒有 budget ceiling 機制
- 使用者到 Phase 6 看到帳單才知道花了多少（而且如 H3 所述，帳單還不完整）

### M5: Context compaction 後的 critical context 遺失

長時間運行的 teammate 會被 Claude Code 自動 context compaction。Worker 可能遺失：
- File Scope 規則
- API 契約細節
- Project standards
- Communication discipline 規則

Worker prompt 說 "Read the API contract at the START of your work session"，但 compaction 後不會自動重讀。

---

## LOW

### L1: Worker prompt 的 `{peer_list}` 洩漏了其他 worker 資訊

```
Peers: {peer_list} (coordinate through TL if cross-task issues arise).
```

但 Communication Rules 禁止 worker-to-worker 通訊。知道 peers 的名字只會誘惑 worker 嘗試直接通訊。

### L2: `references/` 文件的 on-demand loading 增加 TL context 負擔

每個 Phase 開始時 TL 要 Read template → 填入 → Write。6 個 template 文件 = 6 次額外的 Read 操作。如果 TL 記得 template 結構，直接寫會更省 context。

### L3: Pricing 硬編碼

```
Pricing (as of 2026-02): Opus in=$15/MTok out=$75/MTok | Sonnet in=$3/MTok out=$15/MTok.
```

SKILL.md 說 "Verify current pricing" 但這依賴 TL 主動去查。價格過期時 metrics 就不準確。

---

## 優先修復建議

### Top 3（最需要優先處理）

1. **加入測試執行要求** (C2)
   - Worker 完成任務時必須跑測試並報告結果
   - QA checklist 加入「測試通過」為必要條件
   - 可選：整合 superpowers 的 TDD 流程

2. **TL context 管理策略** (C1)
   - 定義 context checkpoint/summarize 機制
   - 考慮將 TRACE 維護委託給專用 sub-agent
   - 或在 Phase 3 就做一次 context snapshot

3. **git 隔離或真正的 file scope 強制** (C3)
   - 方案 A：每個 worker 使用獨立 worktree/branch
   - 方案 B：worker prompt 加入 pre-edit 自檢腳本
   - 方案 C：QA 增加 `git diff --name-only` 檢查是否越權

### Quick wins（低成本高效益）

- M2：Phase 5 加 skip condition 與 Phase 2 對應
- L1：移除 `{peer_list}` 或改為匿名資訊
- H2：加入 challenger 提前退場條件（例如連續 2 次 "No issues found" 後 shutdown）
