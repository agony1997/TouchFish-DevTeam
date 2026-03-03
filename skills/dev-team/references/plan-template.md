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
