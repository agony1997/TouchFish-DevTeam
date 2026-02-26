# dev-team 技能使用指南

> 版本：1.3.0 | 最後更新：2026-02-26

## 這是什麼？

dev-team 是一個**多角色 Agent 團隊協作技能**，讓 Claude Code 模擬一個開發團隊來完成功能開發。
你只需要提供需求或規格書，技能會自動組建團隊、分派任務、開發、審查、交付。

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
│     需求分析 · API 契約 · spawn 所有人 · 全局協調      │
│                                                      │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐           │
│  │ explore  │  │    pg    │  │    qa    │           │
│  │ -leader  │  │ -leader  │  │ -leader  │           │
│  │ (Opus)   │  │ (Opus)   │  │ (Opus)   │           │
│  │          │  │ 管理者    │  │          │           │
│  │ sub-     │  │ 不寫程式  │  │ sub-     │           │
│  │ agents   │  │          │  │ agents   │           │
│  │ (Sonnet) │  │ ┌──┬──┐ │  │ (Sonnet) │           │
│  │          │  │ │p1│p2│ │  │          │           │
│  │          │  │ │p3│p4│ │  │          │           │
│  │          │  │ └──┴──┘ │  │          │           │
│  │          │  │ Workers  │  │          │           │
│  │          │  │ (Sonnet) │  │          │           │
│  └──────────┘  └──────────┘  └──────────┘           │
└─────────────────────────────────────────────────────┘
```

### 角色說明

| 角色 | Model | 職責 |
|------|-------|------|
| **Team Lead (TL)** | Opus | PM：需求分析、API 契約、spawn 所有 agents、全局協調、QA 觸發 |
| **pg-leader** | Opus | 管理者：任務拆分、指派給 workers、監控進度、協調依賴。**不自己寫程式** |
| **qa-leader** | Opus | 品質保證：收到 TL 通知後派 sub-agent 審查，回報結果 |
| **explore-leader** | Opus | 探索：深入了解專案特定區域（可選） |
| **pg-1, pg-2, ...** | Sonnet | 開發 workers：寫程式碼，由 TL spawn、pg-leader 管理 |

### v1.2 → v1.3 關鍵變更

| 項目 | v1.2 | v1.3 |
|------|------|------|
| 團隊模式 | 一律有 pg-leader | **輕量模式（≤2 workers 跳過 pg-leader）** |
| 訊息迴圈 | 常發生 ping-pong | **STOP RULE：純確認訊息不回覆** |
| 範圍檢查 | 無 | **Phase 1 比對現有程式碼，已實作部分先確認** |
| Worker 回報 | 逐個回報 | **允許批量回報多個小任務** |

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
流水線開發  ───→  契約驗證   ───→  交付
(開發+審查並行)  (最終檢查)      (報告+關閉)
```

### Phase 0: 專案偵察

TL 先了解你的專案。如果有安裝 `explorer` skill 會自動使用，否則手動掃描。
產出：PROJECT_MAP.md（專案地圖）。

### Phase 1: 需求分析

1. TL 閱讀你的需求/規格書
2. **範圍檢查**：TL 比對現有程式碼，若大部分已實作會先問你確認調整範圍
3. **多份規格書時**，TL 會分析領域間的依賴關係，問你要並行/序列/單一
4. 拆分任務，問你確認

### Phase 2: API 契約

TL 定義前後端共用的 API 契約，問你確認。確認後成為開發基準。

### Phase 3: 組建團隊

TL 根據任務數量決定團隊規模和模式：

| 任務數量 | Workers | 模式 |
|---------|---------|------|
| ≤3 | 1 worker | 輕量（TL 直接管理） |
| 4-8 | 2-3 workers | ≤2 輕量 / ≥3 完整 |
| ≥9 | 3+ workers | 完整（pg-leader 管理） |
| 前端+後端 | 至少各 1 | — |

- **輕量模式**（≤2 workers）：跳過 pg-leader，TL 直接管理 workers + qa-leader，減少 overhead
- **完整模式**（≥3 workers）：pg-leader 負責任務分解、指派、協調

### Phase 4: 流水線開發

