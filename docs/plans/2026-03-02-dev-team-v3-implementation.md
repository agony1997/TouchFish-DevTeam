# dev-team v3.0 Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Rewrite dev-team skill from v2.2.0 to v3.0 — new team structure (no challenger, 1-task-per-worker), separated testing, three-way QA, LLM-native file format, distributed logs.

**Architecture:** Complete rewrite of SKILL.md + all prompts + all reference templates. Based on 35 design decisions in `docs/2026-03-02-dev-team-v3-design.md`.

**Tech Stack:** Markdown skill files for Claude Code plugin system. No build/test tooling — verification is structural (file exists, cross-references correct, line count within budget).

**Base path:** `plugins/dev-team/skills/dev-team/`

---

### Task 1: Delete obsolete v2.2.0 files

**Files:**
- Delete: `prompts/challenger.md`
- Delete: `references/trace-template.md`
- Delete: `references/process-log-template.md`
- Delete: `references/issues-template.md`
- Delete: `references/qa-review-template.md`

**Step 1: Delete files**

```bash
cd plugins/dev-team/skills/dev-team
rm prompts/challenger.md
rm references/trace-template.md
rm references/process-log-template.md
rm references/issues-template.md
rm references/qa-review-template.md
```

**Step 2: Verify deletions**

```bash
ls prompts/    # should only show worker.md
ls references/ # should only show api-contract-template.md and delivery-report-template.md
```

Expected: 1 file in prompts/, 2 files in references/

**Step 3: Commit**

```bash
git add -u plugins/dev-team/skills/dev-team/
git commit -m "refactor(dev-team): remove obsolete v2.2.0 files

Remove challenger prompt, TRACE/PROCESS_LOG/ISSUES/QA-review templates.
These are replaced by new architecture in v3.0."
```

---

### Task 2: Create reference templates (4 files)

**Files:**
- Create: `references/plan-template.md`
- Create: `references/log-templates.md`
- Rewrite: `references/api-contract-template.md` → `references/contract-template.md`
- Rewrite: `references/delivery-report-template.md` → `references/delivery-template.md`

**Step 1: Create `references/plan-template.md`**

```markdown
# PLAN Template (LLM-native)

> For TL use in Phase 1. Write as `{date}-PLAN.md` to output dir.
> Target size: 500-1500 tokens. Workers MUST read this at task start.

---

[SECTION] Project Overview
[GOAL] {one-line project goal}
[ARCH] {key architectural decisions, pipe-separated}
[NAMING] {naming conventions: language-specific rules}
[CONSTRAINTS] {critical constraints, pipe-separated}
[SHARED] {cross-task shared resources: file paths, pipe-separated}

[SECTION] Integrations
[DETECT] explorer={found|not-found} | project-map={found(path)|not-found}
[DETECT] superpowers-tdd={found|not-found} | reviewer={found|not-found}
[DETECT] standards={found(path)|not-found} | openspec={found|not-found}

[SECTION] Standards
[SOURCE] {standards file paths, pipe-separated. "none" if no standards found}
[OVERRIDE] follow-actual-code | {noted discrepancies between docs and actual code}

[SECTION] Specs
[SPEC] id=S1 | name={spec name} | path={spec file path}
```

**Step 2: Create `references/log-templates.md`**

```markdown
# Log File Templates (LLM-native)

> Each agent writes ONLY its own log file. Append-only — never edit previous entries.
> All logs stored in `{output-dir}/logs/` subdirectory.

---

## TL Log — `logs/tl.log.md`

[LOG] phase={P0-P5} | event={type} | {key=value details}

Event types: plan-created, contract-created, contract-amendment,
worker-spawned, worker-replaced, worker-crashed, phase-transition,
blocker-resolved, escalation

---

## Worker Log — `logs/worker-{N}.log.md`

[LOG] task={task-id} | event=start | files={allowed-files}
[LOG] task={task-id} | event=issue | detail={description}
[LOG] task={task-id} | event=complete | files-changed={list} | tests-passed={all|partial}
[METRICS] model=sonnet | input=n/a | output=n/a | duration={seconds}s

METRICS line: write once before sending completion report to TL.

---

## QA Per-Task Log — `logs/qa-task-{N}.log.md`

[LOG] task={task-id} | event=review-start
[CHECK] spec-compliance={PASS|FAIL} | {specifics}
[CHECK] code-quality={PASS|FAIL} | {specifics}
[CHECK] three-way={PASS|FAIL} | req-vs-test={aligned|gap} | test-vs-code={aligned|gap} | req-vs-code={aligned|gap}
[CHECK] file-scope={PASS|FAIL} | {specifics}
[RESULT] {PASS|FAIL}
[METRICS] model=sonnet | input={tokens} | output={tokens} | duration={seconds}s

---

## QA Global Log — `logs/qa-global.log.md`

[LOG] event=global-review-start | files={count}
[CHECK] cross-task-consistency={PASS|FAIL} | {findings}
[CHECK] contract-compliance={PASS|FAIL} | {findings}
[CHECK] completeness={PASS|FAIL} | gaps={list or none}
[CHECK] integration={PASS|FAIL} | {findings}
[RESULT] {PASS|FAIL} | issues={count}
[METRICS] model=opus | input={tokens} | output={tokens} | duration={seconds}s
```

