# 實驗紀錄

此目錄存放 dev-team skill 的對照實驗紀錄。

## 實驗清單

| 實驗 | 驗證假設 | 狀態 |
|------|---------|------|
| E1 | LLM-native 格式省 token | 待執行 |
| E2 | 前置 scope 自驗降低重做成本 | 待執行 |
| E3 | ≤ 3 Workers 併發為最佳值 | 待執行 |
| E4 | 分離測試消除自我驗證偏誤 | 待執行 |
| E5 | 2 msgs no reply = crash（觀察性） | 待執行 |

## 執行順序

1. Round 1: 正常流程（M 組）→ 基線
2. Round 2: E1 → token 計數
3. Round 3: E4 → 分離測試 vs 自寫
4. Round 4: E2 → scope 自驗
5. Round 5: E3 → 併發數

## 紀錄格式

每次實驗產出一份 `E{N}-{YYYY-MM-DD}-run{N}.md`，格式見 `experiment-template.md`。
