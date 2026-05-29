## ADDED Requirements

### Requirement: REST API endpoints for honeypot events
The system SHALL provide REST API endpoints running on Express.js at localhost:3001 for all CRUD operations on honeypot events.

#### Scenario: GET /api/events returns event list
- **WHEN** client calls GET /api/events
- **THEN** system returns 200 OK with paginated array of events and pagination metadata

#### Scenario: GET /api/events/{id} returns single event
- **WHEN** client calls GET /api/events/{eventId}
- **THEN** system returns 200 OK with complete event object or 404 Not Found

#### Scenario: POST /api/events creates new event
- **WHEN** n8n webhook calls POST /api/events with event data
- **THEN** system creates honeypot_events record, returns 201 Created with event object

#### Scenario: DELETE /api/events/{id} removes event
- **WHEN** authorized user calls DELETE /api/events/{eventId}
- **THEN** system soft-deletes or archives event, returns 204 No Content

### Requirement: Authentication and authorization endpoints
The system SHALL provide endpoints for user login and token validation.

#### Scenario: POST /api/auth/login authenticates user
- **WHEN** client calls POST /api/auth/login with credentials
- **THEN** system validates password and returns JWT token or 401 Unauthorized

#### Scenario: GET /api/auth/verify validates token
- **WHEN** client calls GET /api/auth/verify with Authorization header
- **THEN** system returns 200 OK with token validity status

#### Scenario: POST /api/auth/logout invalidates token
- **WHEN** client calls POST /api/auth/logout with valid token
- **THEN** system invalidates token (optional, JWT usually client-side managed)

### Requirement: Metrics and analysis endpoints
The system SHALL provide endpoints for retrieving calculated metrics and MITRE analysis data.

#### Scenario: GET /api/metrics returns summary metrics
- **WHEN** client calls GET /api/metrics
- **THEN** system returns MTTD, MTTR, counts, and other aggregated metrics

#### Scenario: GET /api/mitre/* returns MITRE data
- **WHEN** client calls GET /api/mitre/tactics, /api/mitre/techniques, /api/mitre/heatmap
- **THEN** system returns MITRE ATT&CK data or analysis results

### Requirement: Simulation endpoint
The system SHALL provide endpoint to trigger attack simulations (real via n8n or fake).

#### Scenario: POST /api/simulate creates simulation
- **WHEN** client calls POST /api/simulate with simulation parameters
- **THEN** system executes simulation and returns 201 Created or 202 Accepted with event details

#### Scenario: GET /api/simulations lists simulation history
- **WHEN** client calls GET /api/simulations
- **THEN** system returns paginated list of past simulations with status and results

### Requirement: Webhook endpoint for n8n integration
The system SHALL provide endpoint to receive events from n8n honeypot workflows.

#### Scenario: POST /api/webhooks/events receives honeypot events
- **WHEN** n8n sends event to POST /api/webhooks/events
- **THEN** system stores event in PostgreSQL, broadcasts via WebSocket, returns 200 OK

#### Scenario: Webhook idempotency
- **WHEN** n8n sends duplicate event (with same idempotency key)
- **THEN** system detects duplicate, returns 200 OK but doesn't create new event

#### Scenario: Webhook validation
- **WHEN** POST /api/webhooks/events receives request
- **THEN** system validates required fields (timestamp, source_ip, honeypot, event_type), returns 400 Bad Request if missing

### Requirement: JWT validation middleware
The system SHALL apply JWT validation to all protected endpoints before handler execution.

#### Scenario: Middleware extracts JWT from Authorization header
- **WHEN** request contains Authorization: Bearer <token>
- **THEN** middleware extracts token, verifies signature and expiration

#### Scenario: Middleware attaches user to request
- **WHEN** JWT is valid
- **THEN** middleware sets req.user = { userId, username } for handler access

#### Scenario: Middleware rejects invalid/expired tokens
- **WHEN** JWT is invalid, expired, or missing from protected route
- **THEN** middleware returns 401 Unauthorized before handler executes

