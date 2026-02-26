# dev-team 技能使用指南

> 版本：2.0.0 | 最後更新：2026-02-26

## 這是什麼？

dev-team 是一個**任務池架構的多角色 Agent 團隊協作技能**，讓 Claude Code 模擬一個開發團隊來完成功能開發。
你只需要提供需求或規格書，技能會自動組建團隊、建立任務池、Workers 自取任務並行開發、審查、交付。

---

## 團隊架構

```
┌─────────────────────────────────────────────────────┐
│                    你（使用者）                        │
│              提供需求 / 確認方向 / 最終決定             │
└─────────────────────┬───────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────┐
│              Team Lead (Opus) — PM                   │
│     需求分析 · API 契約 · spawn 所有人 · 品質閘門      │
│                                                      │
│  ┌────────────┐  ┌────────┐ ┌────────┐ ┌────────┐  │
│  │ challenger │  │worker-1│ │worker-2│ │worker-N│  │
│  │  (Sonnet)  │  │(Sonnet)│ │(Sonnet)│ │(Sonnet)│  │
│  │  魔鬼代言人 │  │  自取   │ │  自取   │ │  自取  │  │
│  │  持續質疑   │  │  任務池 │ │  任務池 │ │  任務池 │  │
│  └────────────┘  └────────┘ └────────┘ └────────┘  │
│                                                      │
│  ＋ 一次性 QA sub-agents（TL 按需 spawn，審完即銷毀）  │
└─────────────────────────────────────────────────────┘
```

### 角色說明

| 角色 | Model | 職責 |
|------|-------|------|
| **Team Lead (TL)** | Opus | PM：需求分析、API 契約、spawn 所有 agents、品質閘門、QA sub-agent spawn |
| **challenger** | Sonnet | 魔鬼代言人：質疑設計決策、審查跨任務一致性、契約合規性。持續存在 |
| **worker-1..N** | Sonnet | 開發 Workers：從 TaskList 自取任務、寫程式碼、完成後回報 TL |
| QA sub-agent | Sonnet | 一次性審查：TL 收到完成通知後 spawn、審查、回傳結果、銷毀 |

### v1.3 → v2.0 關鍵變更

| 項目 | v1.3 | v2.0 |
|------|------|------|
| 架構 | TL + pg-leader + qa-leader + explore-leader | **扁平：TL + challenger + workers** |
| 任務分派 | pg-leader 指派 | **Workers 從 TaskList 自取（任務池）** |
| 團隊模式 | 輕量 / 完整兩種 | **單一扁平模式** |
| 品質審查 | qa-leader + sub-agents | **TL 直接 spawn 一次性 QA sub-agent** |
| 質疑角色 | 無 | **challenger（魔鬼代言人，持續存在）** |
| 複雜度評分 | 無 | **S(1pt) / M(2pt) / L(3pt)，每 worker 3-5 pts** |
| File Scope | 無 | **每個任務明確定義 ALLOWED / READONLY / FORBIDDEN** |

---

## 使用流程

```
Phase 0        Phase 1          Phase 2         Phase 3
專案偵察  ───→  需求分析    ───→  API 契約  ───→  組建團隊
(探索專案)     (拆分任務)       (定義介面)      (spawn agents)
                  │
          多規格書？──→ 詢問你要並行/序列/單一
                  │
                  ▼
Phase 4          Phase 5          Phase 6
任務池開發  ───→  契約驗證   ───→  交付
(自取+審查並行)  (最終檢查)      (報告+關閉)
```

### Phase 0: 專案偵察

TL 先了解你的專案。如果有安裝 `explorer` skill 會自動使用，否則手動掃描。
產出：PROJECT_MAP.md（專案地圖）。

### Phase 1: 需求分析

1. TL 閱讀你的需求/規格書
2. **範圍檢查**：TL 比對現有程式碼，若大部分已實作會先問你確認調整範圍
3. **多份規格書時**，TL 會分析領域間的依賴關係，問你要並行/序列/單一
4. 拆分任務，每個任務附帶：
   - **複雜度評分**：S(1pt) / M(2pt) / L(3pt)
   - **File Scope**：ALLOWED（可修改）/ READONLY（唯讀參考）/ FORBIDDEN（禁止觸碰）
   - 依賴關係（blockedBy / blocks）
5. 問你確認任務清單

### Phase 2: API 契約

TL 定義前後端共用的 API 契約，問你確認。確認後成為開發基準。
任何契約變更必須經 TL 核准並通知所有 Workers。

### Phase 3: 組建團隊

TL 根據任務總點數決定團隊規模：

```
計算方式：
  總工作量 = 所有任務點數加總（S=1, M=2, L=3）
  目標：每個 worker 分配 3-5 點
  前端 + 後端 → 至少各 1 worker
  相依任務 → 指派給同一 worker
  上限：5 workers
```

Spawn 順序：challenger（1 個）→ workers（N 個）。
TL 指派每個 Worker 第一個任務，之後 Workers 自取。

### Phase 4: 任務池開發

三條平行管線同時運行：

| 管線 | 執行者 | 觸發條件 |
|------|--------|----------|
| 開發 | Workers | 從 TaskList 自取 pending 任務 |
| 審查 | TL → QA sub-agent | Worker 回報任務完成 |
| 質疑 | challenger | TL 在 checkpoint 通知 |

**Worker 自取流程**：
1. 完成當前任務 → TaskUpdate completed → SendMessage TL
2. TaskList → 找 pending、無 blockedBy、無 owner 的任務
3. TaskUpdate 設自己為 owner → 開始執行
4. 沒有可取任務 → 通知 TL 等待指示

