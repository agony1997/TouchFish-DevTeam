# QA Log — T4: M2 — Task Entity + Repository

## Result: QA-PASS

## Files Reviewed
- `entity/Task.java` — Task JPA entity
- `entity/TaskStatus.java` — TaskStatus enum
- `repository/TaskRepository.java` — TaskRepository interface
- `dto/TaskResponse.java` — TaskResponse DTO

## STEP 1: Requirements <-> Tests

### Requirements (from AC + CONTRACT):
1. Task with id/title/description/priority/status/createdBy/createdAt/updatedAt/deleted
2. TaskStatus enum: TODO, IN_PROGRESS, DONE, CLOSED
3. TaskRepository with pagination + search + filter
4. TaskResponse DTO with fields: id, title, description, priority, status, createdBy, createdAt, updatedAt

### Test Coverage:
| Requirement | Test(s) | Covered? |
|---|---|---|
| Task fields (id, title, desc, priority, status, createdBy, timestamps, deleted) | TaskTest: shouldCreateTaskWithAllRequiredFields, shouldHaveNullIdBeforePersistence, shouldHaveTimestampFields, shouldSetAndGetDeletedFlag | YES |
| Task defaults (status=TODO, priority=MEDIUM, deleted=false) | TaskTest: shouldDefaultStatusToTodo, shouldDefaultPriorityToMedium, shouldDefaultDeletedToFalse | YES |
| Task field constraints (title<=200, desc<=5000) | TaskTest: shouldAcceptTitleUpTo200Characters, shouldAcceptDescriptionUpTo5000Characters | YES |
| TaskStatus has exactly 4 values: TODO, IN_PROGRESS, DONE, CLOSED | TaskStatusTest: 5 tests (count + each value) | YES |
| Repository: pagination | TaskRepositoryTest: findByDeletedFalseShouldSupportPagination | YES |
| Repository: soft-delete filtering | TaskRepositoryTest: findByDeletedFalseShouldReturnOnlyNonDeletedTasks + 3 exclusion tests | YES |
| Repository: keyword search (case-insensitive) | TaskRepositoryTest: shouldSearchByKeywordInTitleCaseInsensitive, shouldReturnEmptyPageWhenKeywordDoesNotMatch, keywordSearchShouldExcludeDeletedTasks | YES |
| Repository: priority filter | TaskRepositoryTest: shouldFilterByPriority, priorityFilterShouldExcludeDeletedTasks | YES |
| Repository: status filter | TaskRepositoryTest: shouldFilterByStatus, statusFilterShouldExcludeDeletedTasks | YES |
| Repository: auto-generate id | TaskRepositoryTest: shouldAutoGenerateIdOnPersist | YES |
| TaskResponse DTO all fields | TaskResponseTest: 9 tests (each field + combined CONTRACT check) | YES |

**Verdict: PASS** — All requirements have corresponding tests.

## STEP 2: Tests <-> Code

- All 33 tests pass (verified via `mvn test`): 9 TaskTest, 5 TaskStatusTest, 10 TaskRepositoryTest, 9 TaskResponseTest.
- Test assertions match actual implementation behavior.
- `@DataJpaTest` correctly validates JPA entity mapping and repository queries against H2.
- `@PrePersist` / `@PreUpdate` lifecycle hooks are present in Task entity (tested indirectly via repository test persisting entities).

**Verdict: PASS**

## STEP 3: Requirements <-> Code

| CONTRACT Field | Task Entity | TaskResponse DTO | Match? |
|---|---|---|---|
| id: long | Long id, @GeneratedValue(IDENTITY) | Long id | YES |
| title: string(1-200) | @Column(nullable=false, length=200) | String title | YES |
| description: string(max5000, optional) | @Column(length=5000) | String description | YES |
| priority: enum(HIGH/MEDIUM/LOW) | String priority, default "MEDIUM" | String priority | YES |
| status: enum(TODO/IN_PROGRESS/DONE/CLOSED) | @Enumerated(STRING) TaskStatus, default TODO | String status | YES |
| createdBy: long | Long createdBy | Long createdBy | YES |
| createdAt: ISO8601 | LocalDateTime, @PrePersist | LocalDateTime | YES |
| updatedAt: ISO8601-or-null | LocalDateTime, @PreUpdate | LocalDateTime | YES |

Additional entity field `deleted` (boolean, default false) correctly supports soft-delete pattern used by repository queries.

### Observation (non-blocking):
- `priority` is stored as a raw `String` rather than an enum. This is acceptable since validation will happen at the service/controller layer, and the CONTRACT defines it as an enum in the API but doesn't mandate entity-level enforcement.

**Verdict: PASS**

## STEP 4: Standards Compliance — SKIPPED (no standards files provided)

## STEP 5: Code Quality

- **Entity**: Clean JPA entity with appropriate annotations (`@Entity`, `@Table`, `@Id`, `@GeneratedValue`, `@Column` constraints, `@Enumerated`, `@PrePersist`, `@PreUpdate`). Field defaults are set inline.
- **Repository**: Leverages Spring Data JPA derived query methods — clean and idiomatic. All 4 query methods correctly include `DeletedFalse` to enforce soft-delete.
- **DTO**: Simple POJO with getters/setters. All CONTRACT-specified fields present.
- **No security vulnerabilities** detected (no raw SQL, no injection vectors).
- **No unnecessary complexity** — straightforward implementation.

**Verdict: PASS**

## STEP 6: File Scope

ALLOWED files: entity/Task.java, entity/TaskStatus.java, repository/TaskRepository.java, dto/TaskResponse.java

Files actually modified/created for T4:
- entity/Task.java — ALLOWED
- entity/TaskStatus.java — ALLOWED
- repository/TaskRepository.java — ALLOWED
- dto/TaskResponse.java — ALLOWED

No out-of-scope files touched.

**Verdict: PASS**

## Summary

All 6 steps pass. 33/33 tests green. Implementation correctly fulfills CONTRACT and AC requirements. Code is clean, idiomatic Spring Data JPA. No scope violations.