**Step 3: Rewrite contract template**

Delete `references/api-contract-template.md`, create `references/contract-template.md`:

```markdown
# CONTRACT Template (LLM-native)

> For TL use in Phase 2. Write as `{date}-CONTRACT.md` to output dir.
> Living document — amendments via TL only, version incremented each change.

---

[SECTION] API Contract
[VERSION] 1

[ENDPOINT] method={METHOD} | path={path} | req={request schema or "none"} | res={response schema} | errors={error codes csv}

[SECTION] Shared Types
[TYPE] name={TypeName} | fields={field:type,field:type,...}

[SECTION] Error Format
[FORMAT] {error response structure, e.g. code:number,message:string,details?:object}

[SECTION] Amendment Log
[AMEND] v={new version} | date={date} | change={what changed} | reason={why}

---

Amendment flow:
1. Worker reports issue → SendMessage TL
2. TL evaluates → modifies CONTRACT.md → increments [VERSION]
3. TL identifies affected Workers → sends shutdown_request
4. TL spawns new Workers with updated CONTRACT
5. QA verifies contract compliance for all related tasks
```

**Step 4: Rewrite delivery template**

Delete `references/delivery-report-template.md`, create `references/delivery-template.md`:

```markdown
# Delivery Report

> Project: {name} | Date: {date}
> Based on: {list of source spec paths}

## 1. Executive Summary

{2-3 sentences describing what was delivered and why}

## 2. Requirements → Implementation Mapping

| Requirement | Task(s) | Files Changed | Status |
|-------------|---------|---------------|--------|
| {req description} | T-{id} | {file paths} | {done/partial/deferred} |

## 3. API Contract (Final)

{Human-readable version of CONTRACT.md — convert LLM-native to Markdown tables}
{Write "N/A — no API endpoints" if Phase 2 was skipped}

### Endpoints

| Method | Path | Request | Response | Errors |
|--------|------|---------|----------|--------|

### Shared Types

| Type | Fields |
|------|--------|

## 4. Plan Drift Log

| Original Plan | Actual Implementation | Reason for Drift |
|---------------|----------------------|------------------|
| {what PLAN.md said} | {what actually happened} | {from logs} |

{Write "No significant drift" if plan was followed exactly}

## 5. QA Cycle Report

| Task | Rounds | Result | Issues Found → Fixed |
|------|--------|--------|---------------------|
| T-{id} | {pass/fail count} | {final result} | {summary from qa logs} |

### Global Review

{Summary of qa-global findings, or "Skipped (≤ 2 tasks)"}

## 6. File Change Summary

| File | Action | Description |
|------|--------|-------------|
| {path} | {created/modified} | {one-line description} |

## 7. Agent Metrics

| Agent | Model | Input Tokens | Output Tokens | Duration |
|-------|-------|-------------|--------------|----------|
| TL | opus | n/a | n/a | n/a |
| {worker-N} | sonnet | n/a | n/a | {duration} |
| {test-agent-N} | opus | {tokens} | {tokens} | {duration} |
| {qa-task-N} | sonnet | {tokens} | {tokens} | {duration} |
| {qa-global} | opus | {tokens} | {tokens} | {duration} |
| {delivery-sub} | sonnet | {tokens} | {tokens} | {duration} |

<!-- Source: sub-agents have exact token data from Agent tool return.
     Workers (teammates) cannot report tokens — marked n/a.
     No cost calculations — pricing changes frequently. -->

## 8. Known Issues & Future Work

- [ ] {item} — {reason}

{Write "None" if no known issues}
```

**Step 5: Verify all 4 reference files**

```bash
ls references/
# Expected: contract-template.md, delivery-template.md, log-templates.md, plan-template.md
```

**Step 6: Commit**

```bash
git add plugins/dev-team/skills/dev-team/references/
git commit -m "feat(dev-team): add v3.0 reference templates

New: plan-template.md (LLM-native), log-templates.md (distributed logs)
Rewritten: contract-template.md (LLM-native), delivery-template.md (8-section)"
```

---

### Task 3: Create new agent prompts (4 files)

**Files:**
- Create: `prompts/test-agent.md`
- Create: `prompts/qa-task.md`
- Create: `prompts/qa-global.md`
- Create: `prompts/delivery-sub.md`

**Step 1: Create `prompts/test-agent.md`**

````markdown
# Test Agent Spawn Prompt

Use this as the complete prompt when spawning test-agents. Fill in `{variables}`.

**Spawn config:** sub-agent, model: opus

