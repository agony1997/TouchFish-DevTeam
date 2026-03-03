# Log File Templates (LLM-native)

> Each agent writes ONLY its own log file. Append-only — never edit previous entries.
> All logs stored in `{output-dir}/logs/` subdirectory.

---

## TL Log — `logs/tl.log.md`

[LOG] phase={P0-P4} | event={type} | {key=value details}

Event types: plan-created, contract-created, contract-amendment,
worker-spawned, worker-replaced, worker-crashed, phase-transition,
blocker-resolved, escalation, sub-agent-metrics

Sub-agent metrics (TL records after each sub-agent returns):
[LOG] phase={phase} | event=sub-agent-metrics | agent={name} | task={task-id} | model={model} | input={tokens} | output={tokens}

---

## Test Agent Log — `logs/test-agent-{N}.log.md`

[LOG] task={task-id} | event=test-design-start | strategy={red-green,boundary,error,contract}
[LOG] task={task-id} | event=test-design-complete | test-files={list} | test-count={number}

---

## Worker Log — `logs/worker-{N}.log.md`

[LOG] task={task-id} | event=start | files={allowed-files}
[LOG] task={task-id} | event=issue | detail={description}
[LOG] task={task-id} | event=complete | files-changed={list} | tests-passed={all|partial}
[METRICS] model=sonnet | input=n/a | output=n/a | duration={seconds}s

METRICS line: write once before sending completion report to TL.

---

## QA Per-Task Log — `logs/qa-task-{N}.log.md`

[LOG] task={task-id} | event=review-start
[CHECK] spec-compliance={PASS|FAIL} | {specifics}
[CHECK] code-quality={PASS|FAIL} | {specifics}
[CHECK] three-way={PASS|FAIL} | req-vs-test={aligned|gap} | test-vs-code={aligned|gap} | req-vs-code={aligned|gap}
[CHECK] standards-compliance={PASS|FAIL|SKIPPED} | {specifics or "no standards files"}
[CHECK] file-scope={PASS|FAIL} | {specifics}
[RESULT] {PASS|FAIL}

---

## QA Global Log — `logs/qa-global.log.md`

[LOG] event=global-review-start | files={count}
[CHECK] cross-task-consistency={PASS|FAIL} | {findings}
[CHECK] contract-compliance={PASS|FAIL} | {findings}
[CHECK] completeness={PASS|FAIL} | gaps={list or none}
[CHECK] integration={PASS|FAIL} | {findings}
[CHECK] standards-consistency={PASS|FAIL|SKIPPED} | {findings or "no standards files"}
[RESULT] {PASS|FAIL} | issues={count}
