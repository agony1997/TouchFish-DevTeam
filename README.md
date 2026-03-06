# TouchFish-DevTeam

> 多角色 Agent 團隊協作框架 — 分離測試、三方交叉驗證 QA、LLM-native 文件架構

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Version](https://img.shields.io/badge/version-1.5.0-blue.svg)](CHANGELOG.md)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-Skill-blueviolet.svg)](https://docs.anthropic.com/en/docs/claude-code)

Claude Code 技能插件：由 Team Lead（Opus）指揮 Workers（Sonnet teammates）並行開發，
結合分離測試、三方交叉驗證 QA、嚴格檔案範圍控管，完成大型功能的全流程開發。

**完全獨立運行，無外部 plugin 依賴。**

> **與 TouchFish-Skills 的關係**：本 repo 原為 [TouchFish-Skills](https://github.com/agony1997/TouchFish-Skills) 的 `plugins/dev-team`。
> 因 dev-team 是「編制框架」（完整開發系統），與其他 6 個「管線單元」在本質上不同，故拆分為獨立 repo。

## 核心特色

- **分離測試**：test-agent（Opus）先寫測試 → Worker 只管讓測試通過，消除自我驗證偏誤
- **TDD 紀律內建**：RED-GREEN 語義直接寫進 test-agent prompt，不依賴外部 TDD skill
- **三方交叉驗證 QA**：需求 ↔ 測試、測試 ↔ 程式碼、需求 ↔ 程式碼，不信任任何單一 artifact
- **1-task-per-worker**：每個 Worker 只做一個任務就結束，確保最乾淨的 context
- **嚴格檔案範圍**：ALLOWED / READONLY / FORBIDDEN 三級控管 + Worker 完成前 `git diff` 自驗
- **LLM-native 文件格式**：`[TYPE] key=value` 格式設計以減少 token 用量（待驗證）
- **分散式日誌**：每個 agent 維護自己的 log，取代集中式追蹤文件
- **需求澄清步驟**：Phase 0 自動輸出理解摘要並確認，避免基於錯誤理解拆任務
- **Context 預算控管**：TL 當 router 不當 relay，所有 agent 只返回摘要，防止 context 爆滿
- **Compaction 恢復機制**：tl-state.md 持久化狀態，context 壓縮後可自動恢復
- **Skill 隔離指令**：明確禁止 superpowers 技能攔截，確保 dev-team 流程完整執行
- **複雜度感知排程**：P0 估算 task 複雜度（LOW/MED/HIGH），異質 wave 拆子批次，fast-first spawn 避免慢 agent 阻塞快 agent 的後續 pipeline

## 使用流程

```
Phase 0                    Phase 1           Phase 2
專案理解 + 任務規劃  ───→  API 契約    ───→  開發執行
(掃描+澄清+PLAN.md)       (CONTRACT.md)     (test→code→QA 循環)

                                                │
Phase 4              Phase 3                    │
交付           ←───  全域審查        ←──────────┘
(DELIVERY.md)       (跨任務一致性)
```

詳細設計文件與歷史紀錄請參閱 [docs/ 文件索引](docs/INDEX.md)。

## 團隊架構

| 角色 | Model | 職責 |
|------|-------|------|
| **Team Lead (TL)** | Opus | PM：需求澄清、任務規劃、API 契約、spawn 所有 agents、品質閘門 |
| **test-agent** | Opus | 分離測試：先寫測試（RED），Worker 再寫程式通過（GREEN）。寫 impl-notes 提示框架陷阱 |
| **worker-N** | Sonnet | 開發 Worker：1 任務 1 生命，寫程式碼通過測試，完成前 scope 自驗 |
| **qa-task-N** | Opus | 三方交叉驗證：需求 ↔ 測試 ↔ 程式碼 |
| **qa-global** | Opus | 全域審查：跨任務一致性、契約合規性 |
| **delivery-sub** | Sonnet | 交付報告：彙整日誌，產出 DELIVERY.md |

## 目錄結構

```
TouchFish-DevTeam/
├── skills/dev-team/
│   ├── SKILL.md                        # AI 核心指令（英文，始終載入）
│   ├── prompts/                        # Spawn 模板（按需載入）
│   │   ├── worker.md
│   │   ├── test-agent.md
│   │   ├── qa-task.md
│   │   ├── qa-global.md
│   │   └── delivery-sub.md
│   └── references/                     # 文件模板（按需載入）
│       ├── plan-template.md
│       ├── contract-template.md
│       ├── delivery-template.md
│       └── log-templates.md
├── docs/                              # 設計文件與決策紀錄
│   ├── INDEX.md                       # 文件索引（版本對照 + 完整清單）
│   ├── experiment/                    # 實驗紀錄（E0, E1, E2）
│   ├── plans/                         # 設計文件（brainstorming → design doc）
│   └── ...                            # 設計歷史文件（詳見索引）
├── README.md
├── CHANGELOG.md
└── LICENSE
```

## 設計理念

v1.5.0 經過 E1 基線測試（442 tests）、E2 對照測試（334 tests, global QA 首次通過）持續迭代。核心設計原則：

1. **完全獨立** — 移除所有外部 plugin 偵測 + 明確禁止 superpowers 技能攔截
2. **文件至上** — 不信任 LLM context/memory，PLAN.md 和 DELIVERY.md 是 agent 間溝通的唯一可靠媒介
3. **前置驗證** — Worker scope 自驗在 Worker 層面攔截違規，而非等 QA 事後抓，顯著降低重做成本
4. **穩定知識內建** — TDD RED-GREEN-REFACTOR 是穩定的，直接內建比依賴外部 skill 更可靠
5. **Context 預算控管** — TL 當 router 不當 relay，防止 context 在 Phase 3 前爆滿
6. **Compaction 恢復力** — tl-state.md 持久化關鍵狀態，context 壓縮後可自動恢復
7. **為實證留空間** — execution strategy 設計為可調參數，先上線收集基線再決定優化方向
8. **複雜度感知排程** — P0 估算 task 複雜度，異質 wave 拆子批次（fast-first），避免 parallel tool calls 被最慢 agent 阻塞

## 實驗紀錄

| 實驗 | 版本 | 結果 | 說明 |
|------|------|------|------|
| E0 | v1.2.0 | superpowers 攔截 | dev-team 從未載入，superpowers:brainstorming 取代整個流程 |
| E1 | v1.3.0 | 442/442 tests PASS | 基線測試成功，發現 7 個 issues（5 個已修復於 v1.4.0） |
| E2 | v1.4.0 | 334/334 tests PASS | v1.4.0 驗證：global QA 首次通過，TL session 快 24%，6 new observations |
| E3 | v1.5.0 | 待執行 | 驗證 complexity estimation + fast-first sub-batch spawn |

詳見 [docs/experiment/](docs/experiment/)。

## 關鍵設計決策

以下決策經 v1.1 挑戰者審查 + v1.2 批判審核 + E1 實測確立：

| 決策 | 選項 | 結論 | 理由 |
|------|------|------|------|
| 外部 plugin 耦合 | 完全獨立 / 硬耦合 / 介面契約 | **完全獨立** | 可選偵測是最差設計模式，16 種組合無法測試 |
| qa-task 模型 | Sonnet / Opus | **Opus** | 品質閘門 6 步驟複雜推理，應用最強模型 |
| Challenger 角色 | 持久 teammate / per-task sub-agent / 移除 | **移除** | 分離測試 + 三方 QA + 全域審查已覆蓋其價值 |
| Worker 隔離 | git worktree / 文字聲明 + QA 驗證 | **文字聲明 + QA** | teammate 模式下 worktree 複雜度高，file scope + parallel awareness + QA = 多層防禦 |
| Metrics 記錄 | Sub-agent 自報 / TL 記錄 | **TL 記錄** | Sub-agent 取不到自身 token 消耗，TL 可從 Agent tool 回傳取得 |
| TL context 管理 | 全量轉發 / 路徑轉發 | **路徑轉發** | E1 證實全量轉發導致 context compaction，TL 當 router 不當 relay |
| P3 issue 修復 | TL 直接修 / 建 fix task | **建 fix task** | E1 中 TL 自修膨脹 context，且違反 scope 隔離原則 |

## 待驗證假設

以下設計基於理論推論，尚無完整實測數據支撐：

| 假設 | 理論依據 | 驗證方式 |
|------|---------|---------|
| LLM-native 格式省 token | `[TYPE] key=value` 比 Markdown 表頭/分隔線更精簡 | 同一份文件兩種格式的 token 計數對比 |
| 前置 scope 自驗降低重做成本 | Worker 層攔截 vs QA 事後抓 = 省 test→work→qa 循環 | 記錄有/無自驗時的 QA FAIL 率比較 |
| ≤ 3 Workers 併發為最佳值 | 暫定經驗值，無實測依據 | 不同併發數下的完成時間 / token / 品質比較 |
| 2 msgs no reply = crash | 比時間門檻更好的存活偵測 | 實際執行中的誤判率觀察 |
| 分離測試消除自我驗證偏誤 | Opus 寫測試、Sonnet 寫實作，角色分離 | 有/無分離測試的 QA 通過率比較 |

## License

[MIT](LICENSE)
