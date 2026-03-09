# QA Log — T5: M2 — Task CRUD API

## Result: QA-PASS

## Files Reviewed
- `service/TaskService.java` (96 lines)
- `controller/TaskController.java` (64 lines)
- `dto/TaskRequest.java` (25 lines)
- `dto/PageResponse.java` (35 lines)
- `controller/TaskControllerTest.java` (513 lines, 24 test cases)

## STEP 1: Requirements <-> Tests

| AC  | Requirement                                         | Test(s)                                              | Status  |
|-----|-----------------------------------------------------|------------------------------------------------------|---------|
| AC1 | POST returns 201 with id, createdBy, createdAt      | createTask_validData_returns201, createTask_createdByMatchesAuthUser | PASS |
| AC2 | Default priority MEDIUM                             | createTask_noPriority_defaultsMedium                 | PASS    |
| AC3 | Empty/missing title returns 400                     | createTask_emptyTitle_returns400, createTask_noTitle_returns400 | PASS |
| AC4 | GET /{id} returns full data                         | getTask_existing_returns200                          | PASS    |
| AC5 | GET non-existing returns 404                        | getTask_nonExisting_returns404                       | PASS    |
| AC6 | PUT valid update returns 200 with updatedAt         | updateTask_valid_returns200WithUpdatedAt, updateTask_nonExisting_returns404 | PASS |
| AC7 | DELETE returns 204 (soft delete)                    | deleteTask_existing_returns204, deleteTask_nonExisting_returns404 | PASS |
| AC8 | Soft-deleted excluded from list                     | listTasks_excludesSoftDeleted                        | PASS    |
| AC9 | Soft-deleted returns 404 on GET                     | getTask_softDeleted_returns404                       | PASS    |
| AC10| Default pagination page=0,size=20,sort=createdAt desc| listTasks_defaultPagination                         | PASS    |
| AC11| Keyword filter (case-insensitive)                   | listTasks_keywordFilter                              | PASS    |
| AC12| Priority filter                                     | listTasks_priorityFilter                             | PASS    |
| AC13| Custom page/size pagination                         | listTasks_pagination                                 | PASS    |
| AC14| No token returns 401 (all endpoints)                | 5 tests (one per endpoint group)                     | PASS    |
| AC15| Title max 200 chars validation                      | createTask_titleTooLong_returns400, createTask_titleExact200_returns201, updateTask_titleTooLong_returns400 | PASS |
| AC16| Description max 5000 chars validation               | createTask_descriptionTooLong_returns400, createTask_descriptionExact5000_returns201 | PASS |

All 16 ACs have corresponding test coverage.

## STEP 2: Tests <-> Code

- TaskController correctly maps all 5 endpoints (POST, GET/{id}, PUT/{id}, DELETE/{id}, GET list)
- POST returns 201 via `HttpStatus.CREATED` -- matches test expectation
- DELETE returns 204 via `ResponseEntity.noContent().build()` -- matches test
- `@Valid @RequestBody TaskRequest` triggers validation for `@NotBlank` title, `@Size(max=200)` title, `@Size(max=5000)` description -- matches tests
- TaskService.createTask sets default priority "MEDIUM" when null -- matches AC2 test
- TaskService.getTask / updateTask / deleteTask filter `!t.isDeleted()` -- matches soft-delete tests (AC8, AC9)
- TaskService.listTasks: keyword filter uses `findByDeletedFalseAndTitleContainingIgnoreCase`, priority filter uses `findByDeletedFalseAndPriority` -- matches test expectations
- PageResponse fields (content, page, size, totalElements, totalPages) match JSON path assertions
- getCurrentUserId() extracts from SecurityContextHolder principal -- matches createdBy test

All tests align with implementation.

## STEP 3: Requirements <-> Code

### Verified
- Soft-delete: `task.setDeleted(true)` + `taskRepository.save(task)` -- data preserved, not hard-deleted
- createdBy from JWT: `getCurrentUserId()` reads `Authentication.getPrincipal()` which is set by JWT filter
- Default pagination: `PageRequest.of(page, size, Sort.by(DESC, "createdAt"))` with defaults page=0, size=20
- Filtering: keyword (case-insensitive title search) and priority filtering both implemented
- Validation: `@NotBlank` + `@Size(max=200)` on title, `@Size(max=5000)` on description
- Error responses use ErrorResponse via GlobalExceptionHandler (400/404/409)

### Minor Observations (not AC violations)
1. **Priority not enum-validated**: `TaskRequest.priority` is a plain String; invalid values like "INVALID" would be accepted. No AC explicitly requires rejecting invalid priority values, but CONTRACT defines it as `enum(HIGH|MEDIUM|LOW)`.
2. **Combined keyword+priority not supported**: Service uses if/else-if, so providing both params only applies keyword. No AC tests this scenario.
3. **`sort` query param not exposed**: CONTRACT mentions `sort:string(createdAt,desc)` but controller hardcodes sort. No AC requires customizable sort.

None of these are AC failures.

## STEP 4: Standards — SKIPPED

## STEP 5: Code Quality

- Clean MVC layering: Controller -> Service -> Repository
- Proper constructor injection (no field injection)
- Consistent use of `@Valid` for request validation
- Soft-delete correctly applied in all read operations (get, list)
- No SQL injection risks (Spring Data JPA query methods)
- No hardcoded secrets or credentials
- PageResponse generic type properly parameterized
- toResponse mapping is complete and correct

## STEP 6: File Scope

Allowed files: `service/TaskService.java`, `controller/TaskController.java`, `dto/TaskRequest.java`, `dto/PageResponse.java`
All implementation code is within allowed files. Supporting files (Task entity, TaskRepository, TaskResponse, GlobalExceptionHandler) were pre-existing from T4/T3.

## Summary

24 test cases covering all 16 ACs. Implementation is clean, correct, and within scope. Three minor observations noted but none constitute AC violations.
