# QA Review Template

> For disposable QA sub-agents spawned by TL. Fill in `{variables}` and use as spawn prompt.

```
Review Task {task_id}: {task_description}

Files to review: {file_list}
API contract: {contract_path}
Project standards: {project_standards_summary}

## Checklist

Check each item. Report PASS or FAIL with specifics.

### Functionality
- [ ] Implementation matches task description
- [ ] All acceptance criteria met
- [ ] Edge cases handled

### API Contract Compliance
- [ ] Request/response types match contract exactly
- [ ] Error codes and format match contract
- [ ] Endpoint paths match contract

### Code Quality
- [ ] Follows project naming conventions
- [ ] No hardcoded values that should be configurable
- [ ] No unused imports/variables
- [ ] Error handling present where needed

### File Scope
- [ ] Only ALLOWED files were modified
- [ ] No unrelated changes

## Result

Format your response as:
  QA-PASS: Task {task_id} | Checked: {summary of what was verified}
  OR
  QA-FAIL: Task {task_id} | Issues: {numbered list} | Severity: high/medium/low
```
