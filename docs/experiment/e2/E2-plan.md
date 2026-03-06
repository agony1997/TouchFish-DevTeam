# E2 驗證測試計畫

- **日期**: 2026-03-06
- **版本**: dev-team v1.4.0
- **對照**: E1（v1.3.0, 同一 spec + skeleton）
- **目標**: 驗證 v1.4.0 的 I1-I7 修復是否生效

## 測試條件

- 分支: `experiment/E2-v1.4.0`
- Spec: TaskFlow M1+M2+M3+M6（與 E1 相同）
- Skeleton: 與 E1 相同的乾淨骨架
- 觸發方式: `/dev-team`（透過 Skill 工具，非 Read）

## 觀察項目

| # | 對應 Issue | 觀察指標 | E1 基線 | 預期改善 |
|---|-----------|---------|---------|---------|
| 1 | I1 context compaction | TL context 是否觸及壓縮 | P3 觸發壓縮 (161k/200k) | 不觸發或大幅降低 |
| 2 | I1 minimal return | agent 返回內容大小 | 全量返回 | 僅摘要（路徑+計數） |
| 3 | I1 tl-state.md | compaction 後能否恢復 | agent ID 遺失 | 自動恢復 |
| 4 | I1 TL 不自修 | P3 issue 是否走 fix task | TL 手動 Edit 4 檔 | 建 fix task → Worker 修 |
| 5 | I1 impl-notes | Worker 偵錯循環是否減少 | Worker-7 卡 24 min | 偵錯時間縮短 |
| 6 | I2 parallel awareness | Worker 是否只跑自己的測試 | Worker 移除無關測試檔 | 不觸碰其他 Worker 檔案 |
| 7 | I3 AC cross-check | CONTRACT 是否遺漏 endpoint | v1 遺漏 status filter | 遺漏減少或消除 |
| 8 | I4 cross-cutting | 錯誤處理格式是否一致 | Auth vs Task 格式不同 | 統一格式 |
| 9 | I7 test syntax | test-agent 測試是否有語法錯誤 | 2 個 test-agent 有錯 | 無語法錯誤 |
| 10 | I5 skill loading | 是否透過 Skill 工具載入 | Read 直接讀取 | Skill 工具載入 |

## 成功標準

1. 所有測試通過（≥ E1 的 442 tests）
2. TL context 不觸發壓縮
3. 無 test-agent 語法錯誤
4. 觀察項目中至少 7/10 符合預期改善

## 分析方式

測試完成後比對 E1 vs E2：
- TL log 對照（context 大小、phase 時間）
- agent 數量與重做次數
- Worker 偵錯循環時間
- CONTRACT 版本變動次數
- 錯誤處理一致性
