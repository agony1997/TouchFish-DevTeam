# Implementation Notes — T1: User Entity + Auth DTOs

## Framework Gotchas

- **SecurityConfig blocks compilation**: `SecurityConfig.java` imports `JwtAuthenticationFilter` (T2). Until T2 provides a stub or the real class, the project won't compile. Worker for T1 may need to coordinate with T2 or create a minimal stub in `com.taskflow.security.JwtAuthenticationFilter` to unblock compilation.
- **@DataJpaTest for UserRepositoryTest**: Requires `User` to be a proper `@Entity` with `@Id @GeneratedValue`. The test uses `TestEntityManager.persistAndFlush()` which needs JPA metadata.
- **createdAt auto-set**: `UserRepositoryTest.shouldSetCreatedAtAutomaticallyOnPersist` expects `createdAt` to be set on persist. Use `@PrePersist` callback or `@CreationTimestamp`.

## Per-Test Hints

- **UserRepositoryTest.shouldEnforceUsernameUniquenessConstraint**: Requires `@Column(unique = true)` on `username` field. The test catches any `Exception` from `persistAndFlush` — the actual exception will be `PersistenceException` wrapping a constraint violation.
- **AuthDtoValidationTest — username @Pattern**: The acceptance criteria specifies `@Pattern(regexp = "^[a-zA-Z0-9_]{3,50}$")`. This single annotation handles both the character set and the length constraint. Do NOT use a separate `@Size` for username — the pattern already enforces 3-50 chars.
- **LoginRequest**: No validation annotations needed beyond basic getter/setter. The controller (T3) handles auth logic.
- **LoginResponse**: Simple POJO with `token` (String) and `expiresIn` (long). Consider using constructor or all-args for convenience but getter/setter is sufficient for tests.

## Build Note

The project currently cannot compile due to `SecurityConfig` referencing `JwtAuthenticationFilter` from T2. T1 worker should either:
1. Wait for T2 stub, or
2. Create a minimal no-op `JwtAuthenticationFilter` stub in `com.taskflow.security` package (but this is T2's file count)
