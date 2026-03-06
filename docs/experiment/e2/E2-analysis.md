# E2 Analysis: dev-team v1.4.0

Test: 2026-03-06, dev-team v1.4.0, TaskFlow M1+M2+M3+M6
Branch: `experiment/E2-v1.4.0`

## E1 vs E2 Summary

| Metric | E1 (v1.3.0) | E2 (v1.4.0) | Delta |
|--------|------------|------------|-------|
| Tasks | 7/7 | 7/7 | — |
| Tests | 442 (297+145) | 334 (247+87) | -24% |
| TL session duration | ~79 min | ~60 min | -24% |
| Total wall time (incl. prep) | ~79 min | ~95 min | +20% (see Q7) |
| Global QA | FAIL → fix → PASS | PASS (first attempt) | Significant |
| P3 TL fixes | 5 files | 0 | Eliminated |
| P2 TL interventions | 7 | 1 (import fix) | -86% |
| Per-task QA max rounds | 1 | 3 (T2) | Stricter QA |
| CONTRACT amendments | v1→v2 (status filter) | None | Eliminated |
| Cross-task consistency | FAIL | PASS | Fixed |
| Compaction events | 1 (P3, 161K, disrupted qa-global) | 1 (P2→P3, 168K, no disruption) | Improved |

### v1.4.0 Fix Verification

| E1 Issue | v1.4.0 Fix | E2 Result | Verdict |
|----------|-----------|-----------|---------|
| I1: Context compaction disruption | tl-state.md recovery + minimal return + router-not-relay | Compaction occurred but TL recovered seamlessly | EFFECTIVE |
| I2: Cross-worker file conflicts | PARALLEL AWARENESS in worker prompt | Worker-3 temporarily renamed conflicting file, restored after | EFFECTIVE |
| I3: CONTRACT missing status filter | P1 AC cross-check rule | CONTRACT v1 included status from the start | EFFECTIVE |
| I4: Error format inconsistency | P0 cross-cutting concerns | All handlers use `@ExceptionHandler` + `Map.of("message",...)` | EFFECTIVE |
| I7: Test syntax errors | test-agent OUTPUT syntax validation | Only 1 minor issue (missing import) vs 2 API misuses in E1 | IMPROVED |

---

## Observations & Issues

### Q1: Global QA scanned unchanged code

**Observation**: qa-global ran `mvn test` (all 247 backend tests) and `npx vitest run` (all 87 frontend tests), including skeleton code that existed before E2.

**Analysis**: This is by design. Global QA's 6 checks are inherently cross-cutting:
- CHECK 1 (consistency): needs to compare ALL task implementations against each other
- CHECK 2 (contract compliance): needs to verify every endpoint exists
- CHECK 4 (integration): needs to trace imports/dependencies across all files
- CHECK 6 (full test suite): runs ALL tests to catch cross-task regressions

The per-task QA already verified individual tasks. Global QA's unique value is *integration between tasks* — you can't do that by only looking at changed files.

**Issue for large projects**: For a production codebase with 10K+ tests, running the full suite in qa-global would take too long. Options:
1. qa-global runs only cross-task integration tests (not unit tests)
2. Worker-level QA already ran per-task tests; qa-global trusts those results and only runs the full suite as a final sanity check
3. Use test filtering (e.g., `mvn test -pl changed-modules-only`)

**Severity**: Low for current scope. Medium for production-scale projects.

### Q2: Initial project scan on large projects

**Observation**: P0 reads all specs, scans project structure, reads existing code files, and produces a PLAN. For TaskFlow (~30 source files) this took ~10 min.

**Analysis**: P0 scans are bounded by:
- Number of spec files (user-controlled, typically 1-5)
- Existing codebase files that overlap with spec scope
- Framework config files (pom.xml, package.json, vite.config, etc.)

