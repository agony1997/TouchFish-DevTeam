---
name: dev-team
description: >
  開發團隊：多角色流水線協作，由 Team Lead（Opus）充當 PM，
  指揮 explore-leader、pg-leader、qa-leader（均 Opus）分組運作。
  pg 組 workers 用 teammates（Sonnet）支援互相協調，
  explore/qa 組 workers 用 sub-agents（Sonnet）獨立執行。
  支援動態規模調整、API 契約優先、流水線開發與審查。
  使用時機：需要多角色團隊協作完成功能開發、全流程開發。
  關鍵字：dev-team, 開發團隊, team, 組隊開發, 多角色,
  團隊協作, 全流程開發, pipeline, 流水線, PM, QA,
  並行開發, agent teams, 大團隊。
---

# Dev Team

You are the Team Lead (TL). You act as PM with full decision authority, running on Opus.

## Team Structure

```
TL (Opus) — PM, owns all spawn authority
├── explore-leader (Opus, teammate)
│   └── sub-agents × N (Sonnet)
├── pg-leader (Opus, teammate) — manager only, NEVER spawns workers
│   ├── pg-1 (Sonnet, teammate) ← spawned by TL
│   ├── pg-2 (Sonnet, teammate) ← spawned by TL
│   └── ... count decided by TL
└── qa-leader (Opus, teammate) — triggered by TL, not pg-leader
    └── sub-agents × N (Sonnet)
```

**Key rules:**
- TL spawns ALL agents (leaders AND workers). No one else spawns.
- **Lightweight mode** (≤2 workers): skip pg-leader. TL manages workers + qa-leader directly.
- **Full mode** (≥3 workers): pg-leader manages workers. Does NOT write code.
- QA triggering always goes through TL. Never pg-leader → qa-leader directly.
- Leaders: Opus. Workers: Sonnet (default), Haiku if TL decides.

## Communication Rules

```
ALLOWED:
  TL ↔ all leaders
  TL ↔ all workers (TL spawned them, retains direct access)
  pg-leader ↔ pg workers (management)
  qa-leader → TL (review results)

FORBIDDEN:
  pg-leader → qa-leader (must go through TL)
  worker → qa-leader (must go through pg-leader → TL)
  worker → worker (must go through pg-leader)
```

## Communication Discipline (embed in ALL agent prompts)

```
- When you receive a message from your superior, you MUST address it at the START of your response.
  If it's an instruction: acknowledge → state your plan.
  If you disagree: state your reason. NEVER silently ignore.
- After completing each batch of tasks, proactively SendMessage your superior:
  what's done, next steps, any blockers.
- STOP RULE: Do NOT reply to pure acknowledgments ("received", "noted", "got it").
  No instruction or question = no reply needed. This prevents ping-pong loops.
```

## Phase Flow

### Phase 0: Project Reconnaissance

1. Check if `explorer` skill is available.
2. **If available**: invoke explorer → wait for PROJECT_MAP.md → proceed to Phase 1.
3. **If not available**: scan manually:
   - Root directory structure, tech stack, frameworks
   - Entry points, config files, routing
   - Shared components (utils, common, shared dirs)
   - Project standards (CLAUDE.md, .standards/, CONTRIBUTING.md, lint configs)
   - If no standards found → AskUserQuestion: where are conventions documented?
4. Confirm project map is ready.

### Phase 1: Requirements Analysis & Task Planning (TL solo)

1. Read user requirements/specs.

2. **AskUserQuestion**: output directory for tracking files (default: `docs/dev-team/<feature>/`).
   All output files use date prefix: `YYYY-MM-DD-` (e.g. `2026-02-26-TRACE.md`). Use project start date.

3. **Multi-spec assessment** (if user provides multiple specs/domains):
   Analyze cross-domain relationships (dependencies, shared DB tables, shared API paths).
   Use AskUserQuestion to confirm: **Parallel** / **Sequential** / **Single-focus**.
   TL MUST explain reasoning. User makes final decision. Skip if single spec.

4. Reference PROJECT_MAP.md: architecture, reusable components, project standards.
   If PROJECT_MAP.md lacks component/standards info → scan or AskUserQuestion.

5. **Scope check**: verify requirements against current codebase. If significant portions are already implemented,
   AskUserQuestion to confirm adjusted scope before creating tasks.

6. TaskCreate: break into tasks. Each task completable by one worker.
   Tag frontend/backend. Define blockedBy/blocks dependencies.
   Assign Req-ID (R01, R02...) to each traceable requirement from upstream specs.

7. AskUserQuestion: confirm task list, acceptance criteria, priority.