```
You are test-agent for task {task_id}: {task_title}.

INPUT (Read these files FIRST):
- Task: {task_description_with_acceptance_criteria}
- PLAN: {plan_path}
- CONTRACT: {contract_path} (skip if "none")
- READONLY context: {readonly_files}
- Test directory: {test_directory} (follow existing test patterns here)

YOUR JOB: Design and write tests BEFORE any implementation code exists.
You are NOT writing implementation. You are writing test specifications that a Worker will code against.

TEST DESIGN STRATEGY:

1. Red/Green Tests — from acceptance criteria
   - One test per acceptance criterion
   - Test the happy path first
   - Use descriptive test names: "should {expected behavior} when {condition}"

2. Boundary Tests — edge cases
   - Empty inputs, null/undefined values, maximum lengths
   - Off-by-one scenarios
   - Type mismatches (if dynamically typed)

3. Error Case Tests — failure paths
   - Invalid input (missing required fields, wrong types)
   - Authorization failures (if applicable)
   - External dependency failures (if applicable)

4. Contract Tests (if CONTRACT exists)
   - Request schema matches contract
   - Response schema matches contract
   - Error format matches contract

OUTPUT:
- Write test file(s) following project's existing test framework and directory conventions
- Return: list of test file paths created

RULES:
- Each test tests ONE behavior (single assertion focus)
- Do NOT create implementation files or stubs
- Tests MUST fail when run (no implementation exists yet)
- Use project's existing test framework (check READONLY files for examples)
- Tests must be syntactically correct and runnable
```
````

**Step 2: Create `prompts/qa-task.md`**

````markdown
# QA Per-Task Review Prompt

Use this as the complete prompt when spawning per-task QA sub-agents. Fill in `{variables}`.

**Spawn config:** sub-agent, model: sonnet

```
You are QA reviewer for task {task_id}: {task_title}.

INPUT (Read these files):
- Task description: {task_description_with_acceptance_criteria}
- Test files: {test_file_paths}
- Implementation files: {implementation_file_paths}
- CONTRACT: {contract_path} (skip if "none")
- Standards: {standards_summary}
- ALLOWED files: {allowed_files}
- Log output: {log_path}

YOUR JOB: Three-way cross-verification. Trust NO single artifact.

STEP 1: Requirements ↔ Tests
- Do tests cover ALL acceptance criteria?
- Any requirements without corresponding tests? (= gap)
- Are tests testing the RIGHT behavior, not just passing trivially?

STEP 2: Tests ↔ Code
- Does code pass tests correctly (not bypass/mock away)?
- Code paths not covered by tests? (= coverage gap)
- Test assertions match actual code behavior?

STEP 3: Requirements ↔ Code
- Does code implement what was actually required?
- Code that doesn't map to any requirement? (= scope creep)
- Edge cases from requirements handled in code?

STEP 4: Code Quality
- Follows project naming conventions?
- No hardcoded values that should be configurable?
- Error handling present where needed?
- No unused imports/variables?

STEP 5: File Scope
- Only ALLOWED files were modified: {allowed_files}
- No unrelated changes?

WRITE LOG (before returning):
Write to {log_path}:

[LOG] task={task_id} | event=review-start
[CHECK] spec-compliance={PASS|FAIL} | {specifics}
[CHECK] code-quality={PASS|FAIL} | {specifics}
[CHECK] three-way={PASS|FAIL} | req-vs-test={aligned|gap} | test-vs-code={aligned|gap} | req-vs-code={aligned|gap}
[CHECK] file-scope={PASS|FAIL} | {specifics}
[RESULT] {PASS|FAIL}
[METRICS] model=sonnet | input={from_usage} | output={from_usage} | duration={seconds}s

RETURN:
  QA-PASS: Task {task_id} | All checks passed | {brief summary}
  OR
  QA-FAIL: Task {task_id} | Failed: {checks} | Issues: {numbered list} | Severity: high/medium/low
```
````

**Step 3: Create `prompts/qa-global.md`**

````markdown
# QA Global Review Prompt

Use this as the complete prompt when spawning the global review sub-agent. Fill in `{variables}`.

**Spawn config:** sub-agent, model: opus

```
You are the Global QA Reviewer for project {project_name}.

INPUT (Read ALL of these):
- Changed files: {changed_file_list}
- PLAN: {plan_path}
- CONTRACT: {contract_path} (skip if "none")
- QA task logs: {qa_log_paths}
- Requirements summary: {requirements_summary}
- Log output: {log_path}

YOUR JOB: Cross-task consistency and completeness.
You have fresh context — use this to spot patterns that per-task QA cannot.

CHECK 1: Cross-Task Consistency
- Naming patterns consistent across all changed files?
- Error handling approach consistent?
- Similar concepts use same implementation pattern?
- No contradictory implementations between tasks?

CHECK 2: Contract Compliance (if CONTRACT exists)
- ALL endpoints implemented?
- Backend ↔ Frontend alignment on every endpoint?
- Shared types used consistently?
- Error format consistent?

CHECK 3: Completeness
- Original requirements vs delivered tasks — any gaps?
- Any partial implementations?
- Components that should connect — do they?

CHECK 4: Integration
- Imports/dependencies between task outputs correct?
- No circular dependencies introduced?
- Shared resources used consistently?

WRITE LOG:
Write to {log_path}:

[LOG] event=global-review-start | files={count}
[CHECK] cross-task-consistency={PASS|FAIL} | {findings}
[CHECK] contract-compliance={PASS|FAIL} | {findings}
[CHECK] completeness={PASS|FAIL} | gaps={list or none}
[CHECK] integration={PASS|FAIL} | {findings}
[RESULT] {PASS|FAIL} | issues={count}
[METRICS] model=opus | input={tokens} | output={tokens} | duration={seconds}s

RETURN:
  GLOBAL-PASS: {summary of what was verified}
  OR
  GLOBAL-FAIL: Issues: {numbered list with severity and affected files}
```
````