**QA 審查流程**：
- TL 收到完成通知 → spawn 一次性 QA sub-agent 審查
- PASS → 更新 TRACE 為 `qa-pass`
- FAIL → 建立修復任務放回任務池

**Challenger 審查時機**：Phase 2 後審查契約、每批任務完成後審查一致性、Phase 5 參與契約驗證。

### Phase 5: 契約驗證

TL spawn 專屬 QA sub-agent + 通知 challenger 參與最終契約驗證：
- 後端 API 符合契約？前端呼叫符合契約？
- Request/Response 對齊？錯誤處理？共用型別？

不一致 → 建立修復任務回到任務池。全部通過 → 進入 Phase 6。

### Phase 6: 交付

TL 產出交付報告，關閉團隊。**不會自動 commit/push**，由你決定。

---

## 產出文件

dev-team 會在你指定的輸出目錄（預設 `docs/dev-team/<feature>/`）產出以下追蹤文件。
所有檔案帶日期前綴（`YYYY-MM-DD-`），以專案開始日為準：

```
<output-dir>/
├── YYYY-MM-DD-TRACE.md              ← 追蹤矩陣（核心：需求 ↔ 任務 ↔ QA 雙向綁定）
├── YYYY-MM-DD-API_CONTRACT.md       ← API 契約（Phase 2 產出，含修改紀錄）
├── YYYY-MM-DD-PROCESS_LOG.md        ← 流程紀錄（關鍵事件追加式記錄）
├── YYYY-MM-DD-ISSUES.md             ← 問題與決策紀錄
└── YYYY-MM-DD-DELIVERY_REPORT.md    ← 交付報告（Phase 6 產出，含反向連結）
```

**TRACE.md** — 雙向綁定核心：`上游規格節 → Req-ID → 任務 → Worker → 開發狀態 → QA 結果`。
狀態流程：`pending → in-progress → done → qa-pass / qa-fail → fixed`。TL 自動更新。

**PROCESS_LOG.md** — 關鍵事件記錄（非詳細對話）。事件類型：
`team-assembled`, `task-completed`, `qa-triggered`, `review-pass`, `review-fail`, `issue-reported`, `decision`, `contract-amended`, `phase-transition`

**ISSUES.md** — QA 審查失敗或契約驗證不一致時自動建立條目。

**DELIVERY_REPORT.md** — Phase 6 最終產出：已實作項目、QA 結果摘要、契約驗證結果、延遲項目、交叉引用所有追蹤文件。

---

## 溝通路徑

```
         ┌──────┐
         │  TL  │
         └──┬───┘
      ╱   ╱ │ ╲   ╲
  chall  w1 w2 w3  QA ← 一次性 sub-agent

  允許：TL ↔ challenger / TL ↔ workers / TL → QA sub-agent
  禁止：worker ↔ worker / worker ↔ challenger（皆須經 TL）
```

---

## 使用範例

### 單一規格書

```
你：「用 dev-team 實作收貨功能，規格書在 docs/receive-spec.md」
→ TL 走完 Phase 0-6
→ 產出完整實作 + 交付報告
```

### 多份規格書

```
你：「用 dev-team 實作收貨和配貨功能，規格書在 docs/receive-spec.md 和 docs/allocation-spec.md」
→ TL 分析依賴後問你要並行還是序列
→ 你選擇後 TL 組建相應團隊
```

---

## 檔案結構

```
plugins/dev-team/
├── .claude-plugin/plugin.json          ← 插件元資料 (v2.0.0)
├── skills/dev-team/
│   ├── SKILL.md                        ← AI 核心指令（英文，始終載入）
│   ├── prompts/                        ← Spawn 模板（按需載入，每次只讀一個）
│   │   ├── challenger.md               ← 魔鬼代言人（持續 teammate）
│   │   └── worker.md                   ← 開發 Worker（自取任務池）
│   └── references/                     ← 文件模板（按需載入）
│       ├── trace-template.md           ← 追蹤矩陣模板
│       ├── api-contract-template.md    ← API 契約模板
│       ├── process-log-template.md     ← 流程紀錄模板
│       ├── issues-template.md          ← 問題紀錄模板
│       ├── delivery-report-template.md ← 交付報告模板
│       └── qa-review-template.md       ← QA 審查模板（一次性 sub-agent 用）
└── docs/
    └── GUIDE.zh-TW.md                  ← 本文件（中文，給人讀的）
```

---

## 常見問題

**Q: 還有 pg-leader / qa-leader / explore-leader 嗎？**
A: v2.0 移除了所有中間管理層。TL 直接管理 challenger 和所有 workers，QA 由 TL spawn 一次性 sub-agent 執行。

**Q: Workers 怎麼知道做哪個任務？**
A: Workers 從 TaskList（任務池）自取 pending 且無阻塞的任務。TL 只指派第一個任務，之後 Workers 自行認領。

**Q: challenger 是什麼角色？**
A: 魔鬼代言人。專門找問題：質疑設計決策、檢查跨任務一致性、發現 edge cases。它是持續存在的 teammate，不是一次性的。

**Q: File Scope 是什麼？**
A: 每個任務明確定義 ALLOWED（可修改）、READONLY（唯讀參考）、FORBIDDEN（禁止觸碰）的檔案清單。防止 Workers 互相踩踏。

**Q: 我可以一次給多份規格書嗎？**
A: 可以。TL 會分析依賴後問你要並行、序列或逐個完成。

**Q: 需要搭配 DDD 流程嗎？**
A: 不必。dev-team 可以獨立使用。但如果先用 ddd-core 做完設計再交給 dev-team 實作，效果最好。

**Q: Worker 數量可以控制嗎？**
A: TL 根據任務點數自動計算（每 worker 3-5 pts，上限 5 workers）。如果你想調整，可以在 Phase 1 確認時告訴 TL。
