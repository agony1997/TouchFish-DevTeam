# Implementation Notes — T2: JWT Security Infrastructure

## JwtUtil

- **Constructor**: Must accept `(String secret, long expiration)`. Tests instantiate directly with `new JwtUtil(secret, expiration)`. Map `@Value("${app.jwt.secret}")` and `@Value("${app.jwt.expiration}")` in production.
- **HMAC key**: jjwt 0.12.x uses `Keys.hmacShaKeyFor(secret.getBytes())` — secret must be >= 32 bytes for HS256. The test secret is 56 chars.
- **Claims**: Store `userId` as the JWT subject (`setSubject(String.valueOf(userId))`), then parse back with `Long.parseLong(subject)`.
- **validateToken(null)**: Must handle null input gracefully (return false, not NPE). Add a null check before parsing.
- **Expiration 0ms**: `new Date(now + 0)` means the token expires at creation time. Token may or may not be valid in the same millisecond — tests expect `false`. Use `< now` check or let jjwt's built-in expiration handling catch it.
- **getUserIdFromToken with invalid/expired token**: Tests expect an exception to be thrown (any `Exception` subclass). jjwt throws `JwtException` subclasses — let them propagate.

## JwtAuthenticationFilter

- **Extends**: `OncePerRequestFilter` (Spring framework class)
- **Constructor injection**: `new JwtAuthenticationFilter(JwtUtil jwtUtil)` — tests mock JwtUtil
- **doFilterInternal**: This is a `protected` method on `OncePerRequestFilter`. Tests call it directly since the test class is in the same package-access scope. The method signature is `doFilterInternal(HttpServletRequest, HttpServletResponse, FilterChain)`.
- **Bearer parsing**: Extract token from `Authorization: Bearer <token>`. If header is missing, not "Bearer " prefix, or token is empty after "Bearer " → skip auth, just call `filterChain.doFilter()`.
- **SecurityContext**: On valid token, create `UsernamePasswordAuthenticationToken` with `principal = userId (Long)`, credentials = null, authorities = empty list. Set it via `SecurityContextHolder.getContext().setAuthentication(auth)`.
- **Exception handling**: If `jwtUtil.validateToken()` throws, catch the exception, do NOT set SecurityContext, still call `filterChain.doFilter()`.
- **Must be a @Component**: SecurityConfig injects it via constructor.

## UserResponse

- **Implement as Java record**: `public record UserResponse(Long id, String username, String displayName) {}`. Tests call `.id()`, `.username()`, `.displayName()` — these are record accessor methods.

## File List (3 files in T2 scope)
1. `src/main/java/com/taskflow/security/JwtUtil.java`
2. `src/main/java/com/taskflow/security/JwtAuthenticationFilter.java`
3. `src/main/java/com/taskflow/dto/UserResponse.java` (if not already created by T1)
