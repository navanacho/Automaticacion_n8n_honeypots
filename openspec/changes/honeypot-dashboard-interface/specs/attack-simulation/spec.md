## ADDED Requirements

### Requirement: Simulate attack via real n8n workflow
The system SHALL allow users to trigger real attack simulations by invoking n8n webhooks, which execute actual honeypot workflows and return captured events.

#### Scenario: Trigger real simulation on n8n
- **WHEN** client calls POST /api/simulate with { type: "brute_force", mode: "real" }
- **THEN** system calls n8n webhook endpoint with simulation parameters and waits for response (5s timeout)

#### Scenario: n8n returns simulated event
- **WHEN** n8n webhook successfully executes
- **THEN** system receives event data and stores it in honeypot_events table with is_simulated=true, timestamp=current_time

#### Scenario: n8n timeout fallback to fake event
- **WHEN** n8n webhook does not respond within 5 seconds
- **THEN** system automatically creates fake event with random IP, current timestamp, and specified type, logs the n8n failure

#### Scenario: n8n returns error
- **WHEN** n8n webhook returns HTTP 5xx or invalid response
- **THEN** system creates fallback fake event, logs error with n8n response code, returns 202 Accepted to client

#### Scenario: Record real simulation attempt
- **WHEN** simulation mode is "real"
- **THEN** system records attempt in attack_simulations table with status=requested, n8n_workflow_id, timestamp_requested

#### Scenario: Update simulation status on n8n response
- **WHEN** n8n webhook responds
- **THEN** system updates attack_simulations record with status=completed or status=failed_fallback, timestamp_completed, n8n_response_data

### Requirement: Simulate attack with fake event data
The system SHALL allow users to create fake attack events directly in the database without invoking n8n, useful for testing UI and alerts.

#### Scenario: Create simulated login attempt
- **WHEN** client calls POST /api/simulate with { type: "brute_force", mode: "simulated" }
- **THEN** system creates honeypot_events record with: source_ip=random(10.0.0.0/8), event_type=login_attempt, timestamp=now(), honeypot=random(cowrie|dionaea), is_simulated=true, protocol=ssh

#### Scenario: Create simulated command execution
- **WHEN** client calls POST /api/simulate with { type: "command_execution", mode: "simulated" }
- **THEN** system creates event with event_type=command_execution, timestamp=now(), maps to T1059 technique, is_simulated=true

#### Scenario: Create simulated file access
- **WHEN** client calls POST /api/simulate with { type: "file_access", mode: "simulated" }
- **THEN** system creates event with event_type=file_access, timestamp=now(), maps to T1083 technique, is_simulated=true

#### Scenario: Random technique selection
- **WHEN** client calls POST /api/simulate with { mode: "simulated", randomTechnique: true }
- **THEN** system selects random MITRE technique from all available, creates event mapped to that technique

#### Scenario: Record simulated event
- **WHEN** simulation mode is "simulated"
- **THEN** system records entry in attack_simulations table with status=completed, mode=simulated, source=local

### Requirement: Simulation request validation
The system SHALL validate all simulation requests and return appropriate error codes for invalid input.

#### Scenario: Missing required parameters
- **WHEN** client calls POST /api/simulate with missing type or mode
- **THEN** system returns 400 Bad Request with error message listing missing fields

#### Scenario: Invalid simulation type
- **WHEN** client calls POST /api/simulate with type=invalid_type
- **THEN** system returns 400 Bad Request with list of valid types (brute_force, command_execution, file_access)

#### Scenario: Invalid mode
- **WHEN** client calls POST /api/simulate with mode=invalid_mode
- **THEN** system returns 400 Bad Request indicating valid modes are "real" or "simulated"

#### Scenario: Unauthorized simulation request
- **WHEN** client calls POST /api/simulate without valid JWT token
- **THEN** system returns 401 Unauthorized

### Requirement: Simulation response format
The system SHALL return standardized responses for all simulation requests indicating success and event details.

#### Scenario: Successful real simulation response
- **WHEN** real simulation completes (success or fallback)
- **THEN** system returns 202 Accepted with: { eventId, mode: "real", status: "completed|fallback", created_event: {...}, n8n_status: "success|timeout|error" }

#### Scenario: Successful fake simulation response
- **WHEN** fake simulation completes
- **THEN** system returns 201 Created with: { eventId, mode: "simulated", status: "completed", created_event: {...} }

#### Scenario: Simulation metadata in response
- **WHEN** any simulation completes
- **THEN** response includes: simulation_id, timestamp_requested, timestamp_completed, duration_ms

### Requirement: Idempotent simulation requests
The system SHALL handle duplicate simulation requests gracefully using idempotency keys to prevent duplicate events.

#### Scenario: Idempotency with key parameter
- **WHEN** client calls POST /api/simulate with idempotencyKey=ABC123
- **THEN** system checks if simulation with this key was processed in last 24 hours

#### Scenario: Duplicate request returns cached result
- **WHEN** client resends POST /api/simulate with same idempotencyKey within 24 hours
- **THEN** system returns 200 OK with previously created event, does not create new event

#### Scenario: Idempotency key expires
- **WHEN** client calls POST /api/simulate with idempotencyKey used >24 hours ago
- **THEN** system treats as new request and creates new event

### Requirement: Simulation audit trail
The system SHALL maintain complete history of all simulation requests with timestamps, user information, and outcomes.

#### Scenario: Record simulation attempt
- **WHEN** simulation completes
- **THEN** system stores in attack_simulations table: user_id, timestamp_requested, mode, type, status, event_id, n8n_response, fallback_reason (if applicable)

#### Scenario: Query simulation history
- **WHEN** client calls GET /api/simulations?limit=50
- **THEN** system returns array of simulation records sorted by timestamp descending
