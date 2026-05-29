## ADDED Requirements

### Requirement: Calculate Mean Time to Detection (MTTD)
The system SHALL calculate MTTD as the average time between when an attack event occurred (event timestamp) and when it was first detected/alerted by the honeypot system.

#### Scenario: MTTD for all events
- **WHEN** client calls GET /api/metrics/mttd
- **THEN** system calculates average of (event_timestamp - first_alert_timestamp) across all events, returns { mttd_seconds, mttd_human_readable: "5 minutes 30 seconds" }

#### Scenario: MTTD for date range
- **WHEN** client calls GET /api/metrics/mttd?startDate=2026-05-01&endDate=2026-05-28
- **THEN** system calculates MTTD only for events within date range

#### Scenario: MTTD by honeypot
- **WHEN** client calls GET /api/metrics/mttd?honeypot=cowrie
- **THEN** system returns separate MTTD values for cowrie events only

#### Scenario: MTTD excludes simulated events
- **WHEN** client calls GET /api/metrics/mttd without includeSimulated
- **THEN** calculation excludes events where is_simulated=true

#### Scenario: MTTD handles missing data
- **WHEN** events lack first_alert timestamp
- **THEN** system uses event creation timestamp as fallback for calculation

### Requirement: Calculate Mean Time to Resolution (MTTR)
The system SHALL calculate MTTR as the average time between when an attack is detected and when it is resolved/mitigated.

#### Scenario: MTTR for all events
- **WHEN** client calls GET /api/metrics/mttr
- **THEN** system calculates average of (resolved_timestamp - first_alert_timestamp) for resolved events, returns { mttr_seconds, mttr_human_readable }

#### Scenario: MTTR for date range
- **WHEN** client calls GET /api/metrics/mttr?startDate=2026-05-01&endDate=2026-05-28
- **THEN** system calculates MTTR only for events resolved within date range

#### Scenario: MTTR by event type
- **WHEN** client calls GET /api/metrics/mttr?eventType=login_attempt
- **THEN** system returns MTTR for only login_attempt events

#### Scenario: MTTR with unresolved events
- **WHEN** some events have no resolved_timestamp
- **THEN** system excludes unresolved events from MTTR calculation, reports count of unresolved

#### Scenario: MTTR excludes simulated
- **WHEN** client calls GET /api/metrics/mttr without includeSimulated
- **THEN** calculation excludes is_simulated=true events

### Requirement: Aggregate event counts
The system SHALL provide counts of total events, unique source IPs, and distribution by event type/honeypot.

#### Scenario: Total event count
- **WHEN** client calls GET /api/metrics/counts
- **THEN** system returns { total_events, total_events_real, total_events_simulated }

#### Scenario: Event counts for last 24 hours
- **WHEN** client calls GET /api/metrics/counts?period=24h
- **THEN** system returns counts for events with timestamp >= now() - 24 hours

#### Scenario: Unique source IPs
- **WHEN** client calls GET /api/metrics/counts
- **THEN** response includes: { unique_source_ips, unique_source_ips_list: ["192.168.1.1", ...] }

#### Scenario: Distribution by event type
- **WHEN** client calls GET /api/metrics/counts
- **THEN** response includes event_type_distribution: { login_attempt: 100, command_execution: 50, file_access: 25 }

#### Scenario: Distribution by honeypot
- **WHEN** client calls GET /api/metrics/counts
- **THEN** response includes honeypot_distribution: { cowrie: 120, dionaea: 55 }

#### Scenario: Distribution by hour
- **WHEN** client calls GET /api/metrics/counts?granularity=hourly
- **THEN** response includes hourly_distribution: { "2026-05-28T00": 10, "2026-05-28T01": 15, ... }

### Requirement: Dashboard summary metrics
The system SHALL provide a consolidated endpoint returning all key metrics for dashboard display.

#### Scenario: Dashboard metrics endpoint
- **WHEN** client calls GET /api/metrics/dashboard
- **THEN** system returns: { mttd, mttr, counts, coverage, top_techniques, event_trend_24h }

#### Scenario: Event trend over time
- **WHEN** dashboard metrics are calculated
- **THEN** response includes event_trend_24h: hourly counts for last 24 hours for trend graph

#### Scenario: Top techniques detected
- **WHEN** client calls GET /api/metrics/dashboard
- **THEN** response includes top_techniques: [ { id: "T1110", name: "Brute Force", count: 45 }, ... ]

#### Scenario: Dashboard metrics are cacheable
- **WHEN** client calls GET /api/metrics/dashboard
- **THEN** response includes Cache-Control: max-age=300 (5 minute cache)

### Requirement: Custom metric queries
The system SHALL allow construction of custom metric queries with multiple filters for advanced analysis.

#### Scenario: Metrics filtered by IP
- **WHEN** client calls GET /api/metrics?ip=192.168.1.100
- **THEN** system calculates all metrics (MTTD, MTTR, counts) for only this IP

#### Scenario: Metrics filtered by technique
- **WHEN** client calls GET /api/metrics?technique=T1110
- **WHEN** system calculates metrics for only events mapped to T1110

#### Scenario: Combined filters
- **WHEN** client calls GET /api/metrics?honeypot=cowrie&startDate=2026-05-01&endDate=2026-05-28&excludeSimulated=true
- **THEN** system returns metrics with all filters applied (AND logic)

### Requirement: Metrics data format
The system SHALL return metrics in standardized JSON format with consistent field naming and units.

#### Scenario: Time values in seconds
- **WHEN** metrics are calculated
- **THEN** time values (mttd_seconds, mttr_seconds) are integers representing total seconds

#### Scenario: Percentages as decimals
- **WHEN** percentage metrics are returned
- **THEN** values are between 0.0 and 1.0 (e.g., coverage_percentage: 0.45 for 45%)

#### Scenario: Timestamps in ISO 8601
- **WHEN** metrics include timestamp fields
- **THEN** timestamps are formatted as ISO 8601 strings (e.g., "2026-05-28T14:30:00Z")

#### Scenario: Large counts formatted
- **WHEN** event counts are very large
- **THEN** JSON includes numeric value and optional human_readable field (e.g., "1.2M events")
