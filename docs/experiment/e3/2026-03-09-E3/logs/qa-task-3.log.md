# QA Log — T3: Auth Service + Controller + Error Handling

## Result: QA-PASS

## Files Reviewed
- `service/AuthService.java`
- `controller/AuthController.java`
- `exception/GlobalExceptionHandler.java`
- `exception/ResourceNotFoundException.java`
- `dto/ErrorResponse.java`
- `dto/RegisterRequest.java` (supporting, for validation annotations)
- `dto/LoginResponse.java` (supporting)
- `dto/UserResponse.java` (supporting)
- `entity/User.java` (supporting, for unique constraint)
- `controller/AuthControllerTest.java` (test file)

## STEP 1: Requirements <-> Tests

| AC | Requirement | Test(s) | Status |
|----|-------------|---------|--------|
| AC1 | Register 201 {id, username, displayName} | `register_validData_returns201`, `register_validData_noPasswordInResponse` | COVERED |
| AC2 | Duplicate username 409 | `register_duplicateUsername_returns409` | COVERED |
| AC3 | BCrypt password storage | `register_passwordStoredAsBcrypt` | COVERED |
| AC4 | Login 200 {token, expiresIn=86400} | `login_validCredentials_returns200WithToken`, `login_tokenIsValidJwt` | COVERED |
| AC5 | Wrong password 401 | `login_wrongPassword_returns401` | COVERED |
| AC6 | Non-existent user 401 | `login_nonExistentUser_returns401` | COVERED |
| AC7 | /me 200 {id, username, displayName} | `me_validToken_returns200`, `me_validToken_noPasswordInResponse` | COVERED |
| AC8 | /me no auth 401 | `me_noAuthHeader_returns401` | COVERED |
| AC10 | Malformed/empty/no-bearer 401 | `me_malformedToken_returns401`, `me_emptyBearer_returns401`, `me_noBearerPrefix_returns401` | COVERED |
| AC11 | Username 3-50 chars | 4 boundary tests (too short, too long, exact 3, exact 50) | COVERED |
| AC12 | Password min 8 | 2 boundary tests (too short, exact 8) | COVERED |
| AC13 | DisplayName 1-100 | 2 tests (blank, over 100) | COVERED |
| AC14 | Username alphanumeric+underscore | 3 tests (special chars, spaces, underscore valid) | COVERED |
| BR7 | No user enumeration | `login_errorMessagesAreIdentical_preventEnumeration` | COVERED |
| Error format | {message, timestamp} | `ErrorResponseContractTests` (400, 409, 401) | COVERED |

## STEP 2: Tests <-> Code

- Register: Controller uses `@Valid @RequestBody`, returns 201. AuthService encodes with `passwordEncoder.encode()`, returns `UserResponse` record (no password field). MATCH.
- Login: AuthService returns null for both non-existent user and wrong password. Controller maps null to 401 with `ErrorResponse("Invalid username or password")`. Identical error for both cases. MATCH.
- /me: Controller checks `SecurityContextHolder` authentication, returns 401 for anonymous/null. Casts principal to `Long userId`, delegates to `authService.getCurrentUser()`. MATCH.
- Duplicate: DB `unique` constraint on `User.username` -> `DataIntegrityViolationException` -> `GlobalExceptionHandler` returns 409. MATCH.
- Validation: `RegisterRequest` has `@Pattern(regexp = "^[a-zA-Z0-9_]{3,50}$")`, `@Size(min=8)` + `@NotBlank` for password, `@NotBlank` + `@Size(max=100)` for displayName. `GlobalExceptionHandler` catches `MethodArgumentNotValidException` -> 400. MATCH.

## STEP 3: Requirements <-> Code (Critical Checks)

- **BCrypt**: `passwordEncoder.encode()` at AuthService:30. Spring `PasswordEncoder` bean (BCrypt). CONFIRMED.
- **BR7 no enumeration**: AuthService.login() returns null for both non-existent user (`orElse(null)`) and wrong password. Controller produces identical error message `"Invalid username or password"`. CONFIRMED.
- **expiresIn=86400**: AuthService:44 `response.setExpiresIn(86400)`. CONFIRMED.
- **/me auth check**: Controller checks authentication != null, isAuthenticated(), principal != null, not "anonymousUser". CONFIRMED.
- **ErrorResponse {message, timestamp}**: ErrorResponse constructor sets `timestamp = Instant.now().toString()` (ISO8601). CONFIRMED.

## STEP 4: Standards — SKIPPED (no standards files provided)

## STEP 5: Code Quality / Security

- No password in any response DTO (`UserResponse` is a record with only id, username, displayName). SAFE.
- GlobalExceptionHandler covers validation (400), data integrity (409), resource not found (404). Login 401 in controller. ADEQUATE.
- No SQL injection risk (JPA repository). No XSS (REST API). No command injection.
- Duplicate username detection via DB constraint is acceptable for single unique field.

## STEP 6: File Scope

Allowed: `service/AuthService.java`, `controller/AuthController.java`, `exception/GlobalExceptionHandler.java`, `exception/ResourceNotFoundException.java`, `dto/ErrorResponse.java`.
All implementation files within allowed scope. No out-of-scope modifications.

## Observations

None. All requirements fully covered with correct implementation.
