# E3 驗證測試計畫

- **日期**: 2026-03-09
- **版本**: dev-team v1.5.0
- **對照**: E2（v1.4.0, 同一 spec + skeleton）
- **目標**: 驗證 v1.5.0 的 complexity estimation + sub-batch spawn 是否生效

## 測試條件

- 分支: `E3-v1.5`（從 master 建立，skeleton 與 E1/E2 相同）
- Spec: TaskFlow M1+M2+M3+M6（與 E1/E2 相同）
- Skeleton: 與 E1/E2 相同的乾淨骨架（master 上的 taskflow/）
- 工作目錄: `taskflow/`
- 觸發方式: `/dev-team`（透過 Skill 工具）
- 唯一變數: skill 版本 v1.4.0 → v1.5.0

## 提示詞

在 `taskflow/` 目錄下的新 Claude Code session 中輸入：

```
/dev-team 實作 TaskFlow M1+M2+M3+M6，specs 路徑：taskflow/docs/specs/M1-auth.md、M2-task-crud.md、M3-task-state-machine.md、M6-task-list-page.md
```

> **注意**: E2 session-03 因先觸發 brainstorming skill 而失敗。
> `/dev-team` 前綴應直接觸發 Skill tool 載入 dev-team，跳過 brainstorming。
> 若仍觸發 brainstorming，手動中斷並重新輸入。

## 觀察項目

### A. v1.5.0 核心驗證（complexity + sub-batch）

| # | 觀察指標 | E2 基線 | 預期 (v1.5.0) | 如何驗證 |
|---|---------|---------|-------------|---------|
| A1 | PLAN 含 complexity 欄位 | 無 | 每個 `[TASK]` 有 `complexity=LOW/MED/HIGH` + `reason` | 讀 PLAN 檔 |
| A2 | WAVE 拆 sub-batch | `2=T2,T3,T7` | `2a=T2,T3 \| 2b=T7`（或類似拆法） | 讀 PLAN 檔 |
| A3 | TL 按 sub-batch 順序 spawn | Wave 2 全部 parallel | 先完成 batch 2a 再 spawn batch 2b | TL log / session |
| A4 | backend critical path 提早 | T4 start ≈ t+18min | T4 start ≈ t+10min（-8min） | TL log timeline |

### B. 延續觀察（E2 留觀項）

| # | 觀察指標 | E2 基線 | 預期 | 備註 |
|---|---------|---------|------|------|
| B1 | Pipeline idle time | ~12min / 60min (20%) | 因 sub-batch 間接改善 | E2-I1，未直接修復 |
| B2 | Worker-7 (Vue) 耗時 | ~13min | 穩定 ~13min（不受 sub-batch 影響） | E2-I2，未修 |
| B3 | qa-global 重跑全測試 | 重跑全部 247+87 | 仍重跑（未修） | E2-I5，留觀 |
| B4 | Compaction | 1次, 168K, 無中斷 | ≤1次, 無中斷 | E2-I4 |
| B5 | TL session 時長 | ~60min | ≤55min（sub-batch 減少閒置） | 總體效率指標 |

### C. 回歸檢查（v1.4.0 修復不能退化）

| # | v1.4.0 Fix | E2 狀態 | 不應退化 |
|---|-----------|---------|---------|
| C1 | minimal return | EFFECTIVE | agent 仍只返回摘要 |
| C2 | parallel awareness | EFFECTIVE | Worker 不觸碰其他 Worker 檔案 |
| C3 | CONTRACT AC cross-check | EFFECTIVE | CONTRACT v1 含全部 endpoint |
| C4 | cross-cutting concerns | EFFECTIVE | 錯誤格式一致 |
| C5 | tl-state.md recovery | EFFECTIVE | compaction 後 TL 恢復 |

## 成功標準

1. **A1+A2**: PLAN 正確產出 complexity + sub-batch（v1.5.0 格式驗證）
2. **A3**: TL 實際按 sub-batch 順序執行（非全部 parallel）
3. **A4**: backend critical path 提早 ≥5min（相對 E2）
4. **B5**: TL session 時長 ≤ E2 (60min)
5. **C1-C5**: 無回歸（v1.4.0 修復全部保持 EFFECTIVE）
6. 所有測試通過（≥ E2 的 334 tests）

## 分析方式

測試完成後比對 E2 vs E3：
- PLAN 檔對比（complexity 欄位 + wave 格式）
- TL log timeline 重建（每個 agent 的 spawn/return 時間）
- sub-batch 切換點確認（TL 何時從 batch a 切換到 batch b）
- worker-7 獨立耗時（排除 blocking 影響後的純淨時間）
- 總 idle time 計算
- 回歸指標逐項確認
