## ADDED Requirements

### Requirement: List honeypot events with pagination
The system SHALL provide an endpoint to retrieve paginated list of honeypot events from the database, supporting filtering by IP address, date range, event type, and honeypot source.

#### Scenario: Fetch events with default pagination
- **WHEN** client calls GET /api/events with no query parameters
- **THEN** system returns array of up to 50 events sorted by timestamp descending, with pagination metadata (page, limit, total)

#### Scenario: Filter events by source IP
- **WHEN** client calls GET /api/events?ip=192.168.1.100
- **THEN** system returns only events where source_ip matches 192.168.1.100

#### Scenario: Filter events by date range
- **WHEN** client calls GET /api/events?startDate=2026-05-01&endDate=2026-05-28
- **THEN** system returns only events where timestamp is between start and end date (inclusive)

#### Scenario: Filter events by type
- **WHEN** client calls GET /api/events?eventType=login_attempt
- **THEN** system returns only events of specified type (login_attempt, command_execution, file_access)

#### Scenario: Filter by honeypot source
- **WHEN** client calls GET /api/events?honeypot=cowrie
- **THEN** system returns only events from specified honeypot (cowrie or dionaea)

#### Scenario: Exclude simulated events by default
- **WHEN** client calls GET /api/events without includeSimulated parameter
- **THEN** system filters out events where is_simulated=true

#### Scenario: Include simulated events explicitly
- **WHEN** client calls GET /api/events?includeSimulated=true
- **THEN** system includes events where is_simulated=true

### Requirement: Full-text search on events
The system SHALL provide full-text search capability across event fields (IP, event_type, protocol, command data if available).

#### Scenario: Search by IP address
- **WHEN** client calls GET /api/events/search?q=192.168
- **THEN** system returns events matching IP pattern in source_ip field

#### Scenario: Search by event type
- **WHEN** client calls GET /api/events/search?q=ssh
- **THEN** system returns events containing "ssh" in protocol or event_type

#### Scenario: Empty search results
- **WHEN** client calls GET /api/events/search?q=nonexistent_pattern
- **THEN** system returns empty array with status 200

### Requirement: Get event details
The system SHALL return complete event object with all fields including timestamp, source IP, honeypot type, event type, and risk score.

#### Scenario: Retrieve single event details
- **WHEN** client calls GET /api/events/{eventId}
- **THEN** system returns event object with fields: id, timestamp, source_ip, destination_port, honeypot, event_type, protocol, risk_score, is_simulated, created_at

#### Scenario: Event not found
- **WHEN** client calls GET /api/events/{invalidEventId}
- **THEN** system returns 404 Not Found with error message

### Requirement: Get metrics across events
The system SHALL calculate and return aggregate metrics: total event count, unique source IPs, and distribution by event type.

#### Scenario: Retrieve event metrics
- **WHEN** client calls GET /api/metrics/events
- **THEN** system returns: total_count, unique_ips, event_type_distribution (object with counts per type)

#### Scenario: Metrics for date range
- **WHEN** client calls GET /api/metrics/events?startDate=2026-05-01&endDate=2026-05-28
- **THEN** system calculates metrics only for events within date range

### Requirement: Event response format
The system SHALL return events with consistent JSON schema including optional fields for enrichment.

#### Scenario: Event JSON structure
- **WHEN** client retrieves any event via GET /api/events/{eventId}
- **THEN** response includes: { id, timestamp, source_ip, destination_port, honeypot, event_type, protocol, risk_score, techniques (array of MITRE technique IDs), is_simulated, created_at }

#### Scenario: Pagination metadata
- **WHEN** client calls any listing endpoint GET /api/events
- **THEN** response wrapper includes: { data: [...], pagination: { page, limit, total, hasMore } }
