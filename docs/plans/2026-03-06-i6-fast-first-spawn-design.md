# Design: E2-I6 Fast-First Sub-Batch Spawn

**Date**: 2026-03-06
**Issue**: E2-I6 — Parallel tool calls block on slowest agent, all others idle
**Target version**: v1.5.0

## Problem

Claude Code's parallel tool calls require ALL agents to return before TL regains control. When TL spawns heterogeneous workers in one batch (e.g., worker-2 at 5min + worker-7 at 13min), fast workers sit idle for 8+ minutes. TL cannot process their results or spawn successors until the slowest agent returns.

This is a compound issue — it amplifies E2-I1 (pipeline idle), E2-I2 (Vue task slow), and E2-I4 (compaction from idle agent context).

## Solution: Complexity Estimation + Sub-Batch Waves

Two changes at two levels:

### P0: Task Complexity Estimation

Each `[TASK]` in PLAN gets a `complexity` field:

```
[TASK] id=T2 | name=Auth Endpoints | ... | complexity=LOW | reason=spring-crud x2
[TASK] id=T7 | name=Task List Page | ... | complexity=HIGH | reason=vue-components x5+jsdom-mocking
```

**Classification rules:**
- **LOW**: Pure backend CRUD, config, utility; <=3 ALLOWED files; no complex mocking
- **MED**: Cross-module integration, auth/security; or 4-5 ALLOWED files
- **HIGH**: Frontend UI components, complex framework mocking (Pinia/vue-router/jsdom), or TL judges >2x average duration

`reason` is a short justification for traceability (survives compaction via PLAN file).

### P0: Wave Sub-Batches

`[WAVE]` line splits heterogeneous waves into sub-batches:

```
Before: [WAVE] 1=T1,T6 | 2=T2,T3,T7 | 3=T4 | 4=T5
After:  [WAVE] 1=T1,T6 | 2a=T2,T3 | 2b=T7 | 3=T4 | 4=T5
```

**Sub-batch rules:**
- Homogeneous wave (all same complexity) → no split
- Heterogeneous wave (LOW/MED + HIGH mixed) → split into `Na=LOW/MED` + `Nb=HIGH`
- `a` batch (fast) ordered before `b` batch (slow)
- All-HIGH wave → no split (no fast batch to prioritize)

### P2: Sequential Sub-Batch Execution

TL spawns agents per PLAN sub-batch order:
- Within a sub-batch → parallel spawn (current behavior)
- Between sub-batches → sequential: fully process batch `a` results before spawning batch `b`

Concrete flow for wave 2:
```
Batch 2a (T2, T3 - LOW):
  1. parallel spawn test-agent-2, test-agent-3
  2. return → parallel spawn worker-2, worker-3
  3. return → spawn qa-task-2, qa-task-3

Batch 2b (T7 - HIGH):
  4. spawn test-agent-7
  5. return → spawn worker-7
  6. return → spawn qa-task-7
```

The `<= 3 Workers concurrent` rule is unchanged. Sub-batches are naturally within this limit.

## Expected Impact (E2 Data Simulation)

```
E2 actual (all parallel):
  t+0:  worker-2, worker-3, worker-7 start simultaneously
  t+5:  worker-2/3 done → idle, waiting for worker-7
  t+13: worker-7 done → TL unblocks → spawns qa-2/3/7 + test-agent-4
  t+18: test-agent-4 done → worker-4

New design (fast-first sub-batch):
  t+0:  worker-2, worker-3 start (batch 2a)
  t+5:  done → spawn qa-2/3 + test-agent-4; start test-agent-7 (batch 2b)
  t+10: test-agent-4 done → worker-4; test-agent-7 done → worker-7
  t+14: worker-4 done → qa-4 + test-agent-5
  t+23: worker-7 done → qa-7
```

| Metric | E2 Actual | New Design | Delta |
|--------|-----------|------------|-------|
| worker-4 start time | t+18 | t+10 | **-8 min** |
| worker-7 finish time | t+13 | t+23 | **+10 min** |
| T4->T5 chain completion | ~t+30 | ~t+22 | **-8 min** |

**Key trade-off**: worker-7 finishes ~10 min later, but the backend critical path (T4->T5) starts ~8 min earlier. From E2 data, the backend chain was the actual critical path, so this is a net improvement.

## Risks

| Risk | Severity | Mitigation |
|------|----------|------------|
| Complexity estimation wrong (HIGH marked LOW) | Low | Degrades to current behavior; `reason` field aids review |
| Critical path misjudged (frontend is bottleneck) | Medium | All-HIGH waves don't split; only heterogeneous waves affected |
| P0 decision overhead increases | Low | Only two new fields; classification rules are simple |
| Compaction loses sub-batch logic | Low | Sub-batches are in PLAN file on disk; re-read recovers |

## Scope

**Changes:**
- `SKILL.md` — P0 section (complexity estimation rule + wave sub-batch rule) + P2 section (sub-batch spawn rule)
- `references/plan-template.md` — add complexity field + sub-batch wave format

**Not changed:**
- Agent prompts (test-agent, worker, qa-task, qa-global, delivery-sub) — they don't need to know about sub-batches
- I1 (pipeline idle from test-agent wait) — deferred to next version
- No numeric scoring (1-10); three levels (LOW/MED/HIGH) are sufficient