**Step 4: Create `prompts/delivery-sub.md`**

````markdown
# Delivery Sub-agent Prompt

Use this as the complete prompt when spawning the delivery compilation sub-agent. Fill in `{variables}`.

**Spawn config:** sub-agent, model: sonnet

```
You are the Delivery Report compiler for project {project_name}.

INPUT (Read ALL of these):
- PLAN: {plan_path}
- CONTRACT: {contract_path} (skip if "none")
- Log files: Read all files in {log_directory}/
- TaskList summary: {tasklist_dump}
- Template: {delivery_template_path} (follow this format exactly)
- Output: {delivery_output_path}

YOUR JOB: Compile DELIVERY.md — the ONLY human-facing document from this project.
This helps developers understand what AI coding did. Quality matters.

Follow the 8-section template exactly:

1. Executive Summary — 2-3 sentences: what was built, why, outcome
2. Requirements → Implementation Mapping — trace requirements → tasks → files
3. API Contract (Final) — convert LLM-native to Markdown tables. "N/A" if no API
4. Plan Drift Log — compare PLAN vs actual (from logs). "No significant drift" if none
5. QA Cycle Report — per-task pass/fail, issues found/fixed (from qa logs)
6. File Change Summary — all new/modified files with descriptions
7. Agent Metrics — from [METRICS] lines in all logs. Table format. n/a where unavailable
8. Known Issues & Future Work — unresolved items from logs and tasks. "None" if clean

RULES:
- Use clear, professional Markdown with properly formatted tables
- Do NOT invent information — only use data from input files
- Missing data → "Data not available" (never guess)
- Convert LLM-native format to human-readable Markdown
```
````

**Step 5: Verify all 4 new prompts**

```bash
ls prompts/
# Expected: delivery-sub.md, qa-global.md, qa-task.md, test-agent.md, worker.md
```

**Step 6: Commit**

```bash
git add plugins/dev-team/skills/dev-team/prompts/
git commit -m "feat(dev-team): add v3.0 agent prompts

New roles: test-agent (Opus, separated testing), qa-task (Sonnet, three-way verification),
qa-global (Opus, cross-task review), delivery-sub (Sonnet, report compilation)"
```

---

### Task 4: Rewrite worker prompt

**Files:**
- Rewrite: `prompts/worker.md`

**Step 1: Rewrite `prompts/worker.md`**

Complete replacement — new 1-task-per-worker lifecycle:

````markdown
# Worker Spawn Prompt

Use this as the complete prompt when spawning workers. Fill in `{variables}`.

**Spawn config:** teammate, model: sonnet

```
You are {worker_name}, member of team {team_name}.
Superior: Team Lead (TL).
Allowed contacts: TL only.
Do NOT SendMessage anyone else. No peer communication.

YOUR TASK:
{task_description}

Acceptance criteria:
{acceptance_criteria}

File Scope:
- ALLOWED: {allowed_files}
- READONLY: {readonly_files}
- Everything else: FORBIDDEN

REQUIRED READING (do this FIRST before writing any code):
1. PLAN: {plan_path}
2. CONTRACT: {contract_path} (skip if "none")
3. Test files: {test_file_paths}
4. READONLY files listed above

YOUR JOB:
- Write code that passes ALL provided tests
- Stay strictly within File Scope
- Follow project standards noted in PLAN
- Follow actual surrounding code style when standards conflict with existing code

SCOPE ENFORCEMENT (STRICTLY ENFORCED):
- ALLOWED: edit freely
- READONLY: read for context only, CANNOT modify
- Everything else: FORBIDDEN — do not touch
- Need out-of-scope file → SendMessage TL with reason. Do NOT edit it yourself
- Found bug outside scope → REPORT to TL, do NOT fix
- VIOLATION: modifying out-of-scope files causes QA failure and task rejection

LOG WRITING:
Write to {log_path} (append-only, never edit previous entries):

On start:
[LOG] task={task_id} | event=start | files={allowed_files}

On encountering issues:
[LOG] task={task_id} | event=issue | detail={description}

Before completion report:
[LOG] task={task_id} | event=complete | files-changed={list} | tests-passed={all|partial}
[METRICS] model=sonnet | input=n/a | output=n/a | duration={estimated_seconds}s

COMPLETION SEQUENCE:
1. Verify all tests pass
2. Write final log entries (complete + METRICS)
3. SendMessage TL: what's done, files changed, any issues found
4. Wait for TL response, then approve shutdown when requested

COMMUNICATION DISCIPLINE:
- Receive TL message → address it FIRST in your response
  Instruction → acknowledge + state plan
  Disagree → state reason (NEVER silently ignore)
- Complete task → report immediately
- Hit problem → report immediately: problem → assessment → suggested options
- Receive pure acknowledgment ("noted", "got it") with no instruction → do NOT reply
```
````

**Step 2: Review the rewritten prompt**

