# dev-team 改版驗證 Checklist

> 每次修改 skill 檔案後，依此清單逐項驗證。
> 來源：v1.0.0 設計決策驗證方式 + v1.1.0 批判審核補充

---

## 結構一致性

- [ ] SKILL.md Phase 編號連續（P0-P4）且無跳號
- [ ] SKILL.md Team Structure 表與各 prompt spawn config 的 model 一致
- [ ] README.md 團隊架構表與 SKILL.md 一致
- [ ] docs/INDEX.md 列出所有 docs/ 文件且連結正確

## 外部依賴

- [ ] 全文搜尋 `superpowers`、`openspec`、`explorer`、`reviewer` — 應為零結果（docs/ 歷史文件除外）
- [ ] plan-template.md 無 `[DETECT]` 行
- [ ] SKILL.md 無 plugin 偵測邏輯

## 測試與 QA

- [ ] test-agent.md 有 TDD DISCIPLINE 段落且 RED-GREEN 語義正確
- [ ] test-agent.md RULES 包含模糊需求停止規則和多斷言規則
- [ ] qa-task.md 有完整六步驟（三方驗證 + standards + quality + scope）
- [ ] qa-global.md 有 CHECK 6（可選整合測試）

## Worker 與 Scope

- [ ] worker.md 有 scope 自驗步驟（`git diff --name-only` + ALLOWED 比對）在 completion sequence 中
- [ ] SKILL.md Phase 2 有 QA FAIL → fix task 含失敗摘要的步驟

## 交付

- [ ] delivery-template.md 只有 7 section 且無 Agent Metrics
- [ ] delivery-template.md 支援部分完成標記（Completion Status + failed 狀態）
- [ ] delivery-sub.md 引用 7-section 且有部分完成規則

## 日誌與 Metrics

- [ ] sub-agent prompts（test-agent、qa-task、qa-global、delivery-sub）無 [METRICS] 自報行
- [ ] log-templates.md sub-agent 區段無 [METRICS] 行
- [ ] log-templates.md tl.log 有 sub-agent-metrics 事件類型和格式
- [ ] Worker log 保留 [METRICS] n/a（客觀限制，不移除）

## 文件與宣稱

- [ ] README.md 無未經驗證的具體數字（如 ~40%、10 倍）
- [ ] design-rationale.md 無未經驗證的具體數字
