# Implementation Notes for T1: User Entity + JWT Security Foundation

## Files to Create/Modify

1. `backend/src/main/java/com/taskflow/entity/User.java`
2. `backend/src/main/java/com/taskflow/repository/UserRepository.java`
3. `backend/src/main/java/com/taskflow/security/JwtTokenProvider.java`
4. `backend/src/main/java/com/taskflow/security/JwtAuthenticationFilter.java`
5. `backend/src/main/java/com/taskflow/config/SecurityConfig.java` (modify existing)

## Framework Gotchas

### User Entity
- NO Lombok. Write getters/setters manually.
- Use `@Entity`, `@Table(name = "users")` (avoid reserved word `user` in H2).
- `@Id` + `@GeneratedValue(strategy = GenerationType.IDENTITY)` for auto-increment.
- `@Column(unique = true)` on username to enforce uniqueness at DB level.
- Username validation: `@Size(min = 3, max = 50)` + `@Pattern(regexp = "^[a-zA-Z0-9_]+$")`.
- DisplayName validation: `@NotBlank` + `@Size(min = 1, max = 100)`.
- `createdAt` should use `@Column(updatable = false)` + `@PrePersist` callback to set `LocalDateTime.now()`.
- Tests expect `getCreatedAt()` to return non-null after persistence.

### UserRepository
- Simple interface extending `JpaRepository<User, Long>`.
- Must define `Optional<User> findByUsername(String username)`.
- Spring Data derives the query automatically.

### JwtTokenProvider
- Tests instantiate directly with `new JwtTokenProvider()` and call `setSecret(String)` / `setExpiration(long)`.
- Must provide public setter methods for `secret` and `expiration` (or use `@Value` + setters).
- Use JJWT 0.12.5 API:
  - Key: `Keys.hmacShaKeyFor(secret.getBytes(StandardCharsets.UTF_8))` -- secret must be >= 32 bytes.
  - Generate: `Jwts.builder().subject(String.valueOf(userId)).issuedAt(now).expiration(expiryDate).signWith(key).compact()`
  - Parse: `Jwts.parser().verifyWith(key).build().parseSignedClaims(token).getPayload()`
  - Store userId in `subject` claim as String, parse back with `Long.parseLong()`.
- `validateToken()` must catch all exceptions (ExpiredJwtException, MalformedJwtException, etc.) and return false.
- `validateToken(null)` and `validateToken("")` must return false.
- For expired token test: expiration=0L means token expires at issuedAt time. Make sure this works.

### JwtAuthenticationFilter
- Extends `OncePerRequestFilter`.
- Tests inject `JwtTokenProvider` via constructor: `new JwtAuthenticationFilter(jwtTokenProvider)`.
- Must have constructor that takes `JwtTokenProvider`.
- `doFilterInternal()` is tested directly (it's protected in OncePerRequestFilter, tests call it because they're in same package or subclass).
- Extract token: check `Authorization` header starts with `"Bearer "`, strip prefix.
- If no header or not Bearer prefix -> skip, just call filterChain.doFilter().
- If token valid -> set `UsernamePasswordAuthenticationToken` with principal = userId (Long).
- If token invalid or exception -> do NOT set auth, just continue filter chain.
- Always call `filterChain.doFilter()` regardless of token validity.

### SecurityConfig Modifications
- Add `JwtAuthenticationFilter` bean or inject it.
- Register with `.addFilterBefore(jwtAuthenticationFilter, UsernamePasswordAuthenticationFilter.class)`.
- The existing config already has correct URL patterns and STATELESS sessions.
- Must handle 401 for unauthenticated requests: add `.exceptionHandling(ex -> ex.authenticationEntryPoint(...))` that returns 401 JSON.
- Without an explicit AuthenticationEntryPoint, Spring Security may return 403 instead of 401 for unauthenticated requests.

### Key Integration Test Expectations
- `GET /api/tasks` without token -> 401 (not 403).
- `GET /api/tasks` with expired/malformed token -> 401.
- `GET /api/tasks` with valid token -> NOT 401/403 (may be 404 if TaskController doesn't exist yet).
- `GET /api/auth/me` -> NOT 403 (auth endpoints are public via SecurityConfig).

## Mock Strategies
- JwtAuthenticationFilterTest: Mock `JwtTokenProvider`, use `MockHttpServletRequest/Response`.
- UserRepositoryTest: `@DataJpaTest` with `TestEntityManager` for persistence.
- JwtTokenProviderTest: Pure unit test, no Spring context, direct instantiation.
- SecurityIntegrationTest: `@SpringBootTest` + `@AutoConfigureMockMvc` for full context.

## Common Pitfall
- Spring Security 6.x returns 403 by default for unauthenticated requests (no AuthenticationEntryPoint configured). You MUST configure an `AuthenticationEntryPoint` that returns 401 to pass the integration tests.
