# 實驗紀錄

此目錄存放 dev-team skill 的實驗紀錄與對照實驗規劃。

## 已完成實驗

| 實驗 | 日期 | 版本 | 結果 | 說明 |
|------|------|------|------|------|
| [E0](e0/) | 2026-03-03 | v1.2.0 | FAIL | superpowers 攔截 dev-team，Skill 工具從未被調用 → v1.3.0 修復 |
| [E1](e1/) | 2026-03-06 | v1.3.0 | PASS (442/442) | 基線測試，24 agents，~79min，7 issues 發現 → v1.4.0 修復 5 個 |
| [E2](e2/) | 2026-03-06 | v1.4.0 | PASS (334/334) | v1.4.0 驗證測試，~60min TL session，global QA 首次通過，6 new observations |

### E2 vs E1 關鍵改善
- Global QA：FAIL→fix→PASS → **首次 PASS**
- P3 TL 修復：5 檔案 → **0**
- TL 介入：7 次 → **1 次**
- CONTRACT 偏移：v1→v2 → **無偏移**
- 新發現：pipeline idle 20%、parallel tool calls 阻塞、Vue task 3-4x 慢、compaction ~168K 地板

## 待執行對照實驗

| 實驗 | 驗證假設 | 狀態 |
|------|---------|------|
| E-token | LLM-native 格式省 token | 待執行 |
| E-scope | 前置 scope 自驗降低重做成本 | 待執行 |
| E-concurrent | ≤ 3 Workers 併發為最佳值 | 待執行 |
| E-separated | 分離測試消除自我驗證偏誤 | 待執行 |
| E-timeout | 2 msgs no reply = crash（觀察性） | 待執行 |

## 紀錄格式

每次實驗產出一份紀錄，格式見 `experiment-template.md`。