通知鏈依模式不同：
- **輕量**：`worker → TL → qa-leader`
- **完整**：`worker → pg-leader → TL → qa-leader`

所有跨組通知都經過 TL，確保 QA 審查一定會發生。
Worker 可批量回報多個完成的小任務。

### Phase 5: 契約驗證

qa-leader 做最終檢查：後端 API、前端呼叫、Request/Response、錯誤處理是否全部符合契約。

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

### 追蹤矩陣（TRACE.md）

追蹤矩陣是雙向綁定的核心，映射：

```
上游規格節 → Req-ID → 任務 → Worker → 開發狀態 → QA 結果
```

狀態流程：`pending → in-progress → done → qa-pass / qa-fail → fixed`

每當任務狀態變更，TL 會自動更新 TRACE.md。最終 Summary 計數所有需求項的通過/延遲/未解決數量。

### 流程紀錄（PROCESS_LOG.md）

記錄關鍵事件（非詳細對話），事件類型包括：
`team-assembled`, `task-completed`, `qa-triggered`, `review-pass`, `review-fail`, `issue-reported`, `decision`, `contract-amended`, `phase-transition`

### 問題紀錄（ISSUES.md）

QA 審查失敗或契約驗證不一致時自動建立條目，記錄問題描述、相關任務/需求、解決方式。

### 交付報告（DELIVERY_REPORT.md）

Phase 6 最終產出，包含：
- 已實作項目（前端/後端分列）
- QA 結果摘要
- 契約驗證結果
- 延遲/手動項目
- 交叉引用所有追蹤文件

---

## 溝通路徑

```
          ┌──────────┐
          │    TL    │
          └────┬─────┘
        ╱      │      ╲
  ┌─────┐  ┌───┴──┐  ┌─────┐
  │explo│  │  pg  │  │ qa  │
  │-re  │  │leader│  │lead │
  └─────┘  └──┬───┘  └─────┘
            ╱  │  ╲
        ┌──┐┌──┐┌──┐
        │p1││p2││p3│
        └──┘└──┘└──┘

  ── 允許    ╳ 禁止

  ✅ TL ↔ 所有 leaders
  ✅ TL ↔ 所有 workers（TL spawn 的）
  ✅ pg-leader ↔ workers
  ❌ pg-leader → qa-leader（經 TL）
  ❌ worker → worker（經 pg-leader）
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
├── .claude-plugin/plugin.json          ← 插件元資料 (v1.3.0)
├── skills/dev-team/
│   ├── SKILL.md                        ← AI 核心指令（英文，始終載入）
│   ├── prompts/                        ← Spawn 模板（按需載入，每次只讀一個）
│   │   ├── pg-leader.md                ← 開發組管理者
│   │   ├── qa-leader.md                ← 品質保證組長
│   │   ├── explore-leader.md           ← 探索組長
│   │   └── worker.md                   ← 開發 Worker
│   └── references/                     ← 文件模板（按需載入）
│       ├── trace-template.md           ← 追蹤矩陣模板
│       ├── api-contract-template.md    ← API 契約模板
│       ├── process-log-template.md     ← 流程紀錄模板
│       ├── issues-template.md          ← 問題紀錄模板
│       └── delivery-report-template.md ← 交付報告模板
└── docs/
    └── GUIDE.zh-TW.md                  ← 本文件（中文，給人讀的）
```

---

## 常見問題

**Q: pg-leader 一定會出現嗎？**
A: 不一定。≤2 workers 時使用輕量模式，跳過 pg-leader，TL 直接管理 workers。≥3 workers 才會有 pg-leader。

**Q: 我可以一次給多份規格書嗎？**
A: 可以。TL 會分析依賴後問你要並行、序列或逐個完成。

**Q: 需要搭配 DDD 流程嗎？**
A: 不必。dev-team 可以獨立使用。但如果先用 ddd-core 做完設計再交給 dev-team 實作，效果最好。

**Q: Worker 數量可以控制嗎？**
A: TL 根據任務數量自動決定。如果你想要更多或更少，可以在 Phase 1 確認時告訴 TL。
