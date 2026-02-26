# Delivery Report

> Project: {name} | Date: {date}
> Based on: {list of source spec paths}

## Summary

{2-3 sentences describing what was delivered}

## Implemented Items

### Backend

- [x] T-{ID}: {description} — {files changed}

### Frontend

- [x] T-{ID}: {description} — {files changed}

## QA Results

| Task | Result | Rounds | Notes |
|------|--------|--------|-------|

## Agent Metrics

### Team Composition

| Agent | Model | Role | Tasks Completed |
|-------|-------|------|-----------------|
| TL | opus | Planner + QA gate | — |
| challenger | sonnet | Devil's advocate | — |
| {worker} | {model} | {role} | {count} |

### Resource Usage

| Agent | Tokens (in/out) | Duration | Est. Cost (USD) | Source |
|-------|-----------------|----------|-----------------|--------|
| TL | n/a | {duration} | n/a | tracked |
| challenger | n/a | {duration} | n/a | tracked |
| {worker} | n/a | {duration} | n/a | tracked |
| QA ({n} reviews) | {in} / {out} | {duration} | ${cost} | exact |
| **Total** | | **{total}** | **~${total}** | |

<!-- Source: "exact" = from Task tool usage return (QA sub-agents only), "tracked" = TL wall-clock tracking.
     Token counts for teammates (TL, workers, challenger) are NOT available via API — marked n/a. -->

### QA Cost Breakdown (exact data only)

| QA Batch | Reviews | Tokens (in/out) | Cost (USD) |
|----------|---------|-----------------|------------|
| Task QA | {n} | {in}/{out} | ${cost} |
| Contract QA | {n} | {in}/{out} | ${cost} |
| **QA Total** | | | **${total}** |

<!-- Pricing (as of 2026-02): Opus in=$15/MTok out=$75/MTok | Sonnet in=$3/MTok out=$15/MTok | Haiku in=$0.80/MTok out=$4/MTok -->

## Contract Verification

- Backend <-> Contract: {pass/fail}
- Frontend <-> Contract: {pass/fail}

## Deferred / Manual Items

- [ ] {item} — {reason}

## Issues Summary

- Total: 0 | Resolved: 0 | Deferred: 0
- Details: see {date}-ISSUES.md

## Cross-references

- {date}-TRACE.md | {date}-PROCESS_LOG.md | {date}-ISSUES.md | {date}-API_CONTRACT.md
