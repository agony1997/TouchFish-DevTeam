# dev-team 技能使用指南

> 版本：1.0.0 | 最後更新：2026-03-03

## 這是什麼？

dev-team 是一個**多角色 Agent 團隊協作技能**，讓 Claude Code 模擬一個開發團隊來完成功能開發。
你只需要提供需求或規格書，技能會自動組建團隊、分離測試、並行開發、三方交叉驗證、交付。

### v1.0.0 重大變更（從 v0.5.0）

- **移除 challenger 角色** — 品質改由分離測試 + 三方 QA + 全域審查確保
- **Worker 1 任務 1 生命** — 每個 Worker 只做一個任務就結束，TL 指派（非自取）
- **分離測試** — test-agent（Opus）先寫測試 → Worker 寫程式碼通過測試
- **三方交叉驗證** — QA 不信任任何單一 artifact：需求 ↔ 測試 ↔ 程式碼三方比對
- **LLM-native 工作文件** — PLAN 和 CONTRACT 使用 LLM 最佳格式（非 Markdown）
- **分散式日誌** — 每個 agent 維護自己的 log 檔案，取代集中式 TRACE/PROCESS_LOG
- **DELIVERY 升級** — 8 區塊開發回歸文件，幫助開發者了解 AI coding 做了什麼

---

## 團隊架構

```
┌──────────────────────────────────────────────────────────┐
│                      你（使用者）                           │
│               提供需求 / 確認方向 / 最終決定                 │
└──────────────────────┬───────────────────────────────────┘
                       │
                       ▼
┌──────────────────────────────────────────────────────────┐
│               Team Lead (Opus) — PM                       │
│      需求分析 · API 契約 · spawn 所有人 · 品質閘門          │
│                                                           │
│  ┌───────────┐  ┌──────────┐  ┌──────────┐              │
│  │test-agent │  │ worker-1 │  │ worker-N │              │
│  │  (Opus)   │  │ (Sonnet) │  │ (Sonnet) │   ≤ 3 個    │
│  │  先寫測試  │  │ 1任務1命  │  │ 1任務1命  │   同時運行   │
│  └───────────┘  └──────────┘  └──────────┘              │
│                                                           │
│  ＋ QA sub-agents（每任務三方驗證）                          │
│  ＋ QA 全域審查（Opus，Phase 4）                            │
│  ＋ 交付報告 sub-agent（Sonnet，Phase 5）                   │
└──────────────────────────────────────────────────────────┘
```

### 角色說明

| 角色 | Model | 職責 |
|------|-------|------|
| **Team Lead (TL)** | Opus | PM：需求分析、任務規劃、API 契約、spawn 所有 agents、品質閘門 |
| **test-agent** | Opus | 分離測試：先寫測試（紅綠測試、邊界、錯誤案例），Worker 再寫程式通過 |
| **worker-N** | Sonnet | 開發 Worker：TL 指派 1 個任務、寫程式碼通過測試、完成即結束 |
| **qa-task-N** | Sonnet | 三方交叉驗證：需求↔測試、測試↔程式碼、需求↔程式碼 |
| **qa-global** | Opus | 全域審查：跨任務一致性、契約合規性、完整性檢查 |
| **delivery-sub** | Sonnet | 交付報告：彙整所有日誌，產出 8 區塊 DELIVERY.md |

---

## 使用流程

```
Phase 0        Phase 1              Phase 2           Phase 3
專案偵察  ───→  需求分析+任務規劃 ───→  API 契約    ───→  開發執行
(探索專案)     (PLAN.md)            (CONTRACT.md)     (test→code→QA 循環)

                                                         │
Phase 5              Phase 4                             │
交付           ←───  全域審查        ←───────────────────┘
(DELIVERY.md)       (跨任務一致性)
```

### Phase 0: 專案偵察

TL 先了解你的專案。自動偵測 explorer skill、PROJECT_MAP、superpowers TDD、reviewer、.standards/ 等。
偵測結果記錄在 PLAN 中，有對應插件時自動升級體驗。

### Phase 1: 需求分析 + 任務規劃

1. TL 閱讀需求/規格書
2. **範圍檢查**：比對現有程式碼，大部分已實作會先問你
3. **多份規格書**：分析依賴後問你要並行/序列/單一
4. 拆分任務，每個任務附帶：
   - **File Scope**：ALLOWED（可修改，最多 5 個檔案）/ READONLY（唯讀參考）
   - 依賴關係（blockedBy / blocks）
5. 產出 PLAN.md（LLM-native）+ 人類可讀摘要
6. 問你確認任務清單

### Phase 2: API 契約

不涉及 API 時跳過。否則 TL 定義 CONTRACT.md（LLM-native）+ 人類可讀摘要，問你確認。
任何契約變更：TL 修改 → 終止受影響 Worker → 重新啟動。

### Phase 3: 開發執行

每個任務的三階段流程（最多 3 個 Worker 同時進行）：

