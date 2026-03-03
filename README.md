# TouchFish-DevTeam

> 多角色 Agent 團隊協作框架 — 分離測試、三方交叉驗證 QA、LLM-native 文件架構

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Version](https://img.shields.io/badge/version-1.0.0-blue.svg)](.claude-plugin/plugin.json)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-Skill-blueviolet.svg)](https://docs.anthropic.com/en/docs/claude-code)

**繁體中文** | [English](README.en.md)

Claude Code 技能插件：由 Team Lead（Opus）指揮 Workers（Sonnet teammates）並行開發，
結合分離測試、三方交叉驗證 QA、嚴格檔案範圍控管，完成大型功能的全流程開發。

> **與 TouchFish-Skills 的關係**：本 repo 原為 [TouchFish-Skills](https://github.com/agony1997/TouchFish-Skills) 的 `plugins/dev-team`。
> 因 dev-team 是「編制框架」（完整開發系統），與其他 6 個「管線單元」在本質上不同，故拆分為獨立 repo。
> TouchFish-Skills 仍包含 ddd-core、git-nanny、reviewer、spec-to-md、md-to-code、explorer 等 6 個工作流型插件。

## 核心特色

- **分離測試**：test-agent（Opus）先寫測試 → Worker 只管讓測試通過，消除自我驗證偏誤
- **三方交叉驗證 QA**：需求 ↔ 測試、測試 ↔ 程式碼、需求 ↔ 程式碼，不信任任何單一 artifact
- **1-task-per-worker**：每個 Worker 只做一個任務就結束，確保最乾淨的 context
- **嚴格檔案範圍**：ALLOWED / READONLY / FORBIDDEN 三級控管，防止 Worker 互相踩踏
- **LLM-native 文件格式**：`[TYPE] key=value` 格式比 Markdown 省 ~40% tokens
- **分散式日誌**：每個 agent 維護自己的 log，取代集中式追蹤文件

## 安裝

```bash
# 直接在 Claude Code 中安裝
claude mcp add-plugin https://github.com/agony1997/TouchFish-DevTeam
```

## 使用流程

```
Phase 0        Phase 1              Phase 2           Phase 3
專案偵察  ───→  需求分析+任務規劃 ───→  API 契約    ───→  開發執行
(探索專案)     (PLAN.md)            (CONTRACT.md)     (test→code→QA 循環)

                                                         │
Phase 5              Phase 4                             │
交付           ←───  全域審查        ←───────────────────┘
(DELIVERY.md)       (跨任務一致性)
```

詳細使用說明請參閱 [使用指南](docs/GUIDE.zh-TW.md)。

## 團隊架構

| 角色 | Model | 職責 |
|------|-------|------|
| **Team Lead (TL)** | Opus | PM：需求分析、任務規劃、API 契約、spawn 所有 agents、品質閘門 |
| **test-agent** | Opus | 分離測試：先寫測試，Worker 再寫程式通過 |
| **worker-N** | Sonnet | 開發 Worker：1 任務 1 生命，寫程式碼通過測試 |
| **qa-task-N** | Sonnet | 三方交叉驗證：需求 ↔ 測試 ↔ 程式碼 |
| **qa-global** | Opus | 全域審查：跨任務一致性、契約合規性 |
| **delivery-sub** | Sonnet | 交付報告：彙整日誌，產出 DELIVERY.md |

## 目錄結構

```
TouchFish-DevTeam/
├── .claude-plugin/plugin.json          # 插件定義
├── skills/dev-team/
│   ├── SKILL.md                        # AI 核心指令（英文，始終載入）
│   ├── prompts/                        # Spawn 模板（按需載入）
│   │   ├── worker.md
│   │   ├── test-agent.md
│   │   ├── qa-task.md
│   │   ├── qa-global.md
│   │   └── delivery-sub.md
│   └── references/                     # 文件模板（按需載入）
│       ├── plan-template.md
│       ├── contract-template.md
│       ├── delivery-template.md
│       └── log-templates.md
├── docs/
│   └── GUIDE.zh-TW.md                 # 中文使用指南
├── README.md
├── README.en.md
├── CHANGELOG.md
└── LICENSE
```

## 搭配的外部插件

| 插件 | 來源 | 搭配方式 |
|------|------|---------|
| [TouchFish-Skills](https://github.com/agony1997/TouchFish-Skills) | agony1997 | explorer 偵察、reviewer 規範審查、ddd-core → dev-team 串接 |
| [superpowers](https://github.com/obra/superpowers-marketplace) | obra | brainstorming、TDD 鐵律、systematic debugging、verification |
| [OpenSpec](https://github.com/Fission-AI/OpenSpec) | Fission-AI | delta spec 作為需求來源、archive 歸檔 |

> 所有外部插件皆為可選。dev-team 可獨立使用，偵測到外部插件時自動升級體驗。

## License

[MIT](LICENSE)