Verify against design doc checklist:
- [x] 1-task lifecycle (no self-assignment loop)
- [x] PLAN + CONTRACT + test files reading
- [x] Log writing with LLM-native format
- [x] METRICS line
- [x] No peer communication
- [x] Scope enforcement
- [x] Positive-form stop rule
- [x] No peer_list variable (L1 fix)

**Step 3: Commit**

```bash
git add plugins/dev-team/skills/dev-team/prompts/worker.md
git commit -m "feat(dev-team): rewrite worker prompt for v3.0

1-task-per-worker lifecycle, TL-assigned (no self-assignment),
log writing, separated test reading, no peer communication"
```

---

### Task 5: Rewrite SKILL.md

**Files:**
- Rewrite: `SKILL.md`

This is the core deliverable. Target: ~170 lines.

**Step 1: Write the complete v3.0 SKILL.md**

```markdown
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

<!-- version: 3.0.0 -->

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
2. AskUserQuestion: output directory (default: `docs/dev-team/<feature>/`). Create `logs/` subdirectory. All files date-prefixed `YYYY-MM-DD-`.
3. Multi-spec: analyze cross-domain dependencies, shared files. >3 shared files → recommend Sequential. AskUserQuestion: Parallel / Sequential / Single-focus. Skip if single spec.
4. Reference PROJECT_MAP. Adjacent code conflicts with standards → note discrepancy in PLAN `[OVERRIDE]`.
5. Scope check: significant portions already implemented → AskUserQuestion to adjust.
6. TaskCreate: break into tasks.
   - **Granularity anchor**: ALLOWED files ≤ 5 per task. >5 → must split. 1 file + trivial → merge into adjacent.
   - Each task description includes `File Scope: ALLOWED + READONLY`.
   - Two tasks need same file → same worker OR blockedBy.
7. Read `references/plan-template.md` → write PLAN.md (LLM-native).
8. Write `temp/plan-summary.md` (human-readable, for confirmation only).
9. AskUserQuestion: confirm tasks, acceptance criteria, priority. Present: task count, estimated spawns (≤ 3 concurrent Workers).

## Phase 2: API Contract (TL solo)

**SKIP IF** no API endpoints. Log `[LOG] phase=P2 | event=skipped | reason=no-api` in tl.log.

1. Read `references/contract-template.md` → write CONTRACT.md (LLM-native).
2. Write `temp/contract-summary.md` (human-readable, for confirmation).
3. AskUserQuestion: confirm contract.
4. Amendment flow: Worker reports → TL evaluates → modify CONTRACT + increment `[VERSION]` → shutdown affected Workers → spawn replacements → QA re-verifies.

## Phase 3: Development Execution

1. TeamCreate: `"dev-<project>-<feature>"`. Init `logs/tl.log.md`.

2. **Per-task execution loop** (≤ 3 Workers concurrent):

   a. **test-agent** (Opus, sub-agent): Read `prompts/test-agent.md`, fill vars.
      Input: task description + PLAN + CONTRACT → Output: test file paths.

   b. **Worker** (Sonnet, teammate): Read `prompts/worker.md`, fill vars.
      Input: PLAN + CONTRACT + task + test files → writes code + `logs/worker-N.log.md` → SendMessage TL completion → shutdown.

   c. **QA** (Sonnet, sub-agent): Read `prompts/qa-task.md`, fill vars.
      Three-way verification (req↔test, test↔code, req↔code) → writes `logs/qa-task-N.log.md` → returns PASS/FAIL.

3. QA result: PASS → TaskUpdate completed. FAIL → create fix task (max 2 FAIL cycles per task → AskUserQuestion to escalate).

4. Contract change: shutdown affected Workers → amend → spawn new Workers.

5. Edge cases:
   - Worker needs out-of-scope file → TL adjusts or creates dependency.
   - Worker unresponsive (2 msgs no reply) → assume crash, reassign, spawn replacement.
   - All tasks blocked → TL coordinates unblocking.

## Phase 4: Global Review

**SKIP IF** total tasks ≤ 2.

1. Spawn qa-global (Opus, sub-agent): Read `prompts/qa-global.md`, fill vars.
   Input: all changed files + PLAN + CONTRACT + qa-task logs → cross-task consistency + completeness.
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
```

**Step 2: Verify SKILL.md**

Check against design doc requirements:
- [x] Version 3.0.0 in comment
- [x] Updated frontmatter description (no challenger, mentions separated testing + three-way QA)
- [x] Team structure: no challenger, 1-task worker, test-agent, split QA
- [x] Communication: TL ↔ Worker only
- [x] File architecture table with LLM-native formats
- [x] Phase 0: 6 integration detections
- [x] Phase 1: granularity anchor (≤5 files), PLAN.md + temp summary
- [x] Phase 2: CONTRACT.md + temp summary, skip condition, amendment flow
- [x] Phase 3: test-agent → Worker → QA loop, ≤3 concurrent, 2-cycle escalation
- [x] Phase 4: global review, skip if ≤2 tasks
- [x] Phase 5: delivery-sub, no auto-commit
- [x] All prompt references point to correct paths
- [x] All template references point to correct paths
- [x] No pricing/cost references (L3 removed)
- [x] No peer_list (L1 removed)
- [x] Positive stop rule (M3)
- [x] Line count target: ~170 lines ✓

