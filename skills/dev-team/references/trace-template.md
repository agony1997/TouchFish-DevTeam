# Traceability Matrix

> Project: {name} | Created: {date} | Updated: {date}

## Source Documents

| ID | Document | Path | Type |
|----|----------|------|------|
| S1 | {spec name} | {path} | upstream |
| S2 | API Contract | {date}-API_CONTRACT.md | dev-team |

## Requirement → Task Mapping

| Req | Source | Task | Worker | Status | QA | Fix Task | Notes |
|-----|--------|------|--------|--------|----|----------|-------|
| R01 | S1:{section} | T-01 | — | pending | — | — | |

<!-- Status flow: pending → in-progress → done → qa-pass / qa-fail → fixed → qa-pass
     qa-fail creates a Fix Task (T-xx-fix). Fill Fix Task column with the fix task ID.
     If fix also fails QA, create another fix task (T-xx-fix2). -->
<!-- One requirement may map to multiple tasks. Use multiple rows with same Req. -->

## API Contract Trace

| Endpoint | Backend Task | Frontend Task | Verified | Notes |
|----------|-------------|---------------|----------|-------|

## Summary

- Total: 0 | Passed: 0 | Deferred: 0 | Open issues: 0

<!-- TL maintains this file. Update after each status change. -->
