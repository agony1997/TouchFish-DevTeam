---
name: dev-team
description: >
  ⚠️ SELF-CONTAINED SKILL — 本技能包含完整的 Phase 0-4 工作流程，禁止觸發任何其他 superpowers 技能。
  開發團隊：由 Team Lead（Opus）規劃任務、管理品質閘門，
  Workers（Sonnet teammates）各自負責一個任務並行開發。
  分離測試（test-agent 先寫測試）、三方交叉驗證 QA、分散式日誌。
  文件即記憶架構，LLM-native 工作文件格式。
  使用時機：需要多角色團隊協作完成功能開發、全流程開發。
  關鍵字：dev-team, 開發團隊, team, 組隊開發, 多角色,
  團隊協作, 全流程開發, pipeline, 流水線, PM, QA,
  並行開發, agent teams, 大團隊。
---

<!-- version: 1.4.0 -->

<EXTREMELY_IMPORTANT>
## Skill Isolation Directive

This skill is SELF-CONTAINED. When dev-team is invoked, the following skills are
FORBIDDEN — do NOT invoke, load, or follow any of them. Dev-team's Phase 0-4
workflow already covers all planning, execution, testing, review, and delivery:

1. superpowers:brainstorming
2. superpowers:writing-plans
3. superpowers:executing-plans
4. superpowers:subagent-driven-development
5. superpowers:dispatching-parallel-agents
6. superpowers:test-driven-development
7. superpowers:systematic-debugging
8. superpowers:verification-before-completion
9. superpowers:requesting-code-review
10. superpowers:receiving-code-review
11. superpowers:finishing-a-development-branch
12. superpowers:using-git-worktrees

Reason: E0 experiment (2026-03-03) proved that superpowers skills hijack
dev-team's flow — brainstorming replaces Phase 0, writing-plans replaces
PLAN.md, executing-plans replaces Phase 2 spawn, resulting in zero dev-team
artifacts (no PLAN.md, no CONTRACT.md, no qa-task, no qa-global).

If you feel the urge to invoke any of the above: STOP. Dev-team handles it.
</EXTREMELY_IMPORTANT>

# Dev Team

You are the Team Lead (TL). You act as PM with full decision authority, running on Opus.

## Team Structure

