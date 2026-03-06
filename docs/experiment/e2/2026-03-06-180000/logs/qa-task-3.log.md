# QA Log: T3 - M2 Task Entity + Repository + DTOs

## Test Execution

```
Tests run: 60, Failures: 0, Errors: 0, Skipped: 0
BUILD SUCCESS (8.342s)
```

- TaskTest: 19 tests PASSED
- TaskRepositoryTest: 20 tests PASSED
- TaskRequestTest: 13 tests PASSED
- TaskResponseTest: 4 tests PASSED
- TaskPageResponseTest: 3 tests PASSED

## Three-Way Verification (CONTRACT vs Implementation vs Tests)

### Task Entity

| Check | CONTRACT | Implementation | Tests | Status |
|-------|----------|---------------|-------|--------|
| M2-AC1: Task entity fields (id, title, description, priority, status, createdBy, createdAt, updatedAt, deleted) | TaskResponse type defines all fields | Task.java has all fields with correct types, JPA annotations | TaskTest covers all field access + defaults | PASS |
| M2-AC2: Priority enum (HIGH, MEDIUM, LOW) | enum(HIGH\|MEDIUM\|LOW) | Task.Priority with 3 values, @Enumerated(STRING) | priorityEnumShouldHaveThreeValues, priorityEnumShouldContainHighMediumLow | PASS |
| M2-AC2: Status enum (TODO, IN_PROGRESS, DONE, CLOSED) | enum(TODO\|IN_PROGRESS\|DONE\|CLOSED) | Task.TaskStatus with 4 values, @Enumerated(STRING) | taskStatusEnumShouldHaveFourValues, taskStatusEnumShouldContainAllStates | PASS |
| M2-AC3: Title 1-200 chars, required | title:string(1-200,required) | @NotBlank @Size(min=1,max=200) | boundary tests at 1, 200, 201 chars + null/empty rejection | PASS |
| M2-AC15: Default priority MEDIUM | default:MEDIUM | `priority = Priority.MEDIUM` | shouldDefaultPriorityToMedium (entity + repo) | PASS |
| M2-AC16: Description max 5000 | max5000 | @Size(max=5000) @Column(length=5000) | boundary tests at 5000/5001 chars | PASS |
| Soft delete flag | DELETE endpoint returns 204 | `deleted = false` default, boolean field | shouldHaveDeletedFalseByDefault, shouldSetAndGetDeletedField | PASS |
| createdAt auto-set | createdAt:ISO8601 | @PrePersist onCreate() sets LocalDateTime.now() | shouldSetCreatedAtAutomatically | PASS |
| Auto-generated ID | id:long | @GeneratedValue(IDENTITY) | shouldAutoGenerateId | PASS |

### TaskRepository

| Check | CONTRACT | Implementation | Tests | Status |
|-------|----------|---------------|-------|--------|
| M2-AC8: Keyword search (case-insensitive, partial match) | keyword:string? | JPQL LOWER(LIKE) query | searchTasks_shouldFilterByKeywordCaseInsensitive, keywordShouldMatchPartialTitle | PASS |
| M2-AC10: Pagination support | page:int(0)&size:int(20) | Pageable parameter in findByFilters | searchTasks_shouldSupportPagination (7 items, page size 3) | PASS |
| M2-AC11: Priority filter | priority:enum? | :priority param in JPQL | searchTasks_shouldFilterByPriority | PASS |
| M2-AC12: Status filter | status:enum? | :status param in JPQL | searchTasks_shouldFilterByStatus | PASS |
| findByIdAndDeletedFalse | Soft delete exclusion | Spring Data derived query | 3 tests: active found, deleted excluded, nonexistent empty | PASS |
| Combined filters | All params optional | JPQL AND conditions with null checks | shouldCombineKeywordAndPriorityFilters, shouldCombineAllFilters | PASS |
| Sort support | sort:string(createdAt,desc) | Pageable accepts Sort | searchTasks_shouldSupportSorting (DESC createdAt) | PASS |
| Soft delete exclusion in search | Implicit | `t.deleted = false` in JPQL WHERE | searchTasks_shouldExcludeSoftDeletedTasks | PASS |

### DTOs

| Check | CONTRACT | Implementation | Tests | Status |
|-------|----------|---------------|-------|--------|
| TaskRequest fields | title(required), description?, priority? | 3 fields with validation annotations matching entity | TaskRequestTest: 13 tests covering all fields + validation | PASS |
| TaskResponse fields | id, title, description?, priority, status, createdBy, createdAt, updatedAt? | 8 fields matching CONTRACT type definition | shouldMatchContractFields verifies all fields | PASS |
| TaskPageResponse fields | content[], page, size, totalElements, totalPages | 5 fields matching CONTRACT type definition | shouldMatchContractStructure verifies all fields + types | PASS |

## Code Quality Review

### Entity (Task.java)
- JPA annotations correct: @Entity, @Table, @Id, @GeneratedValue(IDENTITY), @Enumerated(STRING), @Column
- Validation annotations correct: @NotBlank, @Size on title and description
- @PrePersist for createdAt auto-population
- @Column(updatable=false) on createdAt prevents accidental overwrites
- @Column(length=5000) on description matches DB schema to validation
- Clean getter/setter pattern, no unnecessary code

### Repository (TaskRepository.java)
- Extends JpaRepository correctly
- findByIdAndDeletedFalse: Spring Data derived query, clean
- findByFilters: JPQL with parameterized null-safe filters, no SQL injection risk
- LOWER() on both sides for case-insensitive search

### DTOs
- TaskRequest: validation mirrors entity constraints (good consistency)
- TaskResponse: all CONTRACT fields present, no `deleted` field exposed (correct - internal only)
- TaskPageResponse: standard pagination wrapper, matches CONTRACT exactly

### Potential Observations (non-blocking)
- Task entity has no @PreUpdate for updatedAt - this will likely be handled by TaskService (not in scope for T3)
- No fromEntity() / toEntity() conversion methods in DTOs - expected to be in Service layer (T4 scope)

## File Scope Check

ALLOWED files:
- [x] entity/Task.java
- [x] repository/TaskRepository.java
- [x] dto/TaskRequest.java
- [x] dto/TaskResponse.java
- [x] dto/TaskPageResponse.java

No files outside the allowed scope were modified.

## Verdict

**QA-PASS**

All 60 tests pass. Implementation matches CONTRACT on all checked ACs (M2-AC1,2,3,8,10,11,12,15,16). Tests provide thorough boundary coverage and contract verification. Code quality is clean with no security issues. Files are within allowed scope.
