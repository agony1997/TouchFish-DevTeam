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
