# TouchFish-DevTeam

> Multi-role Agent Team Collaboration Framework — separated testing, three-way cross-verification QA, LLM-native file architecture

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Version](https://img.shields.io/badge/version-1.0.0-blue.svg)](.claude-plugin/plugin.json)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-Skill-blueviolet.svg)](https://docs.anthropic.com/en/docs/claude-code)

[繁體中文](README.md) | **English**

A Claude Code skill plugin: Team Lead (Opus) orchestrates Workers (Sonnet teammates) for parallel development,
combining separated testing, three-way cross-verification QA, and strict file scope enforcement for end-to-end feature delivery.

> **Relation to TouchFish-Skills**: This repo was originally `plugins/dev-team` in [TouchFish-Skills](https://github.com/agony1997/TouchFish-Skills).
> dev-team is an "orchestration framework" (complete development system), fundamentally different from the other 6 "pipeline units", hence the split into a standalone repo.
> TouchFish-Skills still contains 6 workflow plugins: ddd-core, git-nanny, reviewer, spec-to-md, md-to-code, and explorer.

## Key Features

- **Separated Testing**: test-agent (Opus) writes tests first → Worker only makes tests pass, eliminating self-verification bias
- **Three-way Cross-verification QA**: Requirements ↔ Tests, Tests ↔ Code, Requirements ↔ Code — trusts no single artifact
- **1-task-per-worker**: Each Worker handles one task then shuts down, ensuring cleanest possible context
- **Strict File Scope**: ALLOWED / READONLY / FORBIDDEN three-tier enforcement prevents Workers from stepping on each other
- **LLM-native File Format**: `[TYPE] key=value` saves ~40% tokens compared to Markdown tables
- **Distributed Logging**: Each agent maintains its own log file, replacing centralized tracking

## Installation

```bash
# Install directly in Claude Code
claude mcp add-plugin https://github.com/agony1997/TouchFish-DevTeam
```

## Workflow

```
Phase 0        Phase 1              Phase 2           Phase 3
Recon     ───→ Requirements +   ───→ API Contract ───→ Dev Execution
(Explore)      Task Planning         (CONTRACT.md)     (test→code→QA loop)
               (PLAN.md)
                                                         │
Phase 5              Phase 4                             │
Delivery       ←──── Global Review   ←───────────────────┘
(DELIVERY.md)        (Cross-task)
```

See [Usage Guide](docs/GUIDE.zh-TW.md) for details.

## Team Structure

| Role | Model | Responsibility |
|------|-------|----------------|
| **Team Lead (TL)** | Opus | PM: requirements, task planning, API contract, spawns all agents, quality gate |
| **test-agent** | Opus | Separated testing: writes tests before Worker codes |
| **worker-N** | Sonnet | Dev Worker: 1 task per lifetime, writes code to pass tests |
| **qa-task-N** | Sonnet | Three-way cross-verification: Requirements ↔ Tests ↔ Code |
| **qa-global** | Opus | Global review: cross-task consistency, contract compliance |
| **delivery-sub** | Sonnet | Delivery report: compiles logs into DELIVERY.md |

## Directory Structure

```
TouchFish-DevTeam/
├── .claude-plugin/plugin.json          # Plugin definition
├── skills/dev-team/
│   ├── SKILL.md                        # AI core instructions (English, always loaded)
│   ├── prompts/                        # Spawn templates (on-demand)
│   │   ├── worker.md
│   │   ├── test-agent.md
│   │   ├── qa-task.md
│   │   ├── qa-global.md
│   │   └── delivery-sub.md
│   └── references/                     # Document templates (on-demand)
│       ├── plan-template.md
│       ├── contract-template.md
│       ├── delivery-template.md
│       └── log-templates.md
├── docs/
│   └── GUIDE.zh-TW.md                 # Chinese usage guide
├── README.md
├── README.en.md
├── CHANGELOG.md
└── LICENSE
```

## Companion Plugins

| Plugin | Source | Integration |
|--------|--------|-------------|
| [TouchFish-Skills](https://github.com/agony1997/TouchFish-Skills) | agony1997 | explorer recon, reviewer standards, ddd-core → dev-team pipeline |
| [superpowers](https://github.com/obra/superpowers-marketplace) | obra | brainstorming, TDD discipline, systematic debugging, verification |
| [OpenSpec](https://github.com/Fission-AI/OpenSpec) | Fission-AI | delta spec as requirements source, archive |

> All companion plugins are optional. dev-team works standalone and auto-enhances when companions are detected.

## License

[MIT](LICENSE)