For a large monorepo (thousands of files), the risk is TL reading too many files during P0. Current mitigations:
- TL reads specs by path (not by scanning all files)
- TL reads specific config files (pom.xml, not all Java files)
- PLAN format is compact (`[SECTION]`, `[SPEC]`, `[DEP]`, `[WAVE]`)

**Potential issue**: If specs reference many existing modules, TL might read dozens of files to understand cross-cutting concerns. This could consume significant context early.

**Severity**: Low-Medium. Worth monitoring in larger projects. May need a "scan budget" rule in P0.

### Q3: Test/QA agent idle periods

**Observation**: Between waves, there are periods where some agents are idle waiting for dependencies.

**Timeline reconstruction**:
```
07:26 ─── P0+P1: TL reads specs, creates PLAN+CONTRACT (10 min)
07:36 ─┬─ test-agent-1 (T1, ~5min)
       └─ test-agent-6 (T6, ~5min)
07:41 ─┬─ worker-1 (T1, ~3min) ← test-agent-1 done
07:42 ─└─ worker-6 (T6, ~3min) ← test-agent-6 done
07:44 ─── worker-1 done → qa-task-1 + test-agents 2,3,7 spawned
         ┌─ test-agent-2 (~4min)
         ├─ test-agent-3 (~4min)
         └─ test-agent-7 (~5min)
       *** 4-5 min where 3 test-agents run, no workers active ***
07:48 ─┬─ worker-2 (T2, ~5min) ← test-agent-2 done
       └─ worker-3 (T3, ~4min) ← test-agent-3 done
07:50 ─── worker-7 (T7, ~13min) ← test-agent-7 done
07:53 ─── worker-2,3 done → qa-task-2,3 + test-agent-4 spawned
07:58 ─── worker-4 (T4, ~4min)
08:02 ─── worker-4 done → qa-task-4 + test-agent-5
08:04 ─── qa-task-7 spawned (worker-7 done after ~13min)
08:05 ─── worker-5 (T5, ~5min)
08:10 ─── worker-5 done → qa-task-5
08:12 ─── P3: qa-global
08:15 ─── P4: delivery-sub
08:25 ─── Complete
```

**Idle windows**:
1. **07:44–07:48** (~4 min): test-agents 2,3,7 writing tests; no workers running
2. **07:53–07:58** (~5 min): test-agent-4 writing tests; workers 2,3 done, QA running in parallel
3. **08:02–08:05** (~3 min): test-agent-5 writing tests; worker-4 done

**Root cause**: The sequential pipeline (test-agent → worker → QA) means each phase gate adds latency. Test-agents take 4-5 min, and workers can't start until tests are written.

**Possible optimization**: Overlap test-agent-N+1 with worker-N. TL could speculatively start the next wave's test-agents while current workers are still running. This would reduce pipeline bubbles but add concurrency complexity.

**Severity**: Medium. ~12 min of idle time out of 60 min = 20% pipeline inefficiency.

### Q4: Worker-7 (Vue frontend) is consistently slow

**E1**: Worker-7 stuck for ~24 min (8 Edit + 7 debug cycles, vue-router click test)
**E2**: Worker-7 took ~13 min, 3 session files (553K+499K+550K = ~1.6MB)

Compare with other E2 workers:
| Worker | Duration | Session Size | Task |
|--------|----------|-------------|------|
| worker-1 | ~3 min | 237K×2 | User + JWT (Java) |
| worker-6 | ~3 min | 198K×2 | API layer (TS, no UI) |
| worker-2 | ~5 min | 374K×3 | Auth endpoints (Java) |
| worker-3 | ~4 min | 270K×2 | Task entity (Java) |
| worker-4 | ~4 min | 315K×2 | Task CRUD (Java) |
| worker-5 | ~5 min | 249K×2 | State machine (Java) |
| **worker-7** | **~13 min** | **553K×3** | **5 Vue components** |

