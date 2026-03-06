# QA Log: T1 — M1-Foundation (User Entity + JWT Infrastructure)

**Reviewer:** QA Agent
**Date:** 2026-03-06
**Verdict:** QA-FAIL

---

## STEP 1: Requirements <-> Tests

### Spec Acceptance Criteria Coverage

The M1 spec defines AC1-AC14 covering registration, login, and /api/auth/me endpoints. Task T1 scope is narrower: only the **foundation** layer (User entity, UserRepository, JwtTokenProvider, JwtAuthFilter, SecurityConfig). The controller/service layer that handles AC1-AC14 HTTP endpoints is NOT part of T1.

| Foundation Requirement | Test Coverage | Verdict |
|---|---|---|
| User entity with JPA (id, username, password, displayName, createdAt) | UserEntityTest: persistence, auto-id, auto-createdAt, null constraints, unique username | PASS |
| UserRepository.findByUsername() | UserRepositoryTest: found, not found, null, multiple users | PASS |
| JwtTokenProvider generates JWT with userId claim | JwtTokenProviderTest: generation, format, different users | PASS |
| JwtTokenProvider validates JWT (valid/expired/malformed/null/tampered/wrong-secret) | JwtTokenProviderTest: 7 validation scenarios | PASS |
| JwtTokenProvider extracts userId from token | JwtTokenProviderTest: 2 parsing tests | PASS |
| JwtAuthFilter intercepts requests, extracts Bearer token, sets SecurityContext | JwtAuthFilterTest: valid token sets auth, no header, no Bearer prefix, invalid token, empty bearer | PASS |
| JwtAuthFilter always continues filter chain | JwtAuthFilterTest: chain continuation, no getUserIdFromToken on invalid | PASS |
| SecurityConfig: /api/auth/** permitAll | SecurityConfigIntegrationTest: register, login permit tests | PASS |
| SecurityConfig: other endpoints require auth | SecurityConfigIntegrationTest: tasks endpoints | FAIL (see Step 2) |
| SecurityConfig: stateless session | SecurityConfigIntegrationTest: no session created | PASS |
| SecurityConfig: JWT filter integrated | SecurityConfigIntegrationTest: invalid/expired token tests | FAIL (see Step 2) |
| BCryptPasswordEncoder bean exposed | SecurityConfig provides @Bean; no dedicated test | MINOR GAP |

**Step 1 Verdict: Tests cover the foundation requirements well, but 4 tests fail at runtime (see Step 2).**

---

## STEP 2: Tests <-> Code

### Test Execution Results

**Command:** `mvn test -Dtest=UserEntityTest,UserRepositoryTest,JwtTokenProviderTest,JwtAuthFilterTest,SecurityConfigIntegrationTest`

**Result: 40 tests run, 4 FAILURES, 0 errors**

#### Failing Tests (all in SecurityConfigIntegrationTest)

1. `ProtectedEndpoints.shouldReturn401ForTasksWithoutToken` — Expected 401, got **403**
2. `ProtectedEndpoints.shouldReturn401ForPostTasksWithoutToken` — Expected 401, got **403**
3. `JwtFilterIntegration.shouldReturn401WithInvalidToken` — Expected 401, got **403**
4. `JwtFilterIntegration.shouldReturn401WithExpiredToken` — Expected 401, got **403**

#### Root Cause

`SecurityConfig` does not configure an `AuthenticationEntryPoint`. Spring Security's default behavior returns **403 Forbidden** (not 401 Unauthorized) when an unauthenticated request hits a protected endpoint. To get 401, the config must add:

```java
.exceptionHandling(ex -> ex
    .authenticationEntryPoint((request, response, authException) ->
        response.sendError(HttpServletResponse.SC_UNAUTHORIZED, "Unauthorized"))
)
```

This is a **code defect**, not a test defect. The tests correctly expect 401 per the spec (AC8, AC9, AC10).

#### Passing Tests (36/40)

- UserEntityTest: 6/6 PASS
- UserRepositoryTest: 4/4 PASS
- JwtTokenProviderTest: 11/11 PASS
- JwtAuthFilterTest: 8/8 PASS
- SecurityConfigIntegrationTest: 7/11 PASS (4 FAIL as above)

**Step 2 Verdict: FAIL — 4 integration tests fail due to missing AuthenticationEntryPoint in SecurityConfig.**

---

## STEP 3: Requirements <-> Code

### User.java

| Spec Requirement | Implementation | Verdict |
|---|---|---|
| Fields: id, username, password, displayName | All present | PASS |
| id: auto-generated | `@GeneratedValue(strategy = IDENTITY)` | PASS |
| username: unique | `@Column(unique = true)` | PASS |
| username: not null | `@Column(nullable = false)` | PASS |
| password: not null | `@Column(nullable = false)` | PASS |
| displayName: not null | `@Column(nullable = false)` | PASS |
| createdAt: auto-set | `@PrePersist` sets `LocalDateTime.now()` | PASS |
| BR1: username length 3-50 | NO column length constraint, NO validation annotation | NOTE |
| BR2: password min 8 chars | NO validation annotation | NOTE |
| BR3: displayName 1-100 chars | NO validation annotation | NOTE |

**NOTE:** The BR1-BR3 validation constraints are not on the entity. This is acceptable for T1 scope since validation will be handled at the DTO/controller level (not in T1's ALLOWED files). However, adding `@Column(length = 50)` on username and `@Column(length = 100)` on displayName would provide a database-level safety net.

### UserRepository.java

| Spec Requirement | Implementation | Verdict |
|---|---|---|
| Find user by username | `findByUsername(String)` returning `Optional<User>` | PASS |
| BR6: username uniqueness check | No `existsByUsername()` method | NOTE |

**NOTE:** `existsByUsername()` would be more efficient for duplicate-check, but `findByUsername()` can serve the same purpose. Acceptable.

### JwtTokenProvider.java

| Spec Requirement | Implementation | Verdict |
|---|---|---|
| Generate JWT with userId as subject | `subject(String.valueOf(userId))` | PASS |
| Set issuedAt and expiration | Both set | PASS |
| BR5: 24h expiration (86400000ms) | Configurable via `app.jwt.expiration`, default 86400000 | PASS |
| Validate token (true/false) | Catches all exceptions, returns boolean | PASS |
| Extract userId from token | Parses subject, returns Long | PASS |
| Use HMAC-SHA key | `Keys.hmacShaKeyFor()` | PASS |

### JwtAuthFilter.java

| Spec Requirement | Implementation | Verdict |
|---|---|---|
| Extract Bearer token from Authorization header | Checks prefix "Bearer ", extracts substring(7) | PASS |
| Validate token | Delegates to `tokenProvider.validateToken()` | PASS |
| Set SecurityContext on valid token | Creates `UsernamePasswordAuthenticationToken` with userId as principal | PASS |
| Continue filter chain always | `filterChain.doFilter()` called unconditionally | PASS |
| Handle missing/empty token | Returns null, no auth set | PASS |

### SecurityConfig.java

| Spec Requirement | Implementation | Verdict |
|---|---|---|
| /api/auth/** permitAll | `requestMatchers("/api/auth/**").permitAll()` | PASS |
| Other endpoints require auth | `.anyRequest().authenticated()` | PASS |
| Stateless session | `SessionCreationPolicy.STATELESS` | PASS |
| CSRF disabled (JWT-based, stateless) | `csrf.disable()` | PASS |
| JwtAuthFilter before UsernamePasswordAuthenticationFilter | `.addFilterBefore(...)` | PASS |
| BCryptPasswordEncoder bean | `@Bean public PasswordEncoder` | PASS |
| Return 401 for unauthenticated requests | **MISSING** AuthenticationEntryPoint | **FAIL** |

**Step 3 Verdict: FAIL — SecurityConfig missing AuthenticationEntryPoint, causing 403 instead of spec-required 401.**

---

## STEP 4: Standards Compliance

SKIPPED (no standards provided).

---

## STEP 5: Code Quality / Security Review

### JWT Secret Handling

- **CONCERN (MEDIUM):** The JWT secret in `application.yml` is a hardcoded dev-only string: `"taskflow-jwt-secret-key-for-development-only-change-in-production"`. This is acceptable for development/testing, but the secret length (67 chars = 536 bits) is sufficient for HS256. No secrets are committed that would be production-dangerous.
- **OK:** Secret is injected via `@Value`, making it configurable per environment.

### Password Field Exposure

- **CONCERN (LOW):** `User.java` has no `@JsonIgnore` on the `password` field. If the entity is accidentally serialized to JSON (e.g., returned directly from a controller), the bcrypt hash would be exposed. Since T1 does not include controllers and the spec uses separate response DTOs (UserResponse without password), this is acceptable at this layer, but a `@JsonIgnore` annotation would be a good defensive measure.

### Filter Chain Ordering

- **OK:** `JwtAuthFilter` is correctly placed before `UsernamePasswordAuthenticationFilter` using `addFilterBefore()`.

### JJWT API Usage

- **OK:** Uses JJWT 0.12.5 modern builder API (`Jwts.builder()`, `.subject()`, `.issuedAt()`, `.expiration()`, `.signWith(key)`, `.verifyWith(key)`, `.parseSignedClaims()`). No deprecated method calls.

### SecurityContext Cleanup

- **MINOR:** `JwtAuthFilter` does not clear the SecurityContext after the filter chain completes. For `OncePerRequestFilter` with stateless sessions this is generally handled by Spring, but explicit cleanup would be safer.

### Missing application-test.yml

- **NOTE:** Tests use `@ActiveProfiles("test")` but no `application-test.yml` exists. Tests fall back to default profile properties, which works but is fragile.

---

## STEP 6: File Scope Check

**ALLOWED files:** User.java, UserRepository.java, JwtTokenProvider.java, JwtAuthFilter.java, SecurityConfig.java

| File | Status | Scope |
|---|---|---|
| User.java | New (untracked) | ALLOWED |
| UserRepository.java | New (untracked) | ALLOWED |
| JwtTokenProvider.java | New (untracked) | ALLOWED |
| JwtAuthFilter.java | New (untracked) | ALLOWED |
| SecurityConfig.java | Modified (existed in skeleton) | ALLOWED |

**Other untracked files detected (from other tasks, not T1):**
- Task.java, TaskStatus.java, TaskRepository.java, CreateTaskRequest.java, TaskResponse.java

These are NOT part of T1's deliverables and were created by other task workers. T1 did not touch any files outside its allowed scope.

**Step 6 Verdict: PASS — T1 only touched allowed files.**

---

## Defect Summary

### BLOCKING (must fix)

| # | Severity | File | Issue |
|---|---|---|---|
| D1 | HIGH | SecurityConfig.java | Missing `AuthenticationEntryPoint` configuration. Spring Security returns 403 instead of 401 for unauthenticated requests. Spec AC8/AC9/AC10 all require 401. 4 integration tests fail. Fix: add `.exceptionHandling(ex -> ex.authenticationEntryPoint(...))` to the filter chain. |

### NON-BLOCKING (recommendations)

| # | Severity | File | Issue |
|---|---|---|---|
| R1 | LOW | User.java | No `@JsonIgnore` on password field — risk of accidental hash exposure if entity is serialized directly. |
| R2 | LOW | User.java | No `@Column(length=50)` on username or `@Column(length=100)` on displayName — no DB-level length enforcement. |
| R3 | LOW | (config) | No `application-test.yml` — tests rely on default profile fallback. |

---

## Verdict

**QA-FAIL**

**Reason:** 4 out of 40 tests fail. SecurityConfig is missing an `AuthenticationEntryPoint`, causing unauthenticated requests to receive HTTP 403 instead of the spec-required HTTP 401. This is a blocking defect that must be fixed before the task can pass QA.

**Required fix:** Add exception handling to `SecurityConfig.securityFilterChain()`:
```java
.exceptionHandling(ex -> ex
    .authenticationEntryPoint((request, response, authException) ->
        response.sendError(HttpServletResponse.SC_UNAUTHORIZED, "Unauthorized"))
)
```