#### Scenario: Middleware skips public endpoints
- **WHEN** request is to public endpoint (POST /api/auth/login, POST /api/webhooks/events)
- **THEN** middleware allows request through without token validation

### Requirement: Error handling and responses
The system SHALL handle errors consistently and return appropriate HTTP status codes with descriptive error messages.

#### Scenario: 400 Bad Request for invalid input
- **WHEN** client sends malformed JSON or missing required fields
- **THEN** system returns 400 Bad Request with { error: "description", code: "INVALID_INPUT" }

#### Scenario: 401 Unauthorized for auth failures
- **WHEN** request lacks valid authentication (missing/expired JWT)
- **THEN** system returns 401 Unauthorized with { error: "description", code: "UNAUTHORIZED" }

#### Scenario: 403 Forbidden for insufficient permissions
- **WHEN** authenticated user lacks permission for resource
- **THEN** system returns 403 Forbidden with { error: "description", code: "FORBIDDEN" }

#### Scenario: 404 Not Found for missing resources
- **WHEN** client requests non-existent resource
- **THEN** system returns 404 Not Found with { error: "Resource not found" }

#### Scenario: 500 Server Error for unhandled exceptions
- **WHEN** unexpected error occurs during request processing
- **THEN** system returns 500 Internal Server Error, logs error details, returns generic message to client

#### Scenario: Error response format
- **WHEN** any error occurs
- **THEN** response includes: { error, code, status, timestamp }

### Requirement: Request/response logging
The system SHALL log all HTTP requests and responses for debugging and audit purposes.

#### Scenario: Log successful requests
- **WHEN** request completes successfully
- **THEN** system logs: timestamp, method, path, status, response_time_ms

#### Scenario: Log errors
- **WHEN** request results in error
- **THEN** system logs: timestamp, method, path, status, error_message, stack_trace (if 5xx)

#### Scenario: Log authentication attempts
- **WHEN** login is attempted (success or failure)
- **THEN** system logs: timestamp, username, ip_address, status, failure_reason (if failed)

#### Scenario: Logs are persisted
- **WHEN** requests are processed
- **THEN** logs are written to both console (development) and file (logs/requests.log)

### Requirement: CORS configuration
The system SHALL enable Cross-Origin Resource Sharing (CORS) to allow React frontend requests.

#### Scenario: CORS headers on responses
- **WHEN** browser makes cross-origin request from http://localhost:3000
- **THEN** system includes Access-Control-Allow-Origin: http://localhost:3000 header

#### Scenario: CORS preflight requests
- **WHEN** browser sends OPTIONS preflight request
- **THEN** system returns 200 OK with appropriate CORS headers

#### Scenario: Credentials in CORS
- **WHEN** frontend sends request with credentials (cookies/auth headers)
- **THEN** system includes Access-Control-Allow-Credentials: true

#### Scenario: CORS whitelist in production
- **WHEN** environment is production
- **THEN** CORS only allows whitelisted origins from .env (not localhost)

### Requirement: API versioning (optional)
The system SHALL support API versioning for future compatibility.

#### Scenario: Current API version
- **WHEN** client calls any endpoint
- **THEN** response includes X-API-Version: 1.0 header

#### Scenario: Deprecation warnings
- **WHEN** endpoint is planned for deprecation
- **THEN** response includes X-Deprecated-After: <date> header

### Requirement: Rate limiting (optional)
The system MAY implement rate limiting on sensitive endpoints (login, simulate).

#### Scenario: Rate limit on login
- **WHEN** client makes >5 login attempts within 5 minutes from same IP
- **THEN** subsequent requests return 429 Too Many Requests

#### Scenario: Rate limit headers
- **WHEN** rate limit is approaching
- **THEN** response includes X-RateLimit-Limit, X-RateLimit-Remaining, X-RateLimit-Reset headers
