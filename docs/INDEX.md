# docs/ 文件索引

> 本目錄存放 dev-team 技能的設計文件、決策紀錄與審查報告。
> 檔名格式：`{date}-v{version}-{category}-{descriptor}.md`
> 類別：`review`（審查）、`design`（設計）、`log`（紀錄）、`plan`（計畫）

## 文件版本對照

本專案從 TouchFish-Skills 拆出後重設版本號。歷史文件內文中的版本對應關係：

| 文件內文版本 | 實際對應 | 說明 |
|-------------|---------|------|
| v2.2.0 | v0.4.0 | task pool + Challenger 架構 |
| v3.0 | v1.0.0 | 首個正式發布（完全重設計） |
| v1.1.0 | v1.1.0 | 分離測試 + 三方 QA + 獨立 repo |
| v1.2.0 | v1.2.0 | 批判審核後改進 |
| v1.3.0 | v1.3.0 | Skill 隔離指令（反 superpowers 截斷） |
| v1.4.0 | v1.4.0 | E1 基線測試後改進（context 預算、compaction 恢復、impl-notes） |

## 當前有效文件

| 文件 | 類別 | 說明 |
|------|------|------|
| [v1.1 設計決策紀錄](2026-03-03-v1.1-design-rationale.md) | design | 挑戰者審查後的四項主要變更 + 兩項附帶改進 |
| [v1.1 批判審核報告](2026-03-03-v1.1-review-critique.md) | review | 系統性批判審核，含架構、prompt、文件一致性等質疑（決策完成） |
| [改版驗證 Checklist](checklist-release.md) | checklist | 每次改版後的逐項驗證清單 |

## 實驗紀錄

| 實驗 | 日期 | 版本 | 結果 | 位置 |
|------|------|------|------|------|
| E0 | 2026-03-03 | v1.2.0 | superpowers 攔截，dev-team 從未載入 | [experiment/e0/](experiment/e0/) |
| E1 | 2026-03-06 | v1.3.0 | 442/442 PASS，7 issues → 5 fixed in v1.4.0 | [experiment/e1/](experiment/e1/) |

詳見 [experiment/README.md](experiment/README.md)。

## 設計歷史（v0.4 → v1.0 設計過程）

以下文件記錄了完整設計演進過程，按時間順序排列：

| # | 文件 | 類別 | 說明 |
|---|------|------|------|
| 1 | [v0.4 質疑報告](2026-03-02-v0.4-review-challenger.md) | review | 對 v0.4 架構的 15 個系統性質疑 |
| 2 | [v0.4 比對分析](2026-03-02-v0.4-review-comparison.md) | review | 與 OpenSpec、superpowers 的架構比較 |
| 3 | [v1.0 設計探索](2026-03-02-v1.0-design-exploration.md) | design | 35 個設計決策的探索過程（摘要版） |
| 4 | [v1.0 完整對話紀錄](2026-03-02-v1.0-log-dialogue.md) | log | 94 組 Q&A 的完整對話（60KB，供追溯用） |
| 5 | [v1.0 設計文件](2026-03-02-v1.0-design.md) | design | v1.0 的完整設計文件 |
| 6 | [v1.0 實作計畫](2026-03-02-v1.0-plan-implementation.md) | plan | v1.0 的逐步實作任務清單 |
| 7 | [v1.0 外部審查摘要](2026-03-03-v1.0-review-external.md) | review | 從 TouchFish-Skills 全套審查報告萃取的 dev-team 評價 |
