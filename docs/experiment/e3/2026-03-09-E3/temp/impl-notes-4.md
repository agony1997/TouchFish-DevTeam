# T4: M2 — Task Entity + Repository — Implementation Notes

## What the tests expect

### Task Entity (`com.taskflow.entity.Task`)
- Must be a JPA `@Entity` with `@Table(name = "tasks")` (or similar)
- Fields:
  - `id` — `Long`, `@Id`, `@GeneratedValue(strategy = GenerationType.IDENTITY)`
  - `title` — `String`, `@Column(nullable = false, length = 200)`
  - `description` — `String`, `@Column(length = 5000)`
  - `priority` — `String`, default `"MEDIUM"` (field initializer)
  - `status` — `TaskStatus` enum, default `TaskStatus.TODO` (field initializer)
  - `createdBy` — `Long`
  - `createdAt` — `LocalDateTime`, consider `@PrePersist` for auto-set
  - `updatedAt` — `LocalDateTime`, consider `@PreUpdate` for auto-set
  - `deleted` — `boolean`, default `false` (field initializer)

### TaskStatus Enum (`com.taskflow.entity.TaskStatus`)
- Already fully implemented in stub: `TODO`, `IN_PROGRESS`, `DONE`, `CLOSED`
- No changes needed

### TaskRepository (`com.taskflow.repository.TaskRepository`)
- Already has correct method signatures in stub
- Extends `JpaRepository<Task, Long>`
- Methods:
  - `findByDeletedFalse(Pageable)` — returns only non-deleted tasks
  - `findByDeletedFalseAndTitleContainingIgnoreCase(String, Pageable)` — keyword search
  - `findByDeletedFalseAndPriority(String, Pageable)` — priority filter
  - `findByDeletedFalseAndStatus(TaskStatus, Pageable)` — status filter

### TaskResponse DTO (`com.taskflow.dto.TaskResponse`)
- Already fully implemented in stub with all getters/setters
- Fields: id, title, description, priority, status, createdBy, createdAt, updatedAt
- No changes needed for DTO tests to pass

## What needs to change for tests to go GREEN
1. **Task.java**: Add JPA annotations (`@Entity`, `@Id`, `@GeneratedValue`, `@Column`), add field initializers for defaults (`priority = "MEDIUM"`, `status = TaskStatus.TODO`, `deleted = false`)
2. **TaskStatus.java**: Already complete — no changes
3. **TaskRepository.java**: Already complete with Spring Data derived query methods — no changes needed once Task is a proper JPA entity
4. **TaskResponse.java**: Already complete — no changes

## Key test categories
- **Entity unit tests** (9 tests): field accessors, defaults, boundary values
- **Enum tests** (5 tests): value count and names
- **Repository integration tests** (10 tests): CRUD, pagination, filtering, soft-delete exclusion
- **DTO unit tests** (9 tests): field accessors, contract compliance

## Test compilation status
- All 4 T4 test files compile successfully
- Cannot run tests yet due to other agents' test files having unresolved dependencies (AuthDtoValidationTest, JwtUtilTest, UserRepositoryTest, JwtAuthenticationFilterTest)
- Expected behavior: entity/enum/DTO unit tests would PASS with current stubs (stubs have getters/setters); repository tests would FAIL because Task is not a JPA entity
