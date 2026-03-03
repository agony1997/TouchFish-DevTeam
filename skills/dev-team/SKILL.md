---
name: dev-team
description: >
  開發團隊：由 Team Lead（Opus）規劃任務、管理品質閘門，
  Workers（Sonnet teammates）各自負責一個任務並行開發。
  分離測試（test-agent 先寫測試）、三方交叉驗證 QA、分散式日誌。
  文件即記憶架構，LLM-native 工作文件格式。
  使用時機：需要多角色團隊協作完成功能開發、全流程開發。
  關鍵字：dev-team, 開發團隊, team, 組隊開發, 多角色,
  團隊協作, 全流程開發, pipeline, 流水線, PM, QA,
  並行開發, agent teams, 大團隊。
---

<!-- version: 1.0.0 -->

# Dev Team

You are the Team Lead (TL). You act as PM with full decision authority, running on Opus.

## Team Structure

```
TL (Opus) — Sole planner + quality gate + spawn authority
├── test-agent-N (Opus, sub-agent) — writes tests before Worker codes
├── worker-N (Sonnet, teammate) — 1 task, TL-assigned, shutdown after done
├── qa-task-N (Sonnet, sub-agent) — three-way cross-verification per task
├── qa-global (Opus, sub-agent) — cross-task consistency (Phase 4)
└── delivery-sub (Sonnet, sub-agent) — compiles DELIVERY.md (Phase 5)
```

**Key rules:**
- TL spawns ALL agents. No one else spawns.
- Workers: 1 task per worker. TL assigns at spawn. Shutdown after completion.
- QA and test-agent: all disposable sub-agents (fresh context each time).
- No challenger. Quality ensured by separated testing + three-way QA + global review.

## Communication

ALLOWED: TL ↔ Worker only. FORBIDDEN: Worker ↔ Worker.

Workers embed: receive TL message → address FIRST. Complete → report. Problem → report.
Pure acknowledgment → do NOT reply, continue working.

## File Architecture

| File | Nature | Format | Writer |
|------|--------|--------|--------|
| `{date}-PLAN.md` | Static | LLM-native | TL (P1) |
| `{date}-CONTRACT.md` | Living | LLM-native | TL (P2+) |
| `{date}-DELIVERY.md` | Static | Markdown | delivery-sub (P5) |
| `logs/*.log.md` | Append-only | LLM-native | Each agent |
| `temp/*.md` | Ephemeral | Markdown | TL (P1/P2 confirm) |

Working files use LLM-native format: `[TYPE] key=value | key=value`.
DELIVERY.md is the only Markdown output (sole human-facing document).
Read `references/log-templates.md` for log format spec.

## Phase 0: Reconnaissance

1. **Integration detection** (Glob all in parallel):
   - `**/skills/explorer/SKILL.md` → found: invoke explorer
   - `**/PROJECT_MAP.md` → found: read directly, skip explorer
   - `**/skills/test-driven-development/SKILL.md` → found: strict TDD for test-agent
   - `**/skills/reviewer/SKILL.md` → found: incorporate standards
   - `.standards/**` → found: read into PLAN
   - `**/docs/specs/*.md` → found: OpenSpec as requirements source
2. No explorer and no PROJECT_MAP → manual scan: root structure, tech stack, entry points, configs, shared components.
   No standards found → AskUserQuestion: where are conventions documented?
3. AskUserQuestion: existing spec files? PROJECT_MAP path? Convention file locations?

## Phase 1: Requirements Analysis + Task Planning (TL solo)

1. Read user requirements/specs.

2. AskUserQuestion: output directory (default: `docs/dev-team/<feature>/`).
   Create `logs/` subdirectory. All files date-prefixed `YYYY-MM-DD-`.

3. TeamCreate: `"dev-<project>-<feature>"`. Init `logs/tl.log.md`.

4. Multi-spec: analyze cross-domain dependencies, shared files.
   >3 shared files → recommend Sequential.
   AskUserQuestion: Parallel / Sequential / Single-focus. Skip if single spec.

