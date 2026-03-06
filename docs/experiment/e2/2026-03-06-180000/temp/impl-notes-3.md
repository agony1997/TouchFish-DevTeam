# Implementation Notes for T3: Task Entity + Repository + DTOs

## Entity Pattern (from User.java)
- Use `@PrePersist` for auto-setting `createdAt` (same as User entity)
- No Lombok — manual getters/setters required
- Jakarta validation annotations on entity fields

## Task Entity Requirements
- Enums `Priority` (HIGH, MEDIUM, LOW) and `TaskStatus` (TODO, IN_PROGRESS, DONE, CLOSED) can be inner enums of Task
- Field defaults: `priority = MEDIUM`, `status = TODO`, `deleted = false`
- `@Size(min=1, max=200)` + `@NotBlank` on title
- `@Size(max=5000)` on description (nullable)
- `@Column(length=5000)` for description to match DB column
- `@Enumerated(EnumType.STRING)` for priority and status enums
- `updatedAt` is nullable — no auto-set in @PrePersist, only on update

## Repository Query Design
- Tests expect a method named `findByFilters(String keyword, Priority priority, TaskStatus status, Pageable pageable)` returning `Page<Task>`
- This method must filter `deleted = false` always
- keyword filter: `LOWER(t.title) LIKE LOWER(CONCAT('%', :keyword, '%'))` when keyword is not null
- priority filter: `t.priority = :priority` when priority is not null
- status filter: `t.status = :status` when status is not null
- Use `@Query` with JPQL and conditional logic via `(:keyword IS NULL OR ...)` pattern
- Also need `findByIdAndDeletedFalse(Long id)` -> `Optional<Task>` (Spring Data derived query)

## DTO Requirements
- TaskRequest: title (@NotBlank @Size(min=1,max=200)), description (@Size(max=5000) nullable), priority (nullable, enum)
- TaskResponse: all fields matching CONTRACT — id, title, description, priority, status, createdBy, createdAt, updatedAt
- TaskPageResponse: content (List<TaskResponse>), page (int), size (int), totalElements (long), totalPages (int)

## Framework Pitfalls
- @DataJpaTest uses H2 by default, ensure enum storage as STRING not ORDINAL
- @PrePersist only fires on entityManager.persist(), not on new Task() — tests verify null before persist
- The `findByFilters` JPQL query must handle null parameters correctly; use `(:param IS NULL OR t.field = :param)` pattern
- For enum parameters in JPQL with the IS NULL pattern, ensure the query works with H2
