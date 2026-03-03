# Delivery Report

> Project: {name} | Date: {date}
> Based on: {list of source spec paths}

## 1. Executive Summary

{2-3 sentences describing what was delivered and why}

**Completion Status:** {Complete | Partial — N of M tasks completed}

## 2. Requirements → Implementation Mapping

| Requirement | Task(s) | Files Changed | Status |
|-------------|---------|---------------|--------|
| {req description} | T-{id} | {file paths} | {done/partial/failed/deferred} |

## 3. API Contract (Final)

{Human-readable version of CONTRACT.md — convert LLM-native to Markdown tables}
{Write "N/A — no API endpoints" if Phase 1 was skipped}

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

## 7. Known Issues & Future Work

- [ ] {item} — {reason}

{Write "None" if no known issues}
