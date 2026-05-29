## ADDED Requirements

### Requirement: Login with username and password
The system SHALL provide a login endpoint that accepts username and password credentials, validates them against stored hashes, and returns a JWT token on success.

#### Scenario: Successful login
- **WHEN** client calls POST /api/auth/login with { username: "admin", password: "correct_password" }
- **THEN** system verifies password against bcrypt hash, returns 200 OK with { token: "eyJ...", expiresIn: 86400, user: { id, username } }

#### Scenario: Incorrect password
- **WHEN** client calls POST /api/auth/login with { username: "admin", password: "wrong_password" }
- **THEN** system returns 401 Unauthorized with error message "Invalid credentials"

#### Scenario: User not found
- **WHEN** client calls POST /api/auth/login with { username: "nonexistent", password: "any_password" }
- **THEN** system returns 401 Unauthorized with error message "Invalid credentials" (no user enumeration)

#### Scenario: Missing credentials
- **WHEN** client calls POST /api/auth/login with missing username or password field
- **THEN** system returns 400 Bad Request with error message listing required fields

### Requirement: JWT token generation
The system SHALL generate JWT tokens with configurable expiration, signed with secret from .env file, containing user identifier.

#### Scenario: Token contains user information
- **WHEN** login succeeds
- **THEN** JWT payload includes: { userId, username, iat (issued at), exp (expiration timestamp) }

#### Scenario: Token signed with secret
- **WHEN** JWT is generated
- **THEN** token is signed using JWT_SECRET from .env file with HS256 algorithm

#### Scenario: Token expires after 24 hours
- **WHEN** JWT is generated at time T
- **THEN** token exp claim equals T + 86400 seconds (24 hours)

#### Scenario: Token structure is valid
- **WHEN** JWT is generated
- **THEN** token follows format: header.payload.signature (3 Base64URL parts separated by dots)

### Requirement: JWT validation for protected endpoints
The system SHALL validate JWT tokens on all protected API endpoints, rejecting requests with missing, expired, or invalid tokens.

#### Scenario: Request with valid token
- **WHEN** client calls GET /api/events with Authorization header "Bearer eyJ..."
- **THEN** system extracts token, verifies signature and expiration, allows request to proceed

#### Scenario: Request with expired token
- **WHEN** client calls GET /api/events with Authorization header containing expired token
- **THEN** system returns 401 Unauthorized with error message "Token expired"

#### Scenario: Request with invalid token signature
- **WHEN** client calls GET /api/events with Authorization header containing token signed with wrong secret
- **THEN** system returns 401 Unauthorized with error message "Invalid token"

#### Scenario: Request with missing Authorization header
- **WHEN** client calls protected endpoint GET /api/events without Authorization header
- **THEN** system returns 401 Unauthorized with error message "Missing authorization token"

#### Scenario: Request with malformed Authorization header
- **WHEN** client calls GET /api/events with Authorization header "InvalidFormat token"
- **THEN** system returns 401 Unauthorized with error message "Invalid authorization header format"

### Requirement: Bearer token extraction
The system SHALL extract JWT from Authorization header using Bearer scheme (RFC 6750).

#### Scenario: Extract token from Bearer scheme
- **WHEN** Authorization header is "Bearer eyJ..."
- **THEN** system extracts "eyJ..." and validates it

#### Scenario: Case insensitive Bearer prefix
- **WHEN** Authorization header is "bearer eyJ..." or "BEARER eyJ..."
- **THEN** system correctly extracts token regardless of case

### Requirement: Password security requirements
The system SHALL store passwords using bcrypt hashing with minimum strength and enforce complexity rules.

#### Scenario: Password hashing
- **WHEN** user account is created with password
- **THEN** system stores bcrypt hash (cost factor ≥10) instead of plaintext

#### Scenario: Password not retrievable
- **WHEN** password is hashed with bcrypt
- **THEN** system cannot reverse hash to retrieve original password, only verify against hash

### Requirement: User session tracking
The system SHALL track active user sessions and log authentication events for audit purposes.

#### Scenario: Record successful login
- **WHEN** login succeeds
- **THEN** system records in user_sessions table: user_id, login_timestamp, ip_address, user_agent

#### Scenario: Record failed login attempt
- **WHEN** login fails
- **THEN** system logs failed attempt with username, timestamp, ip_address for security audit

### Requirement: Token validation middleware
The system SHALL provide reusable middleware that validates JWT for all protected routes.

#### Scenario: Middleware attaches user to request
- **WHEN** JWT is validated successfully
- **THEN** middleware attaches user object to request context (req.user) with userId and username

#### Scenario: Middleware rejects invalid token
- **WHEN** JWT is invalid or missing
- **THEN** middleware returns 401 Unauthorized before reaching route handler

#### Scenario: Middleware applied to all protected routes
- **WHEN** client calls any protected endpoint
- **THEN** middleware is invoked to validate token before handler executes
