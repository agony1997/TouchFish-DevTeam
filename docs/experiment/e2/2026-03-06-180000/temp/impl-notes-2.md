# Implementation Notes for T2: Auth Service + Controller + DTOs

## Critical Observations from Tests

### 1. DTO Validation Annotations
- `RegisterRequest` MUST have Jakarta validation annotations matching the User entity:
  - `@Size(min = 3, max = 50)` + `@Pattern(regexp = "^[a-zA-Z0-9_]+$")` on `username`
  - `@Size(min = 8)` on `password`
  - `@NotBlank` + `@Size(min = 1, max = 100)` on `displayName`
- `LoginRequest` MUST have `@NotBlank` on both `username` and `password`
- `LoginResponse` has two fields: `token` (String) and `expiresIn` (long, hardcoded 86400)

### 2. AuthService Return Types
- `register()` and `getCurrentUser()` return `Map<String, Object>` with exactly 3 keys: `id`, `username`, `displayName`
  - Tests verify no `password`, `createdAt`, or `token` leaks into response
  - Alternatively, you can create a DTO class and return that — but tests use jsonPath so either works
- `login()` returns `LoginResponse` with `token` + `expiresIn`

### 3. AuthService Exception Handling
- **Duplicate username**: Check via `userRepository.findByUsername()` BEFORE save. Throw a custom exception that maps to 409.
- **Bad credentials**: Throw a custom exception that maps to 401. SAME exception class for both "user not found" and "wrong password" (BR7 — no user enumeration).
- **User not found in getCurrentUser**: Throw exception → 401 or 404.

### 4. AuthController Specifics
- `POST /api/auth/register` → `@ResponseStatus(HttpStatus.CREATED)` or return `ResponseEntity.status(201)`
- `POST /api/auth/login` → 200 OK (default)
- `GET /api/auth/me` → Must extract userId from SecurityContext. The JWT filter sets `UsernamePasswordAuthenticationToken` with `principal = userId (Long)`. Access via `SecurityContextHolder` or `@AuthenticationPrincipal`.

### 5. /api/auth/me Authentication Gotcha
- SecurityConfig has `requestMatchers("/api/auth/**").permitAll()` — this means the security filter chain does NOT block unauthenticated requests to /api/auth/me.
- The controller MUST manually check if the user is authenticated. If no valid token → the principal will be null or anonymous.
- Tests expect 401 when no token is provided. Options:
  - Check `Authentication` from SecurityContext; if null or principal is not a Long, return 401 manually.
  - OR use a `@PreAuthorize` / custom check inside the handler method.

### 6. Error Response Format
- All error responses must be JSON: `{"message": "some error description"}`
- Use `@ExceptionHandler` in the controller or a `@ControllerAdvice` class.
- `MethodArgumentNotValidException` → 400 with `{"message": "..."}`
- Custom duplicate username exception → 409 with `{"message": "..."}`
- Custom bad credentials exception → 401 with `{"message": "..."}`
- The login bad-credential 400 test sends `{}` (empty body) and expects 400 — so LoginRequest validation must trigger for missing fields.

### 7. BCrypt Password Hashing
- Inject `PasswordEncoder` (BCrypt bean from SecurityConfig) into AuthService.
- `register()`: `passwordEncoder.encode(request.getPassword())` before saving.
- `login()`: `passwordEncoder.matches(request.getPassword(), user.getPassword())`

### 8. Dependencies Already Available (from T1)
- `UserRepository.findByUsername(String)` → `Optional<User>`
- `JwtTokenProvider.generateToken(Long userId)` → `String`
- `JwtTokenProvider.validateToken(String)` → `boolean`
- `JwtTokenProvider.getUserIdFromToken(String)` → `Long`
- `PasswordEncoder` bean (BCryptPasswordEncoder) from SecurityConfig
- `JwtAuthenticationFilter` sets `authentication.getPrincipal()` = `Long userId`

### 9. RegisterRequest / LoginRequest Must Have Getters/Setters
- No Lombok in this project. Write manual getters and setters.
- Tests call `req.setUsername(...)`, `req.setPassword(...)`, `req.setDisplayName(...)`.

### 10. Test Expectations Summary
| Test | Endpoint | Expected Status | Key Assertions |
|------|----------|----------------|----------------|
| AC1 | POST /register | 201 | id (number), username, displayName; no password |
| AC2 | POST /register (dup) | 409 | message exists |
| AC3 | POST /register | 201 | DB password starts with $2a$ or $2b$ |
| AC4 | POST /login | 200 | token (non-empty string), expiresIn = 86400 |
| AC5 | POST /login (wrong pw) | 401 | message exists |
| AC6 | POST /login (no user) | 401 | message exists |
| AC7 | GET /me (valid token) | 200 | id, username, displayName |
| AC8 | GET /me (no token) | 401 | — |
| AC9 | GET /me (expired) | 401 | — |
| AC10 | GET /me (malformed) | 401 | — |
| AC11 | POST /register | 400 | username 2 chars, 51 chars |
| AC12 | POST /register | 400 | password 7 chars |
| AC13 | POST /register | 400 | displayName empty, 101 chars |
| AC14 | POST /register | 400 | username with space, @!, hyphen |
