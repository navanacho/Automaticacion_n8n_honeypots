## ADDED Requirements

### Requirement: Map honeypot events to MITRE ATT&CK techniques
The system SHALL automatically map detected honeypot events to their corresponding MITRE ATT&CK techniques based on event type, protocol, and behavioral patterns.

#### Scenario: Map login attempt to T1110 Brute Force
- **WHEN** system processes honeypot event with event_type=login_attempt
- **THEN** system maps event to MITRE technique T1110 (Brute Force) and stores mapping

#### Scenario: Map command execution to T1059 Command and Scripting Interpreter
- **WHEN** system processes event with event_type=command_execution
- **THEN** system maps event to MITRE technique T1059 (Command and Scripting Interpreter)

#### Scenario: Event with multiple techniques
- **WHEN** honeypot event contains multiple attack indicators
- **THEN** system creates mapping to all applicable techniques (1:N relationship)

### Requirement: Retrieve MITRE tactic and technique metadata
The system SHALL provide access to complete MITRE ATT&CK framework data including all tactics, techniques, descriptions, and relationships.

#### Scenario: List all MITRE tactics
- **WHEN** client calls GET /api/mitre/tactics
- **THEN** system returns array of 14 tactics: { id, name, description } (e.g., Reconnaissance, Resource Development, Initial Access, Execution, Persistence, Privilege Escalation, Defense Evasion, Credential Access, Discovery, Lateral Movement, Collection, Command and Control, Exfiltration, Impact)

#### Scenario: Get tactics for given technique
- **WHEN** client calls GET /api/mitre/techniques/T1110/tactics
- **THEN** system returns array of tactics that technique T1110 maps to

#### Scenario: Get all techniques for a tactic
- **WHEN** client calls GET /api/mitre/tactics/execution/techniques
- **THEN** system returns array of all techniques under Execution tactic

#### Scenario: Get technique details
- **WHEN** client calls GET /api/mitre/techniques/T1110
- **THEN** response includes: { id, name, description, tactics: [...], subtechniques: [...] }

### Requirement: Generate MITRE heatmap data (tactic × technique)
The system SHALL generate aggregated heatmap data showing frequency of attacks by tactic and technique combination, suitable for visualization.

#### Scenario: Heatmap data for last 24 hours
- **WHEN** client calls GET /api/mitre/heatmap?period=24h
- **THEN** system returns matrix data: { tactics: [...], techniques: [...], counts: [[...]] } where counts[i][j] = number of events mapped to tactic[i] × technique[j]

#### Scenario: Heatmap data for custom date range
- **WHEN** client calls GET /api/mitre/heatmap?startDate=2026-05-01&endDate=2026-05-28
- **THEN** system calculates heatmap for events within specified date range only

#### Scenario: Heatmap excludes simulated events
- **WHEN** client calls GET /api/mitre/heatmap without includeSimulated parameter
- **THEN** heatmap counts exclude events where is_simulated=true

#### Scenario: Heatmap includes simulated events
- **WHEN** client calls GET /api/mitre/heatmap?includeSimulated=true
- **THEN** heatmap counts include all events regardless of is_simulated

### Requirement: Calculate MITRE coverage metrics
The system SHALL calculate and report coverage metrics: total techniques detected, percentage of framework covered, and top tactics/techniques.

#### Scenario: Get coverage summary
- **WHEN** client calls GET /api/mitre/coverage
- **THEN** system returns: { total_techniques_detected, techniques_count, coverage_percentage, top_tactics: [], top_techniques: [] }

#### Scenario: Top tactics for period
- **WHEN** client calls GET /api/mitre/coverage?period=7d
- **THEN** system returns tactics sorted by event count (descending) for last 7 days only

#### Scenario: Coverage for specific honeypot
- **WHEN** client calls GET /api/mitre/coverage?honeypot=cowrie
- **THEN** system returns coverage metrics only for events from cowrie honeypot

### Requirement: Retrieve technique prevalence
The system SHALL calculate and return frequency/prevalence of each MITRE technique based on event data.

#### Scenario: Get technique prevalence ranking
- **WHEN** client calls GET /api/mitre/techniques/prevalence?limit=20
- **THEN** system returns top 20 techniques sorted by event count (descending), including rank, technique_id, name, event_count

#### Scenario: Prevalence for date range
- **WHEN** client calls GET /api/mitre/techniques/prevalence?startDate=2026-05-01&endDate=2026-05-28
- **THEN** prevalence is calculated only from events within date range

#### Scenario: Empty prevalence
- **WHEN** no events have been recorded for a date range
- **THEN** system returns empty array with status 200
