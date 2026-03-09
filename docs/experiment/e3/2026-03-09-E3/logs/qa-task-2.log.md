# QA Log — T2: M1 — JWT Security Infrastructure

## Result: QA-PASS

## Files Reviewed
- `security/JwtUtil.java` — JWT utility: generateToken, getUserIdFromToken, validateToken
- `security/JwtAuthenticationFilter.java` — OncePerRequestFilter extracting Bearer token
- `dto/UserResponse.java` — Record with id, username, displayName

## STEP 1: Requirements <-> Tests

| Requirement | Test Coverage |
|---|---|
| JwtUtil.generateToken | `should_generate_token_when_given_valid_userId`, `should_produce_three_part_jwt_format`, `should_generate_different_tokens_for_different_users` |
| JwtUtil.getUserIdFromToken | `should_return_userId_when_given_valid_token`, `should_contain_userId_claim_in_token`, `should_throw_when_extracting_userId_from_invalid_token`, `should_throw_when_extracting_userId_from_expired_token` |
| JwtUtil.validateToken | `should_return_true_when_validating_valid_token`, `should_return_false_when_validating_malformed_token`, `should_return_false_when_validating_empty_token`, `should_return_false_when_validating_null_token`, `should_return_false_when_token_signed_with_different_secret`, `should_return_false_when_token_is_expired` |
| JWT secret+expiration from config | Constructor tested via @BeforeEach with injected values |
| JwtAuthenticationFilter as OncePerRequestFilter | 7 tests covering valid/invalid/missing/non-bearer/empty/exception scenarios |
| UserResponse DTO | `should_create_user_response_with_all_fields`, `should_have_correct_field_types` |

Verdict: PASS — All AC requirements have corresponding tests.

## STEP 2: Tests <-> Code

- JwtUtilTest (12 tests): All three public methods tested. Boundary cases (null, empty, malformed, wrong secret, expired) covered.
- JwtAuthenticationFilterTest (7 tests): doFilterInternal tested with mocked JwtUtil. extractToken indirectly tested via header variations. filterChain.doFilter always called.
- UserResponseTest (2 tests): Record accessor methods verified.

Verdict: PASS — Tests align with implementation.

## STEP 3: Requirements <-> Code

- JwtUtil: Uses `@Value("${app.jwt.secret}")` and `@Value("${app.jwt.expiration}")` — config-driven. HMAC-SHA key via `Keys.hmacShaKeyFor()`. Modern jjwt API.
- JwtAuthenticationFilter: Extends `OncePerRequestFilter`. Extracts Bearer token, validates, sets `SecurityContextHolder` with userId as principal. Always forwards to filter chain.
- UserResponse: `record(Long id, String username, String displayName)` matches CONTRACT `{id:long,username:string,displayName:string}`.

Verdict: PASS — Code satisfies all requirements.

## STEP 4: Standards Compliance

SKIPPED — No standards files provided.

## STEP 5: Code Quality

- No security vulnerabilities. Secret injected from config, not hardcoded.
- validateToken: Null/empty guard + catch-all Exception (correct — jjwt throws multiple exception types).
- Filter: try-catch ensures filter chain always proceeds even on errors.
- UserResponse: Java record — immutable, concise.
- No unnecessary complexity or over-engineering.

Verdict: PASS

## STEP 6: File Scope

ALLOWED: security/JwtUtil.java, security/JwtAuthenticationFilter.java, dto/UserResponse.java
ACTUAL: Same three files. No out-of-scope modifications.

Verdict: PASS
