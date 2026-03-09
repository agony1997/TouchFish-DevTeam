# Test Agent 3 Output — T3: Auth Service + Controller + Error Handling

## Test Files

- `taskflow/backend/src/test/java/com/taskflow/controller/AuthControllerTest.java`

## Test Count: 29

## Test Breakdown

### Register Tests (15 tests)
| Test | AC | Description |
|------|-----|-------------|
| register_validData_returns201 | AC1 | 201 with id, username, displayName |
| register_validData_noPasswordInResponse | AC1 | No password in response |
| register_duplicateUsername_returns409 | AC2 | 409 Conflict on duplicate |
| register_passwordStoredAsBcrypt | AC3 | Password is bcrypt, not plaintext |
| register_usernameTooShort_returns400 | AC11 | username < 3 chars → 400 |
| register_usernameTooLong_returns400 | AC11 | username > 50 chars → 400 |
| register_usernameMinBoundary_returns201 | AC11 | username = 3 chars → 201 |
| register_usernameMaxBoundary_returns201 | AC11 | username = 50 chars → 201 |
| register_passwordTooShort_returns400 | AC12 | password < 8 chars → 400 |
| register_passwordMinBoundary_returns201 | AC12 | password = 8 chars → 201 |
| register_blankDisplayName_returns400 | AC13 | blank displayName → 400 |
| register_displayNameTooLong_returns400 | AC13 | displayName > 100 chars → 400 |
| register_usernameWithSpecialChars_returns400 | AC14 | special chars → 400 |
| register_usernameWithSpaces_returns400 | AC14 | spaces → 400 |
| register_usernameWithUnderscore_returns201 | AC14 | underscore is valid |

### Login Tests (5 tests)
| Test | AC | Description |
|------|-----|-------------|
| login_validCredentials_returns200WithToken | AC4 | 200 with token + expiresIn=86400 |
| login_tokenIsValidJwt | AC4 | Token is valid JWT for user |
| login_wrongPassword_returns401 | AC5 | Wrong password → 401 |
| login_nonExistentUser_returns401 | AC6 | Non-existent user → 401 |
| login_errorMessagesAreIdentical_preventEnumeration | BR7 | Same message for both errors |

### Me Tests (6 tests)
| Test | AC | Description |
|------|-----|-------------|
| me_validToken_returns200 | AC7 | 200 with id, username, displayName |
| me_validToken_noPasswordInResponse | AC7 | No password in response |
| me_noAuthHeader_returns401 | AC8 | No header → 401 |
| me_malformedToken_returns401 | AC10 | Invalid JWT → 401 |
| me_emptyBearer_returns401 | AC10 | Empty bearer → 401 |
| me_noBearerPrefix_returns401 | AC10 | Missing "Bearer " prefix → 401 |

### Error Response Contract Tests (3 tests)
| Test | Description |
|------|-------------|
| validationError_hasCorrectFormat | 400 has {message, timestamp} |
| conflictError_hasCorrectFormat | 409 has {message, timestamp} |
| unauthorizedError_hasCorrectFormat | 401 has {message} |

## Implementation Notes

Written to `docs/dev-team/2026-03-09-E3/temp/impl-notes-3.md`

## Status

All 29 tests compile and run. All 29 fail (expected — AuthController not yet implemented). Red state confirmed.