```
1. test-agent (Opus) 寫測試
   → 紅綠測試 + 邊界測試 + 錯誤案例
   → 產出測試檔案

2. Worker (Sonnet) 寫程式碼
   → 讀 PLAN + CONTRACT + 測試
   → 寫程式碼通過所有測試
   → 完成後結束

3. QA (Sonnet) 三方交叉驗證
   → 需求↔測試、測試↔程式碼、需求↔程式碼
   → PASS 或 FAIL
```

QA 失敗 → 建立修復任務重新執行（最多 2 輪 → 問你處理）。

### Phase 4: 全域審查

任務 ≤ 2 個時跳過。否則 Opus sub-agent 審查跨任務一致性、契約合規性、完整性。

### Phase 5: 交付

Sonnet sub-agent 彙整所有日誌，產出 DELIVERY.md（8 區塊）。**不會自動 commit/push**。

---

## 產出文件

```
<output-dir>/
├── YYYY-MM-DD-PLAN.md          ← 專案計畫（LLM-native，靜態）
├── YYYY-MM-DD-CONTRACT.md      ← API 契約（LLM-native，可修訂）
├── YYYY-MM-DD-DELIVERY.md      ← 交付報告（Markdown，8 區塊，唯一人類文件）
├── logs/
│   ├── tl.log.md               ← TL 決策與事件紀錄
│   ├── worker-1.log.md         ← Worker 執行紀錄
│   ├── qa-task-1.log.md        ← QA 審查紀錄
│   ├── qa-global.log.md        ← 全域審查紀錄
│   └── ...
└── temp/                       ← 確認用人類摘要（流程結束後可刪除）
    ├── plan-summary.md
    └── contract-summary.md
```

### DELIVERY.md 八區塊

1. **Executive Summary** — 做了什麼、為什麼
2. **Requirements → Implementation Mapping** — 需求到程式碼的完整鏈路
3. **API Contract (Final)** — 人類可讀的最終契約
4. **Plan Drift Log** — 計畫偏移紀錄（原始 vs 實際）
5. **QA Cycle Report** — 每任務審查結果與修復歷程
6. **File Change Summary** — 新增/修改的檔案清單
7. **Agent Metrics** — 團隊組成、token 用量、耗時
8. **Known Issues & Future Work** — 已知問題與後續工作

---

## 檔案結構

```
TouchFish-DevTeam/
├── .claude-plugin/plugin.json
├── skills/dev-team/
│   ├── SKILL.md                        ← AI 核心指令（英文，始終載入）
│   ├── prompts/                        ← Spawn 模板（按需載入）
│   │   ├── worker.md                   ← Worker（1任務 teammate）
│   │   ├── test-agent.md              ← 分離測試（Opus sub-agent）
│   │   ├── qa-task.md                 ← 三方交叉驗證（Sonnet sub-agent）
│   │   ├── qa-global.md              ← 全域審查（Opus sub-agent）
│   │   └── delivery-sub.md           ← 交付報告（Sonnet sub-agent）
│   └── references/                     ← 文件模板（按需載入）
│       ├── plan-template.md           ← PLAN 格式（LLM-native）
│       ├── contract-template.md       ← CONTRACT 格式（LLM-native）
│       ├── delivery-template.md       ← DELIVERY 格式（Markdown 8區塊）
│       └── log-templates.md           ← 日誌格式（LLM-native）
└── docs/
    └── GUIDE.zh-TW.md                 ← 本文件
```

---

## 常見問題

**Q: challenger 去哪了？**
A: v1.0.0 移除了 challenger 角色。品質改由三層機制確保：(1) test-agent 分離測試消除同源偏誤、(2) QA 三方交叉驗證不信任任何單一 artifact、(3) 全域審查檢查跨任務一致性。

**Q: Worker 為什麼只做一個任務？**
A: LLM 的 context 會隨時間退化。1 任務 1 生命確保每個 Worker 都有最乾淨的 context，指令遵從度最高。

**Q: 什麼是「三方交叉驗證」？**
A: QA 不信任任何單一 artifact，而是三方比對：需求↔測試（測試有涵蓋需求嗎？）、測試↔程式碼（程式碼真的通過測試嗎？）、需求↔程式碼（程式碼做的是需求要的嗎？）。比 TDD 更嚴格。

**Q: 什麼是「LLM-native 格式」？**
A: PLAN 和 CONTRACT 使用 `[TYPE] key=value` 格式，比 Markdown 表格省 ~40% tokens，LLM 讀寫更穩定。人類可讀版本在 temp/ 摘要和 DELIVERY.md 中。

**Q: File Scope 是什麼？**
A: 每個任務限定最多 5 個可修改檔案（ALLOWED），防止 Worker 互相踩踏。需要修改範圍外的檔案必須向 TL 請求。

**Q: 需要搭配其他插件嗎？**
A: 不必。dev-team 可獨立使用。但自動偵測到其他插件時會升級體驗（如 explorer 偵察、superpowers TDD 嚴格模式）。

**Q: Worker 數量可以控制嗎？**
A: TL 根據任務數量自動決定（上限 3 個同時進行）。如果想調整，在 Phase 1 確認時告訴 TL。

**Q: Agent Metrics 精確嗎？**
A: Sub-agent（test/QA/delivery）有精確 token 數據。Worker（teammate）的 token 無法取得，標示 n/a。不計算費用。