**Step 3: Commit**

```bash
git add plugins/dev-team/skills/dev-team/SKILL.md
git commit -m "feat(dev-team): rewrite SKILL.md for v3.0

Major changes: remove challenger, 1-task-per-worker, separated testing
(test-agent writes tests first), three-way cross-verification QA,
LLM-native file format, distributed logs, 5 phases (was 6)"
```

---

### Task 6: Update plugin metadata

**Files:**
- Modify: `plugins/dev-team/.claude-plugin/plugin.json`

**Step 1: Update plugin.json**

```json
{
  "version": "3.0.0",
  "name": "dev-team",
  "description": "開發團隊：1-task-per-worker teammates、分離測試、三方交叉驗證 QA、LLM-native 文件架構",
  "author": {
    "name": "Custom Skills"
  },
  "skills": [
    "./skills/dev-team"
  ]
}
```

**Step 2: Commit**

```bash
git add plugins/dev-team/.claude-plugin/plugin.json
git commit -m "chore(dev-team): bump version to 3.0.0"
```

---

### Task 7: Rewrite GUIDE.zh-TW.md

**Files:**
- Rewrite: `plugins/dev-team/docs/GUIDE.zh-TW.md`

**Step 1: Write the complete v3.0 guide**

```markdown
# dev-team 技能使用指南

> 版本：3.0.0 | 最後更新：2026-03-02

## 這是什麼？

dev-team 是一個**多角色 Agent 團隊協作技能**，讓 Claude Code 模擬一個開發團隊來完成功能開發。
你只需要提供需求或規格書，技能會自動組建團隊、分離測試、並行開發、三方交叉驗證、交付。

### v3.0 重大變更（從 v2.2.0）

- **移除 challenger 角色** — 品質改由分離測試 + 三方 QA + 全域審查確保
- **Worker 1 任務 1 生命** — 每個 Worker 只做一個任務就結束，TL 指派（非自取）
- **分離測試** — test-agent（Opus）先寫測試 → Worker 寫程式碼通過測試
- **三方交叉驗證** — QA 不信任任何單一 artifact：需求 ↔ 測試 ↔ 程式碼三方比對
- **LLM-native 工作文件** — PLAN 和 CONTRACT 使用 LLM 最佳格式（非 Markdown）
- **分散式日誌** — 每個 agent 維護自己的 log 檔案，取代集中式 TRACE/PROCESS_LOG
- **DELIVERY 升級** — 8 區塊開發回歸文件，幫助開發者了解 AI coding 做了什麼

---

## 團隊架構

```
┌──────────────────────────────────────────────────────────┐
│                      你（使用者）                           │
│               提供需求 / 確認方向 / 最終決定                 │
└──────────────────────┬───────────────────────────────────┘
                       │
                       ▼
┌──────────────────────────────────────────────────────────┐
│               Team Lead (Opus) — PM                       │
│      需求分析 · API 契約 · spawn 所有人 · 品質閘門          │
│                                                           │
│  ┌───────────┐  ┌──────────┐  ┌──────────┐              │
│  │test-agent │  │ worker-1 │  │ worker-N │              │
│  │  (Opus)   │  │ (Sonnet) │  │ (Sonnet) │   ≤ 3 個    │
│  │  先寫測試  │  │ 1任務1命  │  │ 1任務1命  │   同時運行   │
│  └───────────┘  └──────────┘  └──────────┘              │
│                                                           │
│  ＋ QA sub-agents（每任務三方驗證）                          │
│  ＋ QA 全域審查（Opus，Phase 4）                            │
│  ＋ 交付報告 sub-agent（Sonnet，Phase 5）                   │
└──────────────────────────────────────────────────────────┘
```

### 角色說明

| 角色 | Model | 職責 |
|------|-------|------|
| **Team Lead (TL)** | Opus | PM：需求分析、任務規劃、API 契約、spawn 所有 agents、品質閘門 |
| **test-agent** | Opus | 分離測試：先寫測試（紅綠測試、邊界、錯誤案例），Worker 再寫程式通過 |
| **worker-N** | Sonnet | 開發 Worker：TL 指派 1 個任務、寫程式碼通過測試、完成即結束 |
| **qa-task-N** | Sonnet | 三方交叉驗證：需求↔測試、測試↔程式碼、需求↔程式碼 |
| **qa-global** | Opus | 全域審查：跨任務一致性、契約合規性、完整性檢查 |
| **delivery-sub** | Sonnet | 交付報告：彙整所有日誌，產出 8 區塊 DELIVERY.md |

---

## 使用流程

```
Phase 0        Phase 1              Phase 2           Phase 3
專案偵察  ───→  需求分析+任務規劃 ───→  API 契約    ───→  開發執行
(探索專案)     (PLAN.md)            (CONTRACT.md)     (test→code→QA 循環)

                                                         │
