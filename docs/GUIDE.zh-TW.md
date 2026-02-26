# dev-team 技能使用指南

> 版本：1.1.0 | 最後更新：2026-02-26

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

### v1.0 → v1.1 關鍵變更

| 項目 | v1.0 | v1.1 |
|------|------|------|
| 誰 spawn workers | pg-leader | **TL**（控制權集中） |
| QA 觸發路徑 | pg-leader → qa-leader | **pg-leader → TL → qa-leader** |
| pg-leader 角色 | 可能自己做完所有事 | **嚴格管理者，禁止自己實作** |
| 多規格書 | 不支援 | **支援並行/序列/單一專精模式** |
| 通訊紀律 | 無 | **所有 agent 必須回應上級訊息** |

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
2. **多份規格書時**，TL 會分析領域間的依賴關係，問你要：
   - 並行開發（領域獨立）
   - 序列開發（有依賴）
   - 單一專精（逐個完成）
3. 拆分任務，問你確認

### Phase 2: API 契約

TL 定義前後端共用的 API 契約，問你確認。確認後成為開發基準。

### Phase 3: 組建團隊

TL 根據任務數量決定團隊規模：

| 任務數量 | Workers |
|---------|---------|
| ≤3 | 不 spawn，pg-leader 自己做 |
| 4-8 | 至少 2 workers |
| ≥9 | 至少 3 workers |
| 前端+後端 | 至少各 1 |

**TL 直接 spawn 所有 workers**，pg-leader 只負責管理指派。

### Phase 4: 流水線開發

```
開發                          審查
┌──────┐                    ┌──────┐
│ pg-1 │──完成──→ pg-leader │ qa   │
│ pg-2 │         ──回報──→  │leader│
│ pg-3 │              TL    │      │
│ pg-4 │         ──通知──→  │ sub- │
└──────┘                    │agents│
                            └──────┘
```

通知鏈：`worker → pg-leader → TL → qa-leader → sub-agent 審查 → qa-leader → TL`

所有跨組通知都經過 TL，確保 QA 審查一定會發生。

### Phase 5: 契約驗證

qa-leader 做最終檢查：後端 API、前端呼叫、Request/Response、錯誤處理是否全部符合契約。

### Phase 6: 交付

TL 產出交付報告，關閉團隊。**不會自動 commit/push**，由你決定。

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
├── .claude-plugin/plugin.json          ← 插件元資料 (v1.1.0)
├── skills/dev-team/
│   ├── SKILL.md                        ← AI 核心指令（英文，始終載入）
│   └── prompts/                        ← Spawn 模板（按需載入，每次只讀一個）
│       ├── pg-leader.md                ← 開發組管理者
│       ├── qa-leader.md                ← 品質保證組長
│       ├── explore-leader.md           ← 探索組長
│       └── worker.md                   ← 開發 Worker
└── docs/
    └── GUIDE.zh-TW.md                  ← 本文件（中文，給人讀的）
```

---

## 常見問題

**Q: pg-leader 可以自己寫程式碼嗎？**
A: 只有在任務 ≤3 且 TL 沒派 workers 時可以。否則必須指派給 workers。

**Q: 我可以一次給多份規格書嗎？**
A: 可以。TL 會分析依賴後問你要並行、序列或逐個完成。

**Q: 需要搭配 DDD 流程嗎？**
A: 不必。dev-team 可以獨立使用。但如果先用 ddd-core 做完設計再交給 dev-team 實作，效果最好。

**Q: Worker 數量可以控制嗎？**
A: TL 根據任務數量自動決定。如果你想要更多或更少，可以在 Phase 1 確認時告訴 TL。
