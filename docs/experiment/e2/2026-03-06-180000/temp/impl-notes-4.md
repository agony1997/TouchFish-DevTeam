# Implementation Notes for T4: Task Service + Controller

## TaskService expected signatures
```java
@Service
public class TaskService {
    // inject TaskRepository
    TaskResponse createTask(Long userId, TaskRequest request);
    TaskResponse getTask(Long id);
    TaskResponse updateTask(Long id, TaskRequest request);
    void deleteTask(Long id);
    TaskPageResponse listTasks(String keyword, Task.Priority priority, Task.TaskStatus status, Pageable pageable);
}
```

## Key implementation details

### createTask
- Set `createdBy` from the `userId` parameter (extracted from SecurityContext in controller)
- If `request.getPriority()` is null, the Task entity defaults to MEDIUM (no explicit handling needed IF the entity default works; BUT if the request priority is set, copy it)
- Status is always TODO for new tasks
- Save and return mapped TaskResponse

### getTask
- Use `taskRepository.findByIdAndDeletedFalse(id)`
- Throw exception (mapped to 404) if empty

### updateTask
- Find with `findByIdAndDeletedFalse`, throw 404 if not found
- Update title, description, priority from request
- Set `updatedAt = LocalDateTime.now()`
- Do NOT change status or createdBy
- Save and return

### deleteTask
- Find with `findByIdAndDeletedFalse`, throw 404 if not found
- Set `deleted = true`, save
- Controller returns 204 (no body)

### listTasks
- Delegate to `taskRepository.findByFilters(keyword, priority, status, pageable)`
- Map Page<Task> to TaskPageResponse (content, page, size, totalElements, totalPages)

## TaskController expected structure
```java
@RestController
@RequestMapping("/api/tasks")
public class TaskController {
    // inject TaskService

    @PostMapping -> 201, extract userId from SecurityContext, @Valid @RequestBody TaskRequest
    @GetMapping("/{id}") -> 200
    @PutMapping("/{id}") -> 200, @Valid @RequestBody TaskRequest
    @DeleteMapping("/{id}") -> 204
    @GetMapping -> 200, @RequestParam(required=false) keyword/priority/status, @PageableDefault(size=20, sort="createdAt", direction=DESC)
}
```

## Extracting userId from SecurityContext
```java
Authentication auth = SecurityContextHolder.getContext().getAuthentication();
Long userId = (Long) auth.getPrincipal();
```
The JwtAuthenticationFilter sets `UsernamePasswordAuthenticationToken(userId, null, emptyList())` where principal is `Long userId`.

## Exception handling pattern
Follow AuthController pattern:
- Custom exception class (e.g., TaskNotFoundException) extending RuntimeException
- @ExceptionHandler in controller mapping to 404
- @ExceptionHandler for MethodArgumentNotValidException mapping to 400 with `{"message": "..."}`

## Task -> TaskResponse mapping
Map all fields: id, title, description, priority, status, createdBy, createdAt, updatedAt. This should be a private helper method `toResponse(Task)`.

## Framework pitfalls
- @Valid is REQUIRED on @RequestBody for validation to trigger
- @PageableDefault needs explicit sort direction (Sort.Direction.DESC)
- SecurityContext principal is Long (not String) - cast accordingly
- Error format must be `{"message": "..."}` consistently
- 204 No Content should return `ResponseEntity.noContent().build()` with no body
- Soft delete: never call `repository.delete()`, always set `deleted = true` and save
