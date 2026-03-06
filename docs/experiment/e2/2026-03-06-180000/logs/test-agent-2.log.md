# Test Agent 2 Log — T2: Auth Service + Controller + DTOs

## Input Analysis
- Read T1 implementation: User.java, UserRepository.java, JwtTokenProvider.java, SecurityConfig.java, JwtAuthenticationFilter.java
- Read existing T1 tests for patterns: UserTest.java, UserRepositoryTest.java, SecurityIntegrationTest.java
- Read PLAN.md, CONTRACT.md, M1-auth.md spec
- Read application.yml for JWT secret key

## Key Design Decisions

### AuthService Unit Tests (AuthServiceTest.java)
- Uses @ExtendWith(MockitoExtension.class) with @Mock for UserRepository, PasswordEncoder, JwtTokenProvider
- Tests register(), login(), getCurrentUser() in isolation
- Verifies BCrypt encoding is called before save
- Verifies no user enumeration (same exception type for wrong password vs nonexistent user)
- Verifies response shapes (exactly id/username/displayName for register and getCurrentUser)
- Assumes register/getCurrentUser return Map<String,Object> — worker can change to a DTO if preferred, jsonPath tests will still pass

### AuthController Integration Tests (AuthControllerIntegrationTest.java)
- Uses @SpringBootTest + @AutoConfigureMockMvc + @Transactional
- Tests full HTTP request/response cycle through MockMvc
- Organized into @Nested classes: RegisterEndpoint, LoginEndpoint, MeEndpoint, EndToEndFlow
- Boundary tests for all validation rules (AC11-AC14)
- Contract tests verifying response schema matches CONTRACT.md
- E2E flow: register -> login -> me

### Critical Gotcha: /api/auth/me
- SecurityConfig permits /api/auth/** — controller must manually check authentication
- Tests verify 401 for no token, expired token, malformed token on /me endpoint
- Worker must implement auth check in controller since Spring Security won't block it

## Test Files Created

### 1. AuthServiceTest.java
- Path: backend/src/test/java/com/taskflow/service/AuthServiceTest.java
- 13 test methods

### 2. AuthControllerIntegrationTest.java
- Path: backend/src/test/java/com/taskflow/controller/AuthControllerIntegrationTest.java
- 28 test methods (across 4 nested classes)

### 3. Implementation Notes
- Path: docs/dev-team/2026-03-06-180000/temp/impl-notes-2.md

## Summary
- Total test files: 2
- Total test methods: 41
- Coverage: AC1, AC2, AC3, AC4, AC5, AC6, AC7, AC8, AC9, AC10, AC11, AC12, AC13, AC14 + boundary tests + contract tests + E2E flow
