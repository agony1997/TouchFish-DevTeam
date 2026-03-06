# Test Agent Log - T1: User Entity + JWT Security Foundation

## Test Files Created

1. `backend/src/test/java/com/taskflow/entity/UserTest.java` (11 tests)
   - Field access tests (username, password, displayName, id, createdAt)
   - Username validation: min 3, max 50, alphanumeric+underscore, reject special chars
   - DisplayName validation: reject empty, max 100, accept 100

2. `backend/src/test/java/com/taskflow/repository/UserRepositoryTest.java` (6 tests)
   - findByUsername: found, not found
   - Auto-generate ID
   - Unique username enforcement
   - Bcrypt hash stored (M1-AC3)
   - createdAt auto-set

3. `backend/src/test/java/com/taskflow/security/JwtTokenProviderTest.java` (10 tests)
   - generateToken: non-null
   - validateToken: valid, malformed (M1-AC10), null, empty, wrong signature, expired (M1-AC9)
   - getUserIdFromToken: single, multiple
   - Different tokens for different users

4. `backend/src/test/java/com/taskflow/security/JwtAuthenticationFilterTest.java` (7 tests)
   - No auth header -> no auth set (M1-AC8)
   - Non-Bearer header -> no auth set
   - Valid Bearer -> auth set with userId
   - Invalid token -> no auth set (M1-AC10)
   - Exception during validation -> no auth set
   - Token extraction from Bearer prefix
   - Filter chain always continues

5. `backend/src/test/java/com/taskflow/security/SecurityIntegrationTest.java` (7 tests)
   - Auth endpoints accessible without token
   - Protected endpoint 401 without token (M1-AC8)
   - Protected endpoint 401 with expired token (M1-AC9)
   - Protected endpoint 401 with malformed token (M1-AC10)
   - BCryptPasswordEncoder bean exists (M1-AC3)
   - Bcrypt encoding works
   - H2 console accessible
   - Valid token allows access

## AC Coverage

| AC | Test |
|----|------|
| M1-AC3 | UserRepositoryTest.shouldStoreBcryptHashNotPlaintext, SecurityIntegrationTest.shouldEncodeToBcryptHash |
| M1-AC8 | JwtAuthenticationFilterTest.shouldNotSetAuthenticationWhenNoAuthorizationHeader, SecurityIntegrationTest.shouldReturn401ForProtectedEndpointWithoutToken |
| M1-AC9 | JwtTokenProviderTest.shouldReturnFalseForExpiredToken, SecurityIntegrationTest.shouldReturn401ForProtectedEndpointWithExpiredToken |
| M1-AC10 | JwtTokenProviderTest.shouldReturnFalseForMalformedToken, JwtAuthenticationFilterTest.shouldNotSetAuthenticationWhenInvalidToken, SecurityIntegrationTest.shouldReturn401ForProtectedEndpointWithMalformedToken |

## Compilation Result

All compilation errors are "cannot find symbol" for implementation classes that don't exist yet:
- `com.taskflow.entity.User`
- `com.taskflow.repository.UserRepository`
- `com.taskflow.security.JwtTokenProvider`
- `com.taskflow.security.JwtAuthenticationFilter`

No syntax errors in test code. Tests are in RED state, ready for implementation.

## Total: 41 tests across 5 files
