# PLAN Template (LLM-native)

> For TL use in Phase 0. Write as `{date}-PLAN.md` to output dir.
> Target size: 500-1500 tokens. Workers MUST read this at task start.

---

[SECTION] Project Overview
[GOAL] {one-line project goal}
[ARCH] {key architectural decisions, pipe-separated}
[NAMING] {naming conventions: language-specific rules}
[CONSTRAINTS] {critical constraints, pipe-separated}
[SHARED] {cross-task shared resources: file paths, pipe-separated}

[SECTION] Standards
[SOURCE] {standards file paths, pipe-separated. "none" if user said no standards}
[OVERRIDE] follow-actual-code | {noted discrepancies between docs and actual code}

[SECTION] Specs
[SPEC] id=S1 | name={spec name} | path={spec file path}

[SECTION] Cross-Cutting Concerns
[CROSS-CUTTING] {shared strategies, pipe-separated: error-format, auth, logging, etc. "none" if not applicable}

<!-- Complexity classification:
     LOW  = backend CRUD, config, utility; <=3 files; no complex mocking
     MED  = cross-module integration, auth/security; or 4-5 files
     HIGH = frontend UI, complex framework mocking (Pinia/vue-router/jsdom), or >2x avg duration

     Wave sub-batch rules:
     - Homogeneous wave (all same complexity) → no split (e.g. 1=T1,T6)
     - Heterogeneous wave (LOW/MED + HIGH) → split: Na=LOW/MED tasks | Nb=HIGH tasks
     - a-batch (fast) before b-batch (slow)
     - All-HIGH wave → no split -->

[SECTION] Tasks
[TASK] id=T1 | name={task name} | spec={spec id} | deps={dependency task ids or "none"} | files={ALLOWED file count} | complexity={LOW|MED|HIGH} | reason={short justification}

[SECTION] Task Dependency Graph
[DEP] {dependency chain, e.g. T1 → T2, T3 (parallel) → T4}
[WAVE] {wave assignments with sub-batches for heterogeneous waves, e.g. 1=T1,T6 | 2a=T2,T3 | 2b=T7 | 3=T4}
