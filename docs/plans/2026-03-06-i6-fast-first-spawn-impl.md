# E2-I6 Fast-First Sub-Batch Spawn — Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Add complexity estimation + sub-batch wave splitting to dev-team skill so TL processes fast workers before spawning slow ones.

**Architecture:** Two files changed — SKILL.md (P0 + P2 rules) and plan-template.md (new fields). Pure prompt/template edits, no code, no tests.

**Tech Stack:** Markdown (LLM-native format)

---

### Task 1: Add complexity + sub-batch to plan-template.md

**Files:**
- Modify: `skills/dev-team/references/plan-template.md`

**Step 1: Add Tasks section with complexity field**

Insert after the `[SPEC]` line, before the end of file:

```markdown
[SECTION] Cross-Cutting Concerns
[CROSS-CUTTING] {shared strategies, pipe-separated: error-format, auth, logging, etc. "none" if not applicable}

[SECTION] Tasks
[TASK] id=T1 | name={task name} | spec={spec id} | deps={dependency task ids or "none"} | files={ALLOWED file count} | complexity={LOW|MED|HIGH} | reason={short justification}

[SECTION] Task Dependency Graph
[DEP] {dependency chain, e.g. T1 → T2, T3 (parallel) → T4}
[WAVE] {wave assignments with sub-batches for heterogeneous waves, e.g. 1=T1,T6 | 2a=T2,T3 | 2b=T7 | 3=T4}
```

Add a comment block above the Tasks section explaining complexity classification:

```markdown
<!-- Complexity classification:
     LOW  = backend CRUD, config, utility; <=3 files; no complex mocking
     MED  = cross-module integration, auth/security; or 4-5 files
     HIGH = frontend UI, complex framework mocking (Pinia/vue-router/jsdom), or >2x avg duration

     Wave sub-batch rules:
     - Homogeneous wave (all same complexity) → no split (e.g. 1=T1,T6)
     - Heterogeneous wave (LOW/MED + HIGH) → split: Na=LOW/MED tasks | Nb=HIGH tasks
     - a-batch (fast) before b-batch (slow)
     - All-HIGH wave → no split -->
```

**Step 2: Verify the template reads correctly**

Read the file back. Confirm:
- `[TASK]` line has all 7 fields: id, name, spec, deps, files, complexity, reason
- `[WAVE]` example shows sub-batch notation (`2a=`, `2b=`)
- Comment block has classification rules + sub-batch rules

**Step 3: Commit**

```bash
git add skills/dev-team/references/plan-template.md
git commit -m "feat(plan-template): add complexity field and sub-batch wave format"
```

---

### Task 2: Add complexity estimation rule to SKILL.md P0

**Files:**
- Modify: `skills/dev-team/SKILL.md` (lines 132-148, Phase 0 section)

**Step 1: Add complexity estimation to step 10 (TaskCreate)**

After the existing step 10 (line 132-135), add complexity estimation rules:

```markdown
10. TaskCreate: break into tasks.
   - **Granularity anchor**: ALLOWED files ≤ 5 per task. >5 → must split. 1 file + trivial → merge into adjacent.
   - Each task description includes `File Scope: ALLOWED + READONLY`.
   - Two tasks need same file → same worker OR blockedBy.
   - **Complexity estimation**: assign each task LOW, MED, or HIGH:
     - LOW: backend CRUD, config, utility; ≤3 ALLOWED files; no complex mocking.
     - MED: cross-module integration, auth/security; or 4-5 ALLOWED files.
     - HIGH: frontend UI components, complex framework mocking, or TL judges >2x average duration.
     Record `complexity` + `reason` in PLAN `[TASK]` lines.
```

**Step 2: Add sub-batch rule to PLAN writing step**

Modify step 10 (the second one, line 137) — the PLAN writing step — to include sub-batch wave construction:

```markdown
10. Read `references/plan-template.md` → write PLAN.md (LLM-native).
    **Wave sub-batching**: when a wave contains mixed complexity (LOW/MED + HIGH),
    split into sub-batches: `Na=LOW/MED tasks | Nb=HIGH tasks`.
    Homogeneous waves (all same complexity) → no split.
```

**Step 3: Verify edits**

Read SKILL.md P0 section back. Confirm:
- Step 10 (TaskCreate) has complexity estimation rules with 3 levels
- Step 10 (PLAN write) mentions sub-batch wave construction
- No other P0 steps changed

**Step 4: Commit**

```bash
git add skills/dev-team/SKILL.md
git commit -m "feat(SKILL): add complexity estimation rules to P0"
```

---

### Task 3: Add sub-batch spawn rule to SKILL.md P2

**Files:**
- Modify: `skills/dev-team/SKILL.md` (lines 165-210, Phase 2 section)

**Step 1: Add sub-batch execution rule**

Insert after the "Context budget rule" paragraph (line 170) and before the per-task execution loop (line 172):

```markdown
**Sub-batch execution rule**: TL spawns agents per PLAN sub-batch order.
Within a sub-batch → parallel spawn (current behavior).
Between sub-batches → sequential: fully process batch `a` results
(test-agent → worker → QA for all tasks in batch) before spawning batch `b`.
This prevents slow agents from blocking fast agents' successors.
If PLAN has no sub-batches for a wave, spawn all tasks in parallel (default).
```

**Step 2: Verify edits**

Read SKILL.md P2 section back. Confirm:
- Sub-batch execution rule appears before the per-task loop
- Rule specifies: within sub-batch = parallel, between sub-batches = sequential
- Default behavior (no sub-batches) is unchanged
- Per-task execution loop (a/b/c) is NOT modified
- ≤ 3 Workers concurrent rule is NOT modified

**Step 3: Commit**

```bash
git add skills/dev-team/SKILL.md
git commit -m "feat(SKILL): add sub-batch spawn rule to P2"
```

---

### Task 4: Update version number + verify full SKILL.md

**Files:**
- Modify: `skills/dev-team/SKILL.md` (line 15, version comment)

**Step 1: Bump version**

Change line 15 from:
```
<!-- version: 1.4.0 -->
```
to:
```
<!-- version: 1.5.0 -->
```

**Step 2: Full verification read**

Read the entire SKILL.md. Walk through checklist:
- [ ] P0 step 10 has complexity estimation (LOW/MED/HIGH) with classification rules
- [ ] P0 step 10 (PLAN write) mentions sub-batch wave construction
- [ ] P2 has sub-batch execution rule before per-task loop
- [ ] P2 per-task loop (a/b/c) is unchanged
- [ ] P2 ≤ 3 Workers concurrent is unchanged
- [ ] Version is 1.5.0
- [ ] No other sections modified

**Step 3: Read plan-template.md for final check**

- [ ] `[TASK]` line has complexity + reason fields
- [ ] `[WAVE]` shows sub-batch notation
- [ ] Comment block has classification + sub-batch rules

**Step 4: Commit**

```bash
git add skills/dev-team/SKILL.md
git commit -m "chore: bump dev-team version to 1.5.0"
```