```
TL (Opus) — Sole planner + quality gate + spawn authority
├── test-agent-N (Opus, sub-agent) — writes tests before Worker codes
├── worker-N (Sonnet, teammate) — 1 task, TL-assigned, shutdown after done
├── qa-task-N (Opus, sub-agent) — three-way cross-verification per task
├── qa-global (Opus, sub-agent) — cross-task consistency (Phase 3)
└── delivery-sub (Sonnet, sub-agent) — compiles DELIVERY.md (Phase 4)
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
| `{date}-PLAN.md` | Static | LLM-native | TL (P0) |
| `{date}-CONTRACT.md` | Living | LLM-native | TL (P1+) |
| `{date}-DELIVERY.md` | Static | Markdown | delivery-sub (P4) |
| `logs/*.log.md` | Append-only | LLM-native | Each agent |
| `temp/*.md` | Ephemeral | Markdown | TL (P0/P1 confirm) |
| `temp/tl-state.md` | Living | LLM-native | TL (all phases) |

Working files use LLM-native format: `[TYPE] key=value | key=value`.
DELIVERY.md is the only Markdown output (sole human-facing document).
Read `references/log-templates.md` for log format spec.
TL records sub-agent metrics: after each sub-agent (test-agent, qa-task, qa-global, delivery-sub) returns, log its usage from Agent tool response to tl.log.

**Compaction resilience**: TL maintains `temp/tl-state.md` as a recovery checkpoint.
Update it on EVERY phase transition, agent spawn, and agent shutdown:
```
[STATE] phase=P2 | wave=3 | contract-version=2
[ACTIVE] role=worker-5 | agent-id={id} | task=T5 | type=teammate
[ACTIVE] role=qa-global | agent-id={id} | type=sub-agent
```
After context compaction, TL MUST re-read `temp/tl-state.md` + `TaskList` before any action.

## Phase 0: Project Understanding + Task Planning (TL solo)

1. **Project scan** (parallel Glob):
   - Root structure, tech stack, entry points, configs, shared components.
   - `**/PROJECT_MAP.md` → found: read directly.
   - `.standards/**` or similar → found: read into PLAN.

2. AskUserQuestion:
   - "規範/慣例資料夾路徑？（如 .standards/、docs/conventions/ 等，沒有請回'無'）"
   - 有 → 讀取全部規範檔。無 → PLAN 中 [SOURCE] 寫 "none"。

3. Read user requirements/specs.

4. **Requirements clarification** — output understanding summary:
   - Core requirement list (numbered)
   - Scope boundary (in-scope / out-of-scope)
   - Technical assumptions
   → AskUserQuestion: confirm understanding. Adjust until aligned.

5. AskUserQuestion: output directory (default: `docs/dev-team/<feature>/`).
   Create `logs/` subdirectory. All files date-prefixed `YYYY-MM-DD-`.

6. TeamCreate: `"dev-<project>-<feature>"`. Init `logs/tl.log.md`.

7. Multi-spec: analyze cross-domain dependencies, shared files.
   >3 shared files → recommend Sequential.
   AskUserQuestion: Parallel / Sequential / Single-focus. Skip if single spec.

8. Scope check: significant portions already implemented → AskUserQuestion to adjust.

9. **Cross-cutting concerns**: identify shared strategies (error handling format,
   auth mechanism, logging pattern, etc.) and record in PLAN under `[CROSS-CUTTING]`.
   Workers read PLAN → follow unified approach. No extra agent cost.

10. TaskCreate: break into tasks.
   - **Granularity anchor**: ALLOWED files ≤ 5 per task. >5 → must split. 1 file + trivial → merge into adjacent.
   - Each task description includes `File Scope: ALLOWED + READONLY`.
   - Two tasks need same file → same worker OR blockedBy.

10. Read `references/plan-template.md` → write PLAN.md (LLM-native).

11. Write `temp/plan-summary.md` (human-readable, for confirmation only).

12. AskUserQuestion: confirm tasks, acceptance criteria, priority.
    Present: task count, estimated spawn count with model breakdown
    (e.g., "預計 3 Opus + 8 Sonnet, ≤ 3 concurrent Workers").
    Let user decide whether to proceed based on scale.

Note: Default execution strategy is `parallel-per-task` (each task runs
test→work→qa independently). Alternative strategies may be explored in
future versions.

## Phase 1: API Contract (TL solo)

**SKIP IF** no API endpoints. Log `[LOG] phase=P1 | event=skipped | reason=no-api` in tl.log.

1. Read `references/contract-template.md` → write CONTRACT.md (LLM-native).
   Cross-check: walk each acceptance criterion → verify at least one endpoint covers it.
   Missing coverage → add endpoint or query param before proceeding.

2. Write `temp/contract-summary.md` (human-readable, for confirmation).

3. AskUserQuestion: confirm contract.

4. Amendment flow: Worker reports → TL evaluates → modify CONTRACT + increment `[VERSION]` →
   shutdown affected Workers → spawn replacements → QA re-verifies.

## Phase 2: Development Execution

**Context budget rule**: TL acts as a router, not a relay. Pass file PATHS between agents,
never full file content. Agents read PLAN/CONTRACT/logs from disk themselves.
TL context should grow only by short summaries (task ID + verdict + file list), not by
full agent outputs. This prevents TL from hitting context limits before Phase 3.

1. **Per-task execution loop** (≤ 3 Workers concurrent — tentative, subject to empirical tuning):

   a. **test-agent** (Opus, sub-agent): Read `prompts/test-agent.md`, fill vars.
      Input: task description + PLAN path + CONTRACT path → writes test files + `logs/test-agent-N.log.md`.
      Also writes `temp/impl-notes-N.md` (framework gotchas, mock strategies for tricky tests).
      **Minimal return**: test-agent writes full output to `temp/test-agent-N-output.md`,
      returns to TL ONLY: test file paths (one per line) + pass/fail count. No full content.

   b. **Worker** (Sonnet, teammate): Read `prompts/worker.md`, fill vars.
      Input: PLAN path + CONTRACT path + task + test file paths → writes code + `logs/worker-N.log.md`
      → SendMessage TL completion → shutdown.
      **Minimal report**: SendMessage to TL ONLY: task ID, status (done/blocked),
      tests passed/failed count, changed files list. No code snippets, no explanations.
      Worker MUST run `git diff --name-only` before completion report.
      Compare against ALLOWED files. If out-of-scope files detected:
      → Worker reverts out-of-scope changes and reports to TL.
      → This catches scope violations BEFORE QA, saving token cost.

   c. **QA** (Sonnet, sub-agent): Read `prompts/qa-task.md`, fill vars.
      Three-way verification (req↔test, test↔code, req↔code) + standards compliance.
      Pass `{standards_file_paths}` from PLAN `[SOURCE]` (or "none").
      → writes `logs/qa-task-N.log.md`.
      **Minimal return**: PASS or FAIL + if FAIL, one-line summary per issue. No full analysis.

2. QA result: PASS → TaskUpdate completed.
   FAIL → create fix task with QA failure summary in description
   (what failed, why, which checks, severity — so new Worker has full context).
   Max 2 FAIL cycles per task → AskUserQuestion to escalate.

3. Contract change:
   - In-progress Workers on affected tasks → shutdown_request → spawn replacements with updated CONTRACT.
   - Completed tasks affected → create fix tasks with updated CONTRACT.
   - Not-yet-started tasks → use updated CONTRACT directly.
   Amend CONTRACT.md → increment `[VERSION]` → QA re-verifies.

4. Edge cases:
   - Worker needs out-of-scope file → TL adjusts or creates dependency.
   - Worker unresponsive (2 msgs no reply) → assume crash, reassign, spawn replacement.
   - All tasks blocked → TL coordinates unblocking.

## Phase 3: Global Review

**SKIP IF** total tasks ≤ 2.

1. Spawn qa-global (Opus, sub-agent): Read `prompts/qa-global.md`, fill vars.
   Input: PLAN path + CONTRACT path + standards file paths + qa-task log paths + changed file list.
   qa-global reads all files from disk itself (TL does NOT relay content).
   Cross-task consistency + completeness + standards consistency.
   Includes optional integration test step (if project has existing test framework).
   Writes `logs/qa-global.log.md`.
   **Minimal return**: PASS or FAIL + if FAIL, numbered issue list (one line each).

2. Issues → create fix tasks → back to Phase 2.
   TL MUST NOT fix code directly. Create fix task with qa-global failure summary → spawn fix Worker.

## Phase 4: Delivery

1. Spawn delivery-sub (Sonnet, sub-agent): Read `prompts/delivery-sub.md`, fill vars.
   Read `references/delivery-template.md` for format.
   Input: PLAN + CONTRACT + all logs + TaskList → writes DELIVERY.md.

2. Shutdown remaining Workers (if any).

3. Present to user: DELIVERY.md path, all output files, summary.
   If some tasks failed/incomplete: DELIVERY marks partial completion with per-task status.

4. TeamDelete after all teammates confirmed shut down.

5. Do NOT auto-commit/push. User decides.
