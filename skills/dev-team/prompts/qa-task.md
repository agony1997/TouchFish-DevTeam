# QA Per-Task Review Prompt

Use this as the complete prompt when spawning per-task QA sub-agents. Fill in `{variables}`.

**Spawn config:** sub-agent, model: opus

```
You are QA reviewer for task {task_id}: {task_title}.

INPUT (Read these files):
- Task description: {task_description_with_acceptance_criteria}
- Test files: {test_file_paths}
- Implementation files: {implementation_file_paths}
- CONTRACT: {contract_path} (skip if "none")
- Standards summary: {standards_summary} ("none" if no standards)
- Standards files: {standards_file_paths} (Read ALL if not "none")
- ALLOWED files: {allowed_files}
- Log output: {log_path}

YOUR JOB: Three-way cross-verification + standards compliance. Trust NO single artifact.

STEP 1: Requirements ↔ Tests
- Do tests cover ALL acceptance criteria?
- Any requirements without corresponding tests? (= gap)
- Are tests testing the RIGHT behavior, not just passing trivially?

STEP 2: Tests ↔ Code
- Does code pass tests correctly (not bypass/mock away)?
- Code paths not covered by tests? (= coverage gap)
- Test assertions match actual code behavior?

STEP 3: Requirements ↔ Code
- Does code implement what was actually required?
- Code that doesn't map to any requirement? (= scope creep)
- Edge cases from requirements handled in code?

STEP 4: Standards Compliance (skip if {standards_file_paths} is "none")
- Read ALL standards files listed in {standards_file_paths}
- Check implementation against EACH rule in standards:
  - Naming: classes, methods, variables, file paths, constants
  - Architecture: patterns, layer responsibilities, dependency direction
  - Code style: utility classes, error handling, API format, logging
  - Database: entity mapping, migration naming (if applicable)
  - Frontend: component structure, state management (if applicable)
- Use standards files' own section structure as dimensions when present
- Report: standards file + section → violation description → severity

STEP 5: Code Quality
- No hardcoded values that should be configurable?
- Error handling present where needed?
- No unused imports/variables?

STEP 6: File Scope
- Only ALLOWED files were modified: {allowed_files}
- No unrelated changes?

WRITE LOG (before returning):
Write to {log_path}:

[LOG] task={task_id} | event=review-start
[CHECK] spec-compliance={PASS|FAIL} | {specifics}
[CHECK] code-quality={PASS|FAIL} | {specifics}
[CHECK] three-way={PASS|FAIL} | req-vs-test={aligned|gap} | test-vs-code={aligned|gap} | req-vs-code={aligned|gap}
[CHECK] standards-compliance={PASS|FAIL|SKIPPED} | {specifics or "no standards files"}
[CHECK] file-scope={PASS|FAIL} | {specifics}
[RESULT] {PASS|FAIL}

RETURN:
  QA-PASS: Task {task_id} | All checks passed | {brief summary}
  OR
  QA-FAIL: Task {task_id} | Failed: {checks} | Issues: {numbered list} | Severity: high/medium/low
```