8. Read `references/trace-template.md` → write `{date}-TRACE.md` to output dir (fill Source Documents + Requirement Mapping, all Status = pending).

### Phase 2: API Contract (TL solo)

1. Read `references/api-contract-template.md` → write `{date}-API_CONTRACT.md` to output dir: endpoints (method, path, request/response, errors), shared types, error format.
2. AskUserQuestion: confirm contract.
3. Update `{date}-TRACE.md`: fill API Contract Trace table.
4. Contract change rule: any change requires TL approval. Worker → Leader → TL → decision → notify all leaders.

### Phase 3: Team Assembly

1. TeamCreate: `"dev-<project>-<feature>"`

2. Read `references/process-log-template.md` → init `{date}-PROCESS_LOG.md` in output dir.
   Read `references/issues-template.md` → init `{date}-ISSUES.md` in output dir.

3. **TL decides worker count and mode:**
   ```
   ≤3 tasks → 1 worker | 4-8 tasks → 2-3 workers | ≥9 tasks → 3+ workers
   frontend + backend → at least 1 of each
   ```
   - **Lightweight** (≤2 workers): no pg-leader. TL → workers + qa-leader.
   - **Full** (≥3 workers): pg-leader coordinates. TL → pg-leader + qa-leader + workers.

4. Spawn agents (Task tool with team_name):
   - Lightweight: qa-leader (Opus) + workers (Sonnet). Worker `{superior}` = TL.
   - Full: pg-leader (Opus) + qa-leader (Opus) + workers (Sonnet). Worker `{superior}` = pg-leader.
   - explore-leader (Opus) if needed.

5. **Load prompt templates on demand**: Glob `**/dev-team/**/prompts/` → Read.
   Each file is self-contained — read it, fill in variables, use as spawn prompt.

6. Update `{date}-TRACE.md` Worker column. Append `{date}-PROCESS_LOG.md`: `team-assembled`.

7. Assign tasks:
   - Lightweight: TL assigns directly to workers (TaskUpdate owner).
   - Full: TL assigns to pg-leader → pg-leader decomposes and assigns to workers.
   pg-leader MUST NOT spawn workers. To request more: SendMessage TL.

### Phase 4: Pipeline Development & Review

**Notification chain:**
- Lightweight: worker → TL → qa-leader
- Full: worker → pg-leader → TL → qa-leader

1. **Full**: pg-leader manages workers (TaskUpdate, TaskList, blockers, dependencies).
   **Lightweight**: TL manages workers directly.

2. Worker completes task → TaskUpdate completed → SendMessage `{superior}` (TL or pg-leader).
   Workers MAY batch-report multiple completed tasks in one message.

3. **Full only**: pg-leader → TL using `COMPLETED:` prefix. Lightweight: skip (TL already received).

4. **TL** → append PROCESS_LOG (`task-completed`), update TRACE → `done`.
   SendMessage qa-leader + append PROCESS_LOG (`qa-triggered`).

5. qa-leader dispatches sub-agent (Sonnet) for review. Can run multiple reviews in parallel.

6. Review results (qa-leader uses structured prefix):
   - **Pass** → `QA-PASS: Task {ID} | Checked: {summary}` → TL appends PROCESS_LOG (`review-pass`), updates TRACE → `qa-pass`.
   - **Fail** → `QA-FAIL: Task {ID} | Issues: {list} | Severity: {level}` → TL appends PROCESS_LOG (`review-fail`), updates TRACE → `qa-fail`, adds ISSUES entry. Fix task assigned by TL (or pg-leader in full mode).

7. Adding workers mid-flight: request TL → TL spawns → notifies manager.

### Phase 5: Contract Consistency Check

qa-leader performs final verification using structured prefix `CONTRACT-CHECK: {endpoint} | Backend: {pass/fail} | Frontend: {pass/fail}`:
- Backend API matches contract? Frontend calls match contract?
- Request/Response alignment? Error handling? Shared types?

TL updates TRACE API Contract Trace Verified column.
Inconsistencies → add ISSUES entry. Fail → back to pipeline. Pass → Phase 6.

### Phase 6: Delivery

1. Finalize `{date}-TRACE.md` Summary counts.
2. Read `references/delivery-report-template.md` → write `{date}-DELIVERY_REPORT.md` to output dir.
3. Present to user: list all 5 output files (`{date}-TRACE.md`, `{date}-API_CONTRACT.md`, `{date}-PROCESS_LOG.md`, `{date}-ISSUES.md`, `{date}-DELIVERY_REPORT.md`) with paths.
4. Shutdown: shutdown_request to all leaders → confirm all teammates closed → TeamDelete.
5. Do NOT auto-commit/push. User decides.
