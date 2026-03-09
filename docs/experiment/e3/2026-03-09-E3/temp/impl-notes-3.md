# Implementation Notes — T3: Auth Service + Controller + Error Handling

## Test Design Summary

29 integration tests written using `@SpringBootTest` + `@AutoConfigureMockMvc`.
All tests are HTTP-level (MockMvc), so they test the full stack without importing AuthService/AuthController directly.

## Files Worker Must Create (5 files max)

1. **AuthService** (`com.taskflow.service.AuthService`)
   - `register(RegisterRequest)` → validate uniqueness, bcrypt password, save User, return UserResponse
   - `login(LoginRequest)` → find user, verify password, generate JWT, return LoginResponse with expiresIn=86400
   - `getCurrentUser(Long userId)` → find user by id, return UserResponse

2. **AuthController** (`com.taskflow.controller.AuthController`)
   - `POST /api/auth/register` → `@Valid @RequestBody RegisterRequest`, return 201
   - `POST /api/auth/login` → `@RequestBody LoginRequest`, return 200
   - `GET /api/auth/me` → extract userId from SecurityContext (principal), return 200
   - For `/me`: must check `authentication == null || !authentication.isAuthenticated()` and return 401
     - Note: `/api/auth/**` is `permitAll()` in SecurityConfig, so Spring Security won't auto-reject
     - The JwtAuthenticationFilter still sets SecurityContext if a valid token is present
     - Controller must manually check authentication status

3. **GlobalExceptionHandler** (`com.taskflow.exception.GlobalExceptionHandler`)
   - `@RestControllerAdvice`
   - Handle `MethodArgumentNotValidException` → 400 with ErrorResponse
   - Handle duplicate key / `DataIntegrityViolationException` → 409 with ErrorResponse
   - Handle `ResourceNotFoundException` → 404 with ErrorResponse
   - Handle custom `UnauthorizedException` or use ResponseStatusException → 401

4. **ResourceNotFoundException** (`com.taskflow.exception.ResourceNotFoundException`)
   - Extends `RuntimeException`

5. **ErrorResponse** (`com.taskflow.dto.ErrorResponse`)
   - Fields: `String message`, `String timestamp` (ISO8601)

## Key Constraints from Tests

- Register returns 201 (not 200)
- Login expiresIn must be exactly `86400` (seconds, not milliseconds)
- Password must NOT appear in any response
- BR7: Login error messages for wrong-password and non-existent-user must be IDENTICAL
- `/api/auth/me` without valid token must return 401 (controller-level check needed since path is permitAll)
- ErrorResponse format: `{message: string, timestamp: string}` — no other fields

## SecurityConfig Note

The existing SecurityConfig has `.requestMatchers("/api/auth/**").permitAll()` which means `/api/auth/me` is NOT auto-protected. The controller must manually verify authentication.