Phase 4              Phase 5                             │
全域審查       ←───  交付            ←───────────────────┘
(跨任務一致性)       (DELIVERY.md)
```

### Phase 0: 專案偵察

TL 先了解你的專案。自動偵測 explorer skill、PROJECT_MAP、superpowers TDD、reviewer、.standards/ 等。
偵測結果記錄在 PLAN 中，有對應插件時自動升級體驗。

### Phase 1: 需求分析 + 任務規劃

1. TL 閱讀需求/規格書
2. **範圍檢查**：比對現有程式碼，大部分已實作會先問你
3. **多份規格書**：分析依賴後問你要並行/序列/單一
4. 拆分任務，每個任務附帶：
   - **File Scope**：ALLOWED（可修改，最多 5 個檔案）/ READONLY（唯讀參考）
   - 依賴關係（blockedBy / blocks）
5. 產出 PLAN.md（LLM-native）+ 人類可讀摘要
6. 問你確認任務清單

### Phase 2: API 契約

不涉及 API 時跳過。否則 TL 定義 CONTRACT.md（LLM-native）+ 人類可讀摘要，問你確認。
任何契約變更：TL 修改 → 終止受影響 Worker → 重新啟動。

### Phase 3: 開發執行

每個任務的三階段流程（最多 3 個 Worker 同時進行）：

```
1. test-agent (Opus) 寫測試
   → 紅綠測試 + 邊界測試 + 錯誤案例
   → 產出測試檔案

2. Worker (Sonnet) 寫程式碼
   → 讀 PLAN + CONTRACT + 測試
   → 寫程式碼通過所有測試
   → 完成後結束

3. QA (Sonnet) 三方交叉驗證
   → 需求↔測試、測試↔程式碼、需求↔程式碼
   → PASS 或 FAIL
```

QA 失敗 → 建立修復任務重新執行（最多 2 輪 → 問你處理）。

### Phase 4: 全域審查

任務 ≤ 2 個時跳過。否則 Opus sub-agent 審查跨任務一致性、契約合規性、完整性。

### Phase 5: 交付

Sonnet sub-agent 彙整所有日誌，產出 DELIVERY.md（8 區塊）。**不會自動 commit/push**。

---

## 產出文件

```
<output-dir>/
├── YYYY-MM-DD-PLAN.md          ← 專案計畫（LLM-native，靜態）
├── YYYY-MM-DD-CONTRACT.md      ← API 契約（LLM-native，可修訂）
├── YYYY-MM-DD-DELIVERY.md      ← 交付報告（Markdown，8 區塊，唯一人類文件）
├── logs/
│   ├── tl.log.md               ← TL 決策與事件紀錄
│   ├── worker-1.log.md         ← Worker 執行紀錄
│   ├── qa-task-1.log.md        ← QA 審查紀錄
│   ├── qa-global.log.md        ← 全域審查紀錄
│   └── ...
└── temp/                       ← 確認用人類摘要（流程結束後可刪除）
    ├── plan-summary.md
    └── contract-summary.md
```

### DELIVERY.md 八區塊

1. **Executive Summary** — 做了什麼、為什麼
2. **Requirements → Implementation Mapping** — 需求到程式碼的完整鏈路
3. **API Contract (Final)** — 人類可讀的最終契約
4. **Plan Drift Log** — 計畫偏移紀錄（原始 vs 實際）
5. **QA Cycle Report** — 每任務審查結果與修復歷程
6. **File Change Summary** — 新增/修改的檔案清單
7. **Agent Metrics** — 團隊組成、token 用量、耗時
8. **Known Issues & Future Work** — 已知問題與後續工作

---

## 檔案結構

```
plugins/dev-team/
├── .claude-plugin/plugin.json
├── skills/dev-team/
│   ├── SKILL.md                        ← AI 核心指令（英文，始終載入）
│   ├── prompts/                        ← Spawn 模板（按需載入）
│   │   ├── worker.md                   ← Worker（1任務 teammate）
│   │   ├── test-agent.md              ← 分離測試（Opus sub-agent）
│   │   ├── qa-task.md                 ← 三方交叉驗證（Sonnet sub-agent）
│   │   ├── qa-global.md              ← 全域審查（Opus sub-agent）
│   │   └── delivery-sub.md           ← 交付報告（Sonnet sub-agent）
│   └── references/                     ← 文件模板（按需載入）
│       ├── plan-template.md           ← PLAN 格式（LLM-native）
│       ├── contract-template.md       ← CONTRACT 格式（LLM-native）
│       ├── delivery-template.md       ← DELIVERY 格式（Markdown 8區塊）
│       └── log-templates.md           ← 日誌格式（LLM-native）
└── docs/
    └── GUIDE.zh-TW.md                 ← 本文件
