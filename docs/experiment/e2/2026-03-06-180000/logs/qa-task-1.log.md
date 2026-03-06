# QA Log — T1: M1 - User Entity + JWT Security Foundation

## Test Execution
- Command: `mvn test -Dtest="UserTest,UserRepositoryTest,JwtTokenProviderTest,JwtAuthenticationFilterTest,SecurityIntegrationTest"`
- Result: **43 tests run, 0 failures, 0 errors, 0 skipped — BUILD SUCCESS**

## STEP 1: Requirements <-> Tests

| Requirement | Test Coverage | Verdict |
|---|---|---|
| User entity (id, username, password, displayName, createdAt) | UserTest: field get/set, null id before persist, null createdAt before persist | PASS |
| Username: unique, 3-50, alphanumeric+underscore | UserTest: reject <3, accept 3, accept 50, reject >50, accept alphanum+underscore, reject special chars; UserRepositoryTest: enforce unique | PASS |
| DisplayName: 1-100 | UserTest: reject empty, accept 100, reject >100 | PASS |
| Password: bcrypt stored (M1-AC3) | UserRepositoryTest: shouldStoreBcryptHashNotPlaintext; SecurityIntegrationTest: shouldEncodeToBcryptHash | PASS |
| UserRepository.findByUsername | UserRepositoryTest: found + not found | PASS |
| JwtTokenProvider: generateToken | JwtTokenProviderTest: non-null, valid, different for different users | PASS |
| JwtTokenProvider: validateToken | JwtTokenProviderTest: valid, malformed (AC10), null, empty, wrong signature, expired (AC9) | PASS |
| JwtTokenProvider: getUserIdFromToken | JwtTokenProviderTest: single + multiple user IDs | PASS |
| JwtAuthenticationFilter: Bearer extraction | JwtAuthenticationFilterTest: no header (AC8), non-Bearer, valid Bearer, invalid token, exception handling, filter chain always continues | PASS |
| SecurityConfig: JWT filter registered | SecurityIntegrationTest: valid token allows access, invalid/missing denies | PASS |
| SecurityConfig: AuthenticationEntryPoint 401 | SecurityIntegrationTest: no header->401 (AC8), expired->401 (AC9), malformed->401 (AC10) | PASS |
| SecurityConfig: /api/auth/** permitAll | SecurityIntegrationTest: shouldAllowAccessToAuthEndpointsWithoutToken | PASS |
| BCryptPasswordEncoder bean | SecurityIntegrationTest: shouldHaveBCryptPasswordEncoderBean | PASS |
| JJWT 0.12.5 | pom.xml: jjwt.version=0.12.5 with api/impl/jackson | PASS |

**Step 1 Verdict: PASS** — All requirements have corresponding test coverage.

## STEP 2: Tests <-> Code

| Test Class | Implementation Class | Verified Behavior Matches | Verdict |
|---|---|---|---|
| UserTest (11 tests) | User.java | Entity annotations (@Entity, @Table, @Id, @GeneratedValue, @Size, @Pattern, @NotBlank, @Column unique, @PrePersist) all match test expectations | PASS |
| UserRepositoryTest (6 tests) | UserRepository.java | Interface extends JpaRepository, findByUsername returns Optional<User>, unique constraint enforced by @Column(unique=true) | PASS |
| JwtTokenProviderTest (9 tests) | JwtTokenProvider.java | generateToken uses JJWT builder with subject=userId, validateToken catches all exceptions, getUserIdFromToken parses subject | PASS |
| JwtAuthenticationFilterTest (7 tests) | JwtAuthenticationFilter.java | Extends OncePerRequestFilter, extracts Bearer token, sets SecurityContext on valid token, catches exceptions, always continues filter chain | PASS |
| SecurityIntegrationTest (8 tests) | SecurityConfig.java | CSRF disabled, stateless sessions, custom AuthenticationEntryPoint returning 401+JSON, /api/auth/** and /h2-console/** permitAll, JWT filter before UsernamePasswordAuthenticationFilter, BCryptPasswordEncoder bean | PASS |

**Step 2 Verdict: PASS** — All tests correctly verify the implementation behavior.

## STEP 3: Requirements <-> Code

| Requirement | Code Location | Verdict |
|---|---|---|
| User entity fields: id (Long, auto), username (String unique 3-50 alphanum+_), password (String), displayName (String 1-100 NotBlank), createdAt (LocalDateTime @PrePersist) | User.java:18-82 | PASS |
| UserRepository.findByUsername | UserRepository.java:10 | PASS |
| JwtTokenProvider.generateToken(Long userId) → String | JwtTokenProvider.java:29-41 | PASS |
| JwtTokenProvider.validateToken(String token) → boolean | JwtTokenProvider.java:43-54 | PASS |
| JwtTokenProvider.getUserIdFromToken(String token) → Long | JwtTokenProvider.java:56-65 | PASS |
| JJWT 0.12.5 API usage (Jwts.builder/parser, verifyWith, parseSignedClaims) | JwtTokenProvider.java — correct 0.12.x API | PASS |
| JwtAuthenticationFilter extends OncePerRequestFilter | JwtAuthenticationFilter.java:16 | PASS |
| Bearer token extraction from Authorization header | JwtAuthenticationFilter.java:43-49 | PASS |
| Sets UsernamePasswordAuthenticationToken in SecurityContext | JwtAuthenticationFilter.java:32-34 | PASS |
| SecurityConfig: JWT filter registration before UsernamePasswordAuthenticationFilter | SecurityConfig.java:47 | PASS |
| SecurityConfig: AuthenticationEntryPoint returns 401 + JSON | SecurityConfig.java:33-37 | PASS |
| SecurityConfig: /api/auth/** permitAll | SecurityConfig.java:40 | PASS |
| SecurityConfig: BCryptPasswordEncoder bean | SecurityConfig.java:53-55 | PASS |
| SecurityConfig: stateless session, CSRF disabled | SecurityConfig.java:28-31 | PASS |

**Step 3 Verdict: PASS** — Code fully implements all requirements.

## STEP 4: Standards Compliance
SKIPPED — No standards files provided.

## STEP 5: Code Quality

| Aspect | Assessment | Verdict |
|---|---|---|
| No hardcoded secrets | Default secret via @Value with property placeholder; acceptable for dev | PASS |
| Exception handling | JwtTokenProvider.validateToken catches all exceptions; JwtAuthenticationFilter catches and silently continues — correct for filter | PASS |
| Thread safety | SecurityContextHolder usage is correct | PASS |
| No SQL injection risk | Using Spring Data JPA query methods | PASS |
| No XSS risk | JSON responses only | PASS |
| Proper password handling | BCryptPasswordEncoder provided as bean; password stored as hash | PASS |
| Session management | Stateless — correct for JWT | PASS |
| Token expiration | Configurable via @Value; expired token properly rejected | PASS |

**Step 5 Verdict: PASS**

## STEP 6: File Scope

ALLOWED files: entity/User.java, repository/UserRepository.java, security/JwtTokenProvider.java, security/JwtAuthenticationFilter.java, config/SecurityConfig.java

Modified/created main source files (from git status):
- backend/src/main/java/com/taskflow/entity/User.java — NEW, ALLOWED
- backend/src/main/java/com/taskflow/repository/UserRepository.java — NEW, ALLOWED
- backend/src/main/java/com/taskflow/security/JwtAuthenticationFilter.java — NEW, ALLOWED
- backend/src/main/java/com/taskflow/security/JwtTokenProvider.java — NEW, ALLOWED
- backend/src/main/java/com/taskflow/config/SecurityConfig.java — MODIFIED, ALLOWED

**Step 6 Verdict: PASS** — No out-of-scope files modified.

## Final Verdict: QA-PASS

All 43 tests pass. Three-way cross-verification (Requirements <-> Tests <-> Code) confirms full alignment. No out-of-scope files modified. No code quality issues found.
