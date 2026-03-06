# QA Log: T2 - Auth Service + Controller + DTOs

## Test Execution

```
Tests run: 47, Failures: 0, Errors: 0, Skipped: 0
BUILD SUCCESS (11.662s)
```

- AuthServiceTest: 13 tests PASS
- AuthControllerIntegrationTest: 34 tests PASS (RegisterEndpoint 20 + LoginEndpoint 8 + MeEndpoint 5 + EndToEndFlow 1)

## Three-Way Verification: CONTRACT vs Tests vs Implementation

| AC | Description | Contract | Impl | Tests | Status |
|-----|-------------|----------|------|-------|--------|
| AC1 | Register 201 {id,username,displayName} | Y | Y | Y | PASS |
| AC2 | Duplicate username 409 | Y | Y | Y | PASS |
| AC3 | BCrypt password hashing | Y | Y | Y | PASS |
| AC4 | Login 200 {token, expiresIn:86400} | Y | Y | Y | PASS |
| AC5 | Wrong password 401 | Y | Y | Y | PASS |
| AC6 | Nonexistent user 401 | Y | Y | Y | PASS |
| AC7 | GET /me 200 {id,username,displayName} | Y | Y | Y | PASS |
| AC8 | No auth header 401 | Y | Y | Y | PASS |
| AC9 | Expired token 401 | Y | Y | Y | PASS |
| AC10 | Malformed token 401 | Y | Y | Y | PASS |
| AC11 | Username 3-50, regex ^[a-zA-Z0-9_]+$ | Y | Y | Y | PASS |
| AC12 | Password min 8 | Y | Y | Y | PASS |
| AC13 | DisplayName 1-100 | Y | Y | Y | PASS |
| AC14 | Username regex enforcement | Y | Y | Y | PASS |
| Error fmt | {"message":"..."} | Y | Y | Y | PASS |
| BR7 | No user enumeration (same error msg) | Y | Y | Y | PASS |

## Code Quality

- Password hashing: BCrypt via PasswordEncoder, no plaintext exposure
- Anti-enumeration: login throws same BadCredentialsException with same message for both wrong password and nonexistent user
- DTO validation: Jakarta @Valid annotations match CONTRACT constraints exactly
- Exception handling: custom exceptions with proper HTTP status mapping (409, 401, 400)
- Constructor injection throughout (no @Autowired field injection)
- Response shape verified: no extra fields leak (password, createdAt, etc.)

## File Scope Check

All implementation limited to ALLOWED files:
- dto/RegisterRequest.java
- dto/LoginRequest.java
- dto/LoginResponse.java
- service/AuthService.java
- controller/AuthController.java

No unauthorized file modifications detected.

## Verdict

**QA-PASS**

All 47 tests pass. All 14 ACs + error format + anti-enumeration verified across CONTRACT, implementation, and tests. No code quality issues found.
