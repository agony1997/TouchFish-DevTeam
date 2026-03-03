# dev-team 技能審查摘要

> **審查日期**：2026-03-03
> **來源**：從 TouchFish-Skills 全套審查報告萃取 dev-team 相關評價
> **審查者**：Claude Opus 4.6（Claude Skill 建立者 / 系統架構師視角）

---

## 評分

| 工作流設計 | Prompt 品質 | 可維護性 | Agent 協作 | 完整度 | 綜合 |
|-----------|------------|---------|-----------|--------|------|
| A+ | A+ | A- | A+ | A- | **A+** |

dev-team 是 TouchFish-Skills 七個插件中設計水準最高的，也是唯一獲得 A+ 綜合評價的插件。

---

## 核心優點

### 1. 分離測試設計

test-agent（Opus）先寫測試 → Worker 只管讓測試通過。這解決了 agent 自我驗證的核心問題：如果 Worker 自己寫測試，測試傾向永遠通過。

### 2. 三方交叉驗證 QA

QA 不信任任何單一 artifact：
- 需求 ↔ 測試（測試涵蓋需求嗎？）
- 測試 ↔ 程式碼（程式碼真的通過測試嗎？）
- 需求 ↔ 程式碼（程式碼做的是需求要的嗎？）

### 3. 嚴格 Scope Enforcement

ALLOWED / READONLY / FORBIDDEN 三級檔案權限，防止 Worker 互相踩踏。違反 scope 會導致 QA 失敗和任務被拒。

### 4. LLM-native 格式

`[TYPE] key=value | key=value` 比 Markdown 表格更省 token，LLM 讀寫更穩定。

### 5. Worker Prompt 品質極高

任務明確、通訊紀律嚴格、scope 違規後果清楚。是全套插件中 prompt engineering 最成熟的。

---

## 可改善之處

### 1. 中斷恢復策略缺失

dev-team 有 file-as-memory（PLAN/CONTRACT/logs），理論上可恢復，但缺少明確的「resume from Phase N」指示。

### 2. 與 ddd-core 的定位重疊

ddd-core Phase 4（Implementation Planning）和 dev-team Phase 1 都做需求分析 + 任務拆解。走完 DDD 後用 dev-team，Phase 1 可能重複工作。建議加入快速通道。

### 3. 成本可見性不足

可能 spawn 10+ agents，但缺少成本估算指引。對使用者決策很重要。

### 4. Prompt 變數空值 fallback

大部分 `{variable}` 佔位符沒有空值處理（`{contract_path}` 有 `skip if "none"` 但其他沒有）。

---

## 三套工具中的定位

在 TouchFish-Skills / SuperPowers / OpenSpec 三套工具的對比中，dev-team 的獨有優勢：

| 能力 | dev-team 獨有原因 |
|------|------------------|
| Agent Teams 並行開發 | SuperPowers 只有 sub-agent（無 teammate 通訊） |
| 分離測試 | SuperPowers 的 TDD 是 Worker 自己寫測試（自我驗證問題） |
| 三方交叉 QA | SuperPowers 兩階段 review 缺少「需求 ↔ 測試」維度 |
| LLM-native 文件格式 | SuperPowers / OpenSpec 都用 Markdown |

**定位**：大規模功能的分工執行引擎（Agent Teams + 分離測試 + 三方 QA + 嚴格 scope）。

最佳搭配：
- **探索階段**：SuperPowers brainstorming + TouchFish-Skills explorer
- **實作階段**：dev-team + SuperPowers TDD 紀律注入
- **驗證階段**：SuperPowers verification + TouchFish-Skills reviewer
- **交付階段**：TouchFish-Skills git-nanny + OpenSpec archive