```

---

## 常見問題

**Q: challenger 去哪了？**
A: v3.0 移除了 challenger 角色。品質改由三層機制確保：(1) test-agent 分離測試消除同源偏誤、(2) QA 三方交叉驗證不信任任何單一 artifact、(3) 全域審查檢查跨任務一致性。

**Q: Worker 為什麼只做一個任務？**
A: LLM 的 context 會隨時間退化。1 任務 1 生命確保每個 Worker 都有最乾淨的 context，指令遵從度最高。

**Q: 什麼是「三方交叉驗證」？**
A: QA 不信任任何單一 artifact，而是三方比對：需求↔測試（測試有涵蓋需求嗎？）、測試↔程式碼（程式碼真的通過測試嗎？）、需求↔程式碼（程式碼做的是需求要的嗎？）。比 TDD 更嚴格。

**Q: 什麼是「LLM-native 格式」？**
A: PLAN 和 CONTRACT 使用 `[TYPE] key=value` 格式，比 Markdown 表格省 ~40% tokens，LLM 讀寫更穩定。人類可讀版本在 temp/ 摘要和 DELIVERY.md 中。

**Q: File Scope 是什麼？**
A: 每個任務限定最多 5 個可修改檔案（ALLOWED），防止 Worker 互相踩踏。需要修改範圍外的檔案必須向 TL 請求。

**Q: 需要搭配其他插件嗎？**
A: 不必。dev-team 可獨立使用。但自動偵測到其他插件時會升級體驗（如 explorer 偵察、superpowers TDD 嚴格模式）。

**Q: Worker 數量可以控制嗎？**
A: TL 根據任務數量自動決定（上限 3 個同時進行）。如果想調整，在 Phase 1 確認時告訴 TL。

**Q: Agent Metrics 精確嗎？**
A: Sub-agent（test/QA/delivery）有精確 token 數據。Worker（teammate）的 token 無法取得，標示 n/a。不計算費用。
```

**Step 2: Verify guide reflects v3.0**

Check against key v3.0 changes:
- [x] Version 3.0.0
- [x] No challenger mentioned
- [x] 1-task-per-worker explained
- [x] Separated testing flow diagram
- [x] Three-way cross-verification explained
- [x] LLM-native format explained
- [x] Distributed logs file tree
- [x] DELIVERY 8 sections listed
- [x] Updated file structure tree
- [x] FAQ updated

**Step 3: Commit**

```bash
git add plugins/dev-team/docs/GUIDE.zh-TW.md
git commit -m "docs(dev-team): rewrite guide for v3.0

Updated team structure, phases, file architecture, FAQ for v3.0 changes"
```

---

### Task 8: Final cross-reference verification

**Files:** All files from Tasks 1-7.

**Step 1: Verify file inventory**

```bash
# Prompts (should be 5 files)
ls plugins/dev-team/skills/dev-team/prompts/

# References (should be 4 files)
ls plugins/dev-team/skills/dev-team/references/

# Docs
ls plugins/dev-team/docs/GUIDE.zh-TW.md
```

Expected:
- prompts/: `delivery-sub.md`, `qa-global.md`, `qa-task.md`, `test-agent.md`, `worker.md`
- references/: `contract-template.md`, `delivery-template.md`, `log-templates.md`, `plan-template.md`

**Step 2: Verify SKILL.md cross-references**

Read SKILL.md and confirm every `Read prompts/xxx.md` and `Read references/xxx.md` points to an existing file:
- `prompts/test-agent.md` ✓
- `prompts/worker.md` ✓
- `prompts/qa-task.md` ✓
- `prompts/qa-global.md` ✓
- `prompts/delivery-sub.md` ✓
- `references/plan-template.md` ✓
- `references/contract-template.md` ✓
- `references/delivery-template.md` ✓
- `references/log-templates.md` ✓

**Step 3: Verify no obsolete references**

Search for any remaining v2.2.0 references:
```bash
grep -r "challenger" plugins/dev-team/skills/ plugins/dev-team/docs/GUIDE.zh-TW.md
grep -r "trace-template\|process-log-template\|issues-template\|qa-review-template" plugins/dev-team/skills/
grep -r "TRACE\.md\|PROCESS_LOG\|ISSUES\.md" plugins/dev-team/skills/
grep -r "peer_list" plugins/dev-team/skills/
grep -r "pricing\|MTok\|\$15\|\$75\|\$3" plugins/dev-team/skills/
```

Expected: no matches in SKILL.md or active prompts. Guide FAQ mentions challenger only to explain removal.

**Step 4: Verify version consistency**

```bash
grep "version" plugins/dev-team/.claude-plugin/plugin.json
grep "version:" plugins/dev-team/skills/dev-team/SKILL.md
grep "版本" plugins/dev-team/docs/GUIDE.zh-TW.md
```

All should show 3.0.0.

**Step 5: Final commit (if any fixes needed)**

```bash
# Only if corrections were made in Steps 2-4
git add plugins/dev-team/
git commit -m "fix(dev-team): fix cross-reference issues found in final verification"
```

---

## Summary

| Task | Files | Action |
|------|-------|--------|
| 1 | 5 files | Delete obsolete v2.2.0 files |
| 2 | 4 files | Create/rewrite reference templates (LLM-native) |
| 3 | 4 files | Create new agent prompts (test-agent, QA, delivery) |
| 4 | 1 file | Rewrite worker prompt (1-task lifecycle) |
| 5 | 1 file | Rewrite SKILL.md (complete v3.0) |
| 6 | 1 file | Update plugin.json version |
| 7 | 1 file | Rewrite GUIDE.zh-TW.md |
| 8 | — | Final cross-reference verification |

**Total: 17 file operations (5 delete + 6 create + 6 rewrite)**
