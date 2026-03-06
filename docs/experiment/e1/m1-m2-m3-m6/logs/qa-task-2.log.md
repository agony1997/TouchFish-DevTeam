# QA Log — Task T2: M1-Auth-API

**Reviewer**: QA Agent
**Date**: 2026-03-06
**Verdict**: **QA-PASS**

---

## STEP 1: Requirements <-> Tests

All 14 acceptance criteria from `M1-auth.md` are covered by tests:

| AC  | Requirement                                        | Test Coverage                                                                 | Verdict |
|-----|----------------------------------------------------|-------------------------------------------------------------------------------|---------|
| AC1 | POST /api/auth/register valid -> 201 w/ id,username,displayName | `AuthControllerIntegrationTest.RegisterValidData.shouldReturn201WithUserResponse` + boundary tests (min/max username, password, displayName) | PASS |
| AC2 | POST /api/auth/register duplicate username -> 409  | `RegisterDuplicateUsername.shouldReturn409WhenUsernameExists`                  | PASS |
| AC3 | Password stored as bcrypt hash                     | `PasswordBcryptHash.shouldStoreBcryptHash` (verifies $2a/b/y prefix, not plaintext, verifies with encoder) + `shouldNotVerifyDifferentPassword` | PASS |
| AC4 | POST /api/auth/login correct -> 200 + token + expiresIn | `LoginCorrectCredentials.shouldReturn200WithTokenAndExpiresIn` + `shouldReturnValidJwtToken` + `shouldReturnExpiresIn86400` | PASS |
| AC5 | POST /api/auth/login wrong password -> 401         | `LoginWrongPassword.shouldReturn401WhenPasswordIsWrong`                       | PASS |
| AC6 | POST /api/auth/login non-existent username -> 401  | `LoginNonExistentUsername.shouldReturn401WhenUsernameDoesNotExist`             | PASS |
| AC7 | GET /api/auth/me valid token -> 200 w/ id,username,displayName | `GetMeWithValidToken.shouldReturn200WithUserInfo` + `shouldNotReturnPasswordInMeResponse` | PASS |
| AC8 | GET /api/auth/me no auth header -> 401             | `GetMeWithoutAuthHeader.shouldReturn401WithoutAuthHeader`                     | PASS |
| AC9 | GET /api/auth/me expired token -> 401              | `GetMeWithExpiredToken.shouldReturn401WithExpiredToken` (manually crafted expired JWT) | PASS |
| AC10| GET /api/auth/me malformed token -> 401            | `GetMeWithMalformedToken` — 4 sub-tests: invalid string, random string, empty bearer, wrong auth scheme (Basic) | PASS |
| AC11| Register username <3 or >50 chars -> 400           | `RegisterUsernameLengthViolations` — 4 sub-tests: 2 chars, 1 char, empty, 51 chars | PASS |
| AC12| Register password <8 chars -> 400                  | `RegisterPasswordTooShort` — 3 sub-tests: 7 chars, 1 char, empty            | PASS |
| AC13| Register displayName empty or >100 chars -> 400    | `RegisterDisplayNameViolations` — 2 sub-tests: empty, 101 chars             | PASS |
| AC14| Register username with non-alphanumeric chars -> 400| `RegisterInvalidUsernameChars` — 5 sub-tests: space, @, hyphen, dot, unicode | PASS |

**Additional test coverage beyond ACs:**
- BR7 anti-enumeration: `shouldNotDistinguishBetweenNonExistentAndWrongPassword` verifies same 401 status code for both wrong-password and non-existent-user
- Boundary valid cases: username 3 chars, 50 chars; password 8 chars; displayName 1 char, 100 chars
- No-password-in-response tests on both register and /me endpoints
- End-to-end register -> login -> me flow test
- `AuthServiceTest`: 10 unit tests covering register/login/getCurrentUser service logic with mocks
- `RegisterRequestValidationTest`: 17 Jakarta Validation tests for DTO constraints
- `AuthDtoContractTest`: 14 contract tests for all 4 DTOs (getter/setter types, instantiation)

**Step 1 verdict**: PASS — All 14 ACs have explicit test coverage with good boundary testing.

---

## STEP 2: Tests <-> Code

| Check | Detail | Verdict |
|-------|--------|---------|
| Controller routes match test URLs | `/api/auth/register`, `/api/auth/login`, `/api/auth/me` — all match | PASS |
| `@Valid` on RegisterRequest | `AuthController.register()` uses `@Valid @RequestBody RegisterRequest` | PASS |
| HTTP 201 for register | `ResponseEntity.status(HttpStatus.CREATED).body(response)` | PASS |
| HTTP 409 for duplicate | `ResponseStatusException(HttpStatus.CONFLICT)` — Spring natively resolves this | PASS |
| HTTP 401 for login failure | Both user-not-found and wrong-password throw `ResponseStatusException(HttpStatus.UNAUTHORIZED, "Invalid credentials")` | PASS |
| Token generation uses user ID | `jwtTokenProvider.generateToken(user.getId())` — tests verify via mock and integration | PASS |
| expiresIn = 86400 | `LoginResponse(token, 86400)` hardcoded — test asserts `.value(86400)` | PASS |
| /me returns 401 without auth | Controller checks `authentication == null \|\| !(principal instanceof Long)` — works with `permitAll()` because anonymous principal is String, not Long | PASS |
| Validation annotations match tests | `@NotBlank @Size(min=3,max=50) @Pattern(regexp="^[a-zA-Z0-9_]+$")` on username; `@NotBlank @Size(min=8)` on password; `@NotBlank @Size(max=100)` on displayName | PASS |
| Unit tests mock interactions correct | `AuthServiceTest` verifies `passwordEncoder.encode()` called, `userRepository.save()` captures correct user, `jwtTokenProvider.generateToken(userId)` called | PASS |

