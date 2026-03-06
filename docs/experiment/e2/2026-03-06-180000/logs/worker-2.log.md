# Worker-2 Log: T2 Auth Service + Controller + DTOs

## Phase 1: Reading & Analysis
- Read PLAN, CONTRACT, impl-notes-2, all READONLY files, both test files
- Confirmed JwtAuthenticationFilter sets `principal = Long userId`
- Confirmed SecurityConfig has `permitAll("/api/auth/**")` — /me must manually check auth

## Phase 2: Implementation (5 files)
Created all 5 ALLOWED files:
1. `RegisterRequest.java` — Jakarta validation: @Size(3-50), @Pattern(alphanumeric+underscore), @Size(min=8) password, @NotBlank+@Size(1-100) displayName
2. `LoginRequest.java` — @NotBlank on both username and password
3. `LoginResponse.java` — POJO with constructor(token, expiresIn), getters/setters
4. `AuthService.java` — register/login/getCurrentUser with BCrypt, custom exceptions (DuplicateUsernameException, BadCredentialsException, UserNotFoundException)
5. `AuthController.java` — POST /register (201), POST /login (200), GET /me (manual auth check from SecurityContextHolder), @ExceptionHandler for all error types

## Phase 2.5: Bug Fix
- `RegisterRequest` needed `@NotBlank` on username and `@NotNull` on password to reject null fields (Jakarta `@Size`/`@Pattern` skip null by default)

## Phase 3: Testing
- Run 1: BLOCKED — `AuthControllerIntegrationTest.java` missing JUnit Assertions import (test-agent bug, TL fixed)
- Run 2: 2 failures — missing null validation on username/password in RegisterRequest
- Run 3: ALL 47 TESTS PASSED (13 unit + 34 integration), 0 failures, 0 errors

## Scope Check
- `git diff --name-only` shows no modified files by me (SecurityConfig.java change is from worker-1)
- My 5 files are untracked (new), all within ALLOWED scope:
  - backend/src/main/java/com/taskflow/dto/RegisterRequest.java
  - backend/src/main/java/com/taskflow/dto/LoginRequest.java
  - backend/src/main/java/com/taskflow/dto/LoginResponse.java
  - backend/src/main/java/com/taskflow/service/AuthService.java
  - backend/src/main/java/com/taskflow/controller/AuthController.java