**Why T7 is harder**:
1. **Scope**: 5 component files to create (most of any task); 49 tests to pass (most of any task)
2. **Framework quirks**: jsdom converts hex→rgb in inline styles, Pinia instance management, vue-router mocking
3. **Assertion precision**: Tests check exact hex color codes (`#6B7280`), debounce timing (300ms), 0-indexed pagination math
4. **E2 improvement**: impl-notes-7 warned about Pinia mocking, badge colors, component stubs — worker-7 avoided the vue-router debug loop from E1
5. **Still hit 2 failures**: Pinia double-instance (test-agent wrote `createPinia()` in mountPage AND `setActivePinia` in beforeEach — conflicting)

**Conclusion**: T7 is objectively the hardest task for AI. Vue component testing with visual assertions and framework-specific mocking is an AI-unfriendly domain. The impl-notes mechanism helped (13 min vs 24 min) but this task type will always be the bottleneck.

**Possible mitigations**:
- Split T7 into 2 tasks (page + components separately)
- Use Opus for frontend workers instead of Sonnet
- impl-notes could include a "test fixture template" for common mount patterns

### Q5: ccusage only shows Opus consumption

**Observation**: ccusage (token usage tracker) only shows Opus model consumption; Sonnet teammate usage is invisible.

**Analysis**: This is a Claude Code platform behavior:
- TL runs as Opus — its API calls are tracked
- test-agents and QA agents run as Opus subagents — also tracked (same billing)
- Workers run as Sonnet **teammates** — these use the Agent tool with `model` parameter

Teammates may be billed through a different channel or bundled into the parent session's usage. ccusage hooks into the API response headers which only reflect the current session's model.

**Impact**: Cannot accurately measure total cost or Sonnet vs Opus cost ratio.

**Workaround**: Estimate from session file sizes:
- Opus sessions (test-agents + QA + qa-global): ~14 agents, ~200-340K each
- Sonnet sessions (workers + delivery): ~8 agents, ~200-550K each
- Rough input token estimate: Opus ~60-70%, Sonnet ~30-40% of total

**Severity**: Medium. Affects cost analysis but not functionality.

### Q6: One compaction event

**Details**:
- **When**: 08:09:49 UTC (43 min into TL session, at P2→P3 transition)
- **preTokens**: 168,299
- **Phase**: 6/7 tasks complete, only T5 QA remaining
- **Impact**: None observed — TL continued seamlessly into P3

**Comparison with E1**:
- E1: compaction at 161K during P3 (global QA), lost qa-global agent ID, required restart
- E2: compaction at 168K at P2 end, tl-state.md recovery worked, no disruption

**Why compaction still happened**:
The v1.4.0 "minimal return" and "router-not-relay" rules reduced context growth but didn't eliminate it. TL still accumulates:
- Agent spawn prompts (7 test-agents + 7 workers + 6 QA + 1 global = 21 spawn calls)
- Agent return summaries (even minimal, 21 returns × ~500 tokens each ≈ 10K)
- TL's own reasoning between agent dispatches
- Tool call results (tl-state reads, file reads for test fixes)

168K tokens for 21 agents across 4 waves is ~8K per agent cycle — a reasonable floor.

**Conclusion**: Compaction is likely inevitable for 7-task projects. The key improvement is that it's now **non-disruptive** thanks to tl-state.md recovery.

### Q7: E2 total wall time longer than E1

**Breakdown**:
```
E1: ~79 min (single TL session, P0→P4)
E2:  ~6 min  session 01-02 (branch prep + skeleton verification)
    ~25 min  gap + session 03 (failed first attempt + human intervention)
    ~60 min  session 04 (successful TL session, P0→P4)
    ─────────
    ~95 min  total wall time
```

**Key insight**: The successful E2 TL session (60 min) was actually **19 min faster** than E1 (79 min).

The extra wall time came from:
1. **Branch/skeleton prep** (~6 min): Setting up the experiment branch and verifying skeleton
2. **Failed first attempt** (~25 min including gap): Session 03 appears to have failed early; required human restart
3. **Human think/intervention time**: Gaps between sessions

