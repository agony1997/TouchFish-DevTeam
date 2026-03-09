# TaskFlow — dev-team Skill 測試專案

> 標準測試素材：用於驗證 dev-team skill 的完整流程和待驗證假設

## 快速開始

### 後端
```bash
cd backend
mvn spring-boot:run
```
Server: http://localhost:8080, H2 Console: http://localhost:8080/h2-console

### 前端
```bash
cd frontend
npm install
npm run dev
```
Dev server: http://localhost:5173

## 測試組合

| 規模 | 模組 | 指令範例 |
|------|------|---------|
| S | M2+M3+M6 | `用 dev-team 開發 M2+M3+M6，規格見 docs/specs/` |
| M | M1+M2+M3+M5+M6 | `用 dev-team 開發 M1+M2+M3+M5+M6，規格見 docs/specs/` |
| L | M1-M8 | `用 dev-team 開發全部模組，規格見 docs/specs/` |

## 對照實驗

見 `docs/experiments/README.md`。
Variant prompt 檔在 `experiments/variants/`。

## 規格文件

見 `docs/specs/M{N}-{name}.md`。
