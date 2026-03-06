# 實驗紀錄

此目錄存放 dev-team skill 的實驗紀錄與對照實驗規劃。

## 已完成實驗

| 實驗 | 日期 | 版本 | 結果 | 說明 |
|------|------|------|------|------|
| [E0](e0/) | 2026-03-03 | v1.2.0 | FAIL | superpowers 攔截 dev-team，Skill 工具從未被調用 → v1.3.0 修復 |
| [E1](e1/) | 2026-03-06 | v1.3.0 | PASS (442/442) | 基線測試，24 agents，~1h19m，7 issues 發現 → v1.4.0 修復 5 個 |

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