**Why the successful session was faster**:
- No P3 TL fixes needed (E1 spent ~10-15 min fixing 5 files after qa-global FAIL)
- Worker-7 took 13 min instead of 24 min (impl-notes helped)
- No CONTRACT amendment mid-flight (E1 had v1→v2 disruption)

**Conclusion**: v1.4.0 made the *core execution* faster. The perceived longer duration is due to experiment setup overhead, not skill regression.

---

## New Issues Found in E2

### E2-I1: Pipeline idle time (~20% waste)

Sequential test→worker→QA pipeline creates idle windows between waves. ~12 min of 60 min is waiting for test-agents.

**Potential fix**: Allow TL to speculatively spawn next-wave test-agents while current workers are still running. This is safe because test-agents don't modify source files — they only write test files.

### E2-I2: T7 (Vue component) task inherently slow

Vue component tasks take 3-4x longer than Java backend tasks. This is consistent across E1 and E2.

**Potential fix**: Split large frontend tasks; consider Opus for frontend workers; richer impl-notes with mount templates.

### E2-I3: ccusage cannot track teammate (Sonnet) consumption

Cost visibility is incomplete. Only Opus usage is tracked.

**Status**: Platform limitation. No skill-level fix available.

### E2-I4: Compaction still occurs at ~168K tokens

Minimal return reduced growth rate but 21 agent cycles still hit the limit. Non-disruptive now, but limits scalability to ~7 tasks per TL session.

**Potential fix**: For >7 task projects, consider splitting into multiple TL sessions (batch 1: T1-T4, batch 2: T5-T8).

### E2-I5: qa-global runs full test suite redundantly

Per-task QA already verified all tests pass. qa-global re-runs everything as a sanity check. For large projects this could be prohibitive.

**Potential fix**: qa-global trusts per-task QA test results; only runs cross-task integration checks (consistency, contract compliance, import resolution) without re-executing the full test suite.

### E2-I6: Parallel tool calls block on slowest agent — all others idle (CRITICAL)

**Observed**: User manually inspected all agent sessions mid-run and confirmed every single one was idle. 12+ agents sitting idle simultaneously while TL was blocked waiting for worker-7.

**Root cause**: Claude Code's parallel tool calls require ALL agents to return before TL regains control. When TL spawns agents in parallel:

```
TL spawns: worker-1 (3min), worker-3 (4min), worker-7 (13min) — parallel

t+3min:  worker-1 DONE → idle, waiting for worker-7
t+4min:  worker-3 DONE → idle, waiting for worker-7
         TL also spawned qa-task-1, test-agent-2,3,7 → they finish fast → idle
         12+ agents all idle, all waiting for worker-7
t+13min: worker-7 DONE → TL finally unblocks
         TL discovers a pile of completed agents it hasn't processed yet
```

**Impact**:
- ~8-10 min of total idle time (all agents waiting for worker-7 alone)
- 12+ idle agent sessions consuming TL context space → accelerates compaction
- TL cannot process completed fast agents to spawn their successors (QA, next wave test-agents)

**This is the single biggest efficiency issue** — it compounds E2-I1 (pipeline idle), E2-I2 (Vue task slow), and E2-I4 (compaction). If TL could process fast agents as they complete instead of waiting for the slowest, the pipeline would flow much more smoothly.

**Potential fixes**:
1. **Avoid parallel spawn for heterogeneous tasks**: Don't batch Java workers (3-5min) with Vue workers (13min) in the same parallel call. Spawn worker-7 independently so it doesn't block others.
2. **Staggered spawn**: Instead of parallel tool calls, spawn agents one-by-one and use a polling loop to check completion. This lets TL process fast agents immediately. (Requires Claude Code to support non-blocking agent checks — may not be available today.)
3. **Task complexity estimation in PLAN**: During P0, TL estimates relative complexity per task (Java backend = LOW, Vue components = HIGH). HIGH tasks get their own wave to avoid blocking.