**Step 2 verdict**: PASS — Tests and code are aligned; no mismatches found.

---

## STEP 3: Requirements <-> Code (Security Focus)

| Security Check | Detail | Verdict |
|----------------|--------|---------|
| **bcrypt storage** | `SecurityConfig` registers `BCryptPasswordEncoder` as `PasswordEncoder` bean; `AuthService.register()` calls `passwordEncoder.encode(request.getPassword())` | PASS |
| **No password in response** | `UserResponse` DTO has only `id`, `username`, `displayName` fields — no password field. Register returns `UserResponse`, /me returns `UserResponse` | PASS |
| **Anti-enumeration (BR7)** | Both login failure paths (user not found and wrong password) throw identical `ResponseStatusException(HttpStatus.UNAUTHORIZED, "Invalid credentials")` — same HTTP status and same message string | PASS |
| **JWT claims correct** | `JwtTokenProvider.generateToken()` sets `subject=userId`, `issuedAt=now`, `expiration=now+expirationMs`, signed with HMAC key. Claims match spec (user ID + expiration) | PASS |
| **JWT validation** | `JwtTokenProvider.validateToken()` catches all exceptions (expired, malformed, invalid signature) and returns false; `JwtAuthFilter` only sets authentication if `validateToken()` returns true | PASS |
| **Stateless session** | `SessionCreationPolicy.STATELESS` in SecurityConfig | PASS |
| **Token extraction** | `JwtAuthFilter.extractToken()` correctly handles: no header -> null, non-Bearer -> null, empty token after "Bearer " -> null | PASS |
| **BR5: 24h expiry** | `LoginResponse(token, 86400)` returns 86400 seconds; actual JWT expiry controlled by `app.jwt.expiration` config property | PASS |

**Step 3 verdict**: PASS — All security requirements satisfied.

---

## STEP 4: Standards

SKIPPED (no standards file provided).

---

## STEP 5: Code Quality

| Check | Detail | Verdict |
|-------|--------|---------|
| **No security vulnerabilities** | bcrypt for passwords, JWT with HMAC signing, stateless sessions, anti-enumeration | PASS |
| **Exception handling** | `ResponseStatusException` used consistently for HTTP error responses; `GlobalExceptionHandler` handles `MethodArgumentNotValidException` for validation errors, `DuplicateException`, `ResourceNotFoundException` | PASS |
| **Constructor injection** | All classes use constructor injection (no field injection) | PASS |
| **Clean code** | DTOs are simple POJOs with proper getters/setters; service is focused with clear single-responsibility; controller is thin and delegates to service | PASS |
| **No hardcoded secrets** | JWT secret comes from `@Value("${app.jwt.secret}")` config property | PASS |
| **Null safety** | `JwtTokenProvider.validateToken()` checks for null/empty token; `AuthController.me()` checks for null authentication | PASS |

**Minor observation (non-blocking):** The `SecurityConfig` uses `/api/auth/**` as `permitAll()`, which also permits `/api/auth/me` without Spring Security enforcing authentication. The `AuthController.me()` compensates by manually checking the principal. This works correctly (anonymous user has String principal "anonymousUser", not Long), but a cleaner approach would be to use specific matchers: `requestMatchers("/api/auth/register", "/api/auth/login").permitAll()`. However, this is a design choice, not a defect — the current implementation works correctly as validated by integration tests.

**Step 5 verdict**: PASS

---

## STEP 6: File Scope

ALLOWED files: `AuthService.java`, `AuthController.java`, `RegisterRequest.java`, `LoginRequest.java`, `LoginResponse.java`, `UserResponse.java`

All 6 implementation files reviewed are within the allowed list. No out-of-scope files were created or modified by this task's implementation.

**Step 6 verdict**: PASS

---

## Summary

| Step | Description               | Result |
|------|---------------------------|--------|
| 1    | Requirements <-> Tests    | PASS   |
| 2    | Tests <-> Code            | PASS   |
| 3    | Requirements <-> Code     | PASS   |
| 4    | Standards                 | SKIP   |
| 5    | Code Quality              | PASS   |
| 6    | File Scope                | PASS   |

**Test count**: 4 test classes, ~41+ integration tests, ~17 validation tests, ~14 DTO contract tests, ~10 service unit tests

**Final Verdict**: **QA-PASS**
