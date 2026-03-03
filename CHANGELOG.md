# 變更紀錄

此檔案將記錄本專案的所有重大變更。

格式基於 [Keep a Changelog](https://keepachangelog.com/zh-CN/1.0.0/)，
並且本專案遵循 [語意化版本 (Semantic Versioning)](https://semver.org/lang/zh-TW/)。

> **版本重設說明**：本專案前身為 [TouchFish-Skills](https://github.com/agony1997/TouchFish-Skills) 的 `plugins/dev-team`。
> 原先版本號為隨手分配（v1.0→v1.1→v1.2→v1.3→v2.0→v2.1→v2.2→v3.0），不符合語意化版本原則。
> 現已根據 git 歷史重新分配版本號：0.x 為原型快速迭代期，1.0.0 為首個正式發布。
> 設計文件中出現的「v3.0」即為本版本號重設後的 v1.0.0。

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
- **探測增強**：Phase 0 偵測 6 個整合目標（explorer、PROJECT_MAP、TDD、reviewer、.standards、OpenSpec）
- **DELIVERY 升級**：從簡要報告升級為 8 區塊開發回歸文件
- **獨立 repo 基礎**：README（中/英）、LICENSE、CHANGELOG、使用指南

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
