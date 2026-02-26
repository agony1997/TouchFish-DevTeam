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
- pg-leader is a MANAGER — assigns tasks, monitors progress, coordinates. Does NOT write code (exception: ≤3 tasks and TL sent no workers).
- QA triggering goes through TL: pg-leader → TL → qa-leader. Never pg-leader → qa-leader directly.
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

2. **Multi-spec assessment** (if user provides multiple specs/domains):
   Analyze cross-domain relationships:
   - Dependencies between domains?
   - Shared DB tables / entities?
   - Shared API path prefixes?

   Use AskUserQuestion to confirm execution strategy:
   - **Parallel**: domains are independent → assign separate worker groups per domain
   - **Sequential**: domain A depends on B → complete B first
   - **Single-focus**: high complexity + cross-dependencies → one domain at a time

   TL MUST explain reasoning. User makes final decision.
   Skip this step if only one spec/domain.

3. Reference PROJECT_MAP.md: architecture, reusable components, project standards.
   If PROJECT_MAP.md lacks component/standards info → scan or AskUserQuestion.

4. TaskCreate: break into tasks. Each task completable by one worker.
   Tag frontend/backend. Define blockedBy/blocks dependencies.

5. AskUserQuestion: confirm task list, acceptance criteria, priority.

### Phase 2: API Contract (TL solo)

1. Write API_CONTRACT.md: endpoints (method, path, request/response, errors), shared types, error format.
2. AskUserQuestion: confirm contract.
3. Contract change rule: any change requires TL approval. Worker → Leader → TL → decision → notify all leaders.

### Phase 3: Team Assembly

1. TeamCreate: `"dev-<project>-<feature>"`

2. Spawn leaders (Opus, Task tool with team_name): explore-leader (if needed), pg-leader, qa-leader.

3. **TL decides worker count and spawns directly:**
   ```
   ≤3 implementation tasks → no workers, pg-leader works alone
   4-8 tasks → at least 2 workers
   ≥9 tasks → at least 3 workers
   frontend + backend tasks → at least 1 of each
   ```
   TL spawns workers with team_name. Worker prompts set pg-leader as their superior.

4. **Load prompt templates on demand**: before spawning each agent, use Glob to find the matching template file under `**/dev-team/**/prompts/`, then Read it.
   - Spawning pg-leader → Read `prompts/pg-leader.md`
   - Spawning qa-leader → Read `prompts/qa-leader.md`
   - Spawning explore-leader → Read `prompts/explore-leader.md`
   - Spawning workers → Read `prompts/worker.md`
   Each file is self-contained — read it, fill in variables, use as spawn prompt.

5. TaskUpdate: assign high-level tasks to pg-leader.

6. pg-leader decomposes into fine-grained subtasks, assigns to workers (TaskUpdate owner).
   - pg-leader MUST NOT spawn workers.
   - To request more workers: SendMessage to TL. TL decides.

### Phase 4: Pipeline Development & Review

**Core principle:** All cross-group notifications go through TL. Never rely on pg-leader to notify qa-leader.

1. pg-leader manages workers: assign tasks (TaskUpdate), monitor (TaskList), handle blockers, coordinate dependencies.

2. Worker completes task → TaskUpdate completed → SendMessage pg-leader.

3. pg-leader receives completion → **SendMessage TL**: "Task X done, files: Y, next: Z"
   Do NOT notify qa-leader directly.

4. **TL receives report** → SendMessage qa-leader: "Task X complete, please review" + task description + files.

5. qa-leader dispatches sub-agent (Sonnet) for review. Can run multiple reviews in parallel.

6. Review results:
   - **Pass** → qa-leader notifies TL.
   - **Fail** → qa-leader notifies TL → TL notifies pg-leader → pg-leader creates fix task.

7. Adding workers mid-flight: pg-leader requests TL → TL spawns new worker → notifies pg-leader.

### Phase 5: Contract Consistency Check

qa-leader performs final verification:
- Backend API matches contract? (paths, methods, params)
- Frontend calls match contract? (URLs, request format)
- Request/Response alignment?
- Error handling consistency?
- Shared type definitions consistent?

Fail → back to pipeline. Pass → Phase 6.

### Phase 6: Delivery

1. TL writes delivery report: features, files, QA summary, known limitations.
2. Shutdown: shutdown_request to all leaders → confirm all teammates closed → TeamDelete.
3. Present report to user.
4. Do NOT auto-commit/push. User decides.