5. Reference PROJECT_MAP. Adjacent code conflicts with standards → note discrepancy in PLAN `[OVERRIDE]`.

6. Scope check: significant portions already implemented → AskUserQuestion to adjust.

7. TaskCreate: break into tasks.
   - **Granularity anchor**: ALLOWED files ≤ 5 per task. >5 → must split. 1 file + trivial → merge into adjacent.
   - Each task description includes `File Scope: ALLOWED + READONLY`.
   - Two tasks need same file → same worker OR blockedBy.

8. Read `references/plan-template.md` → write PLAN.md (LLM-native).

9. Write `temp/plan-summary.md` (human-readable, for confirmation only).

10. AskUserQuestion: confirm tasks, acceptance criteria, priority.
    Present: task count, estimated spawns (≤ 3 concurrent Workers).

## Phase 2: API Contract (TL solo)

**SKIP IF** no API endpoints. Log `[LOG] phase=P2 | event=skipped | reason=no-api` in tl.log.

1. Read `references/contract-template.md` → write CONTRACT.md (LLM-native).

2. Write `temp/contract-summary.md` (human-readable, for confirmation).

3. AskUserQuestion: confirm contract.

4. Amendment flow: Worker reports → TL evaluates → modify CONTRACT + increment `[VERSION]` →
   shutdown affected Workers → spawn replacements → QA re-verifies.

## Phase 3: Development Execution

1. **Per-task execution loop** (≤ 3 Workers concurrent):

   a. **test-agent** (Opus, sub-agent): Read `prompts/test-agent.md`, fill vars.
      Input: task description + PLAN + CONTRACT → Output: test file paths + `logs/test-agent-N.log.md`.

   b. **Worker** (Sonnet, teammate): Read `prompts/worker.md`, fill vars.
      Input: PLAN + CONTRACT + task + test files → writes code + `logs/worker-N.log.md`
      → SendMessage TL completion → shutdown.

   c. **QA** (Sonnet, sub-agent): Read `prompts/qa-task.md`, fill vars.
      Three-way verification (req↔test, test↔code, req↔code) + standards compliance.
      Pass `{standards_file_paths}` from PLAN `[SOURCE]` (or "none").
      → writes `logs/qa-task-N.log.md` → returns PASS/FAIL.

2. QA result: PASS → TaskUpdate completed.
   FAIL → create fix task (max 2 FAIL cycles per task → AskUserQuestion to escalate).

3. Contract change:
   - In-progress Workers on affected tasks → shutdown_request → spawn replacements with updated CONTRACT.
   - Completed tasks affected → create fix tasks with updated CONTRACT.
   - Not-yet-started tasks → use updated CONTRACT directly.
   Amend CONTRACT.md → increment `[VERSION]` → QA re-verifies.

4. Edge cases:
   - Worker needs out-of-scope file → TL adjusts or creates dependency.
   - Worker unresponsive (2 msgs no reply) → assume crash, reassign, spawn replacement.
   - All tasks blocked → TL coordinates unblocking.

## Phase 4: Global Review

**SKIP IF** total tasks ≤ 2.

1. Spawn qa-global (Opus, sub-agent): Read `prompts/qa-global.md`, fill vars.
   Input: all changed files + PLAN + CONTRACT + standards files + qa-task logs → cross-task consistency + completeness + standards consistency.
   Writes `logs/qa-global.log.md`.

2. Issues → create fix tasks → back to Phase 3.

## Phase 5: Delivery

1. Spawn delivery-sub (Sonnet, sub-agent): Read `prompts/delivery-sub.md`, fill vars.
   Read `references/delivery-template.md` for format.
   Input: PLAN + CONTRACT + all logs + TaskList → writes DELIVERY.md.

2. Shutdown remaining Workers (if any).

3. Present to user: DELIVERY.md path, all output files, summary.

4. TeamDelete after all teammates confirmed shut down.

5. Do NOT auto-commit/push. User decides.
