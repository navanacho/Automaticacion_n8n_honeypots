<!-- gentle-ai:persona -->
# Automaticacion_n8n_honeypots — Agents Configuration

## Project Overview

**Name**: Automaticacion_n8n_honeypots  
**Description**: Enterprise-grade honeypot automation and threat intelligence system using n8n orchestration with real-time attack capture, analysis, and MITRE ATT&CK mapping.

**Vision**: Reduce MTTD (Mean Time To Detect) and MTTR (Mean Time To Respond) by automating honeypot event capture, enrichment, and tactical intelligence correlation.

### Tech Stack

- **Frontend**: React 18+, TypeScript, Tailwind CSS
- **Backend**: Express.js, TypeScript, Node.js
- **Database**: PostgreSQL (relational event/threat storage)
- **Orchestration**: n8n (SOAR automation workflows)
- **Honeypots**: Cowrie (SSH/Telnet), Dionaea (protocol capture)
- **Infrastructure**: Docker, Docker Compose
- **Threat Intel**: MITRE ATT&CK mapping, IP enrichment
- **Security**: Network segmentation, secret management, centralized logging

### Core Values

- **SOLID Architecture**: Layered, testable, maintainable code with clear boundaries
- **Security-First**: Threat modeling, sanitization, encryption in transit/at-rest
- **Testing-Focused**: Unit, integration, and E2E coverage with automated validation
- **Observable**: Structured logging, metrics, audit trails for forensics
- **Automation**: Infrastructure-as-Code, CI/CD, repeatability

---

## Agent Assignments

### Model-to-Task Mapping

When a task needs execution, use this matrix to determine which agent model is appropriate:

| Task Category | Recommended Model | When to Use | Example |
|---|---|---|---|
| **Database Schema & Migrations** | claude-opus-4 | Creating tables, indexes, foreign keys; complex queries; data migrations | Adding `events_correlation` table, optimizing honeypot_sessions index |
| **React Frontend Development** | claude-opus-4 | Component design, state management, accessibility, styling logic | Building ThreatMap dashboard, implementing filters |
| **Express API Development** | claude-opus-4 | Endpoint logic, middleware, request validation, error handling | Creating `/api/threats` POST endpoint, auth middleware |
| **n8n Workflow Automation** | claude-haiku-4.5 | Simple workflow nodes, basic JSON transformations, condition logic | Webhook receiver for Cowrie events, Dionaea integration |
| **Testing & Unit Coverage** | claude-haiku-4.5 | Writing test cases, mocks, assertions | Jest tests for utility functions, middleware tests |
| **Integration Testing** | claude-opus-4 | E2E flows, cross-service scenarios, complex workflows | Testing full event capture → enrichment → classification flow |
| **Documentation & Specs** | claude-haiku-4.5 | README updates, comment clarity, spec clarification | API documentation, workflow diagrams |
| **Refactoring & Optimization** | claude-opus-4 | Performance profiling, architectural improvements, dependency cleanup | Reducing N+1 queries, consolidating event processors |
| **Security Reviews & Audits** | claude-opus-4 | Threat modeling, credential handling, SQL injection prevention, RBAC design | Reviewing authentication flow, validating input sanitization |

---

## Skills Section

Each skill is documented with triggers, usage, and examples.

### 1. openspec-apply-change

**Status**: ✅ Installed locally  
**Location**: `.opencode/skills/openspec-apply-change/`

**Purpose**: Execute implementation tasks from an OpenSpec change. This is where CODE gets written.

**When to Use**:
- After `openspec-propose` creates the initial proposal
- After `openspec-spec` and `openspec-design` are written and approved
- When starting fresh implementation tasks
- When resuming implementation of a partially-completed change

**What It Does**:
- Fetches task checklist from OpenSpec
- Implements each task incrementally
- Marks tasks as complete as they're finished
- Validates against specs and design
- Creates intermediate commits

**Example Usage**:
```bash
openspec apply-change --change "threat-enrichment-api"
```

**Agent Workflow**:
1. Load skill with `skill openspec-apply-change`
2. Agent fetches pending tasks
3. For each task:
   - Read related spec/design
   - Implement in code
   - Write tests
   - Mark complete

---

### 2. openspec-design

**Status**: 🔧 To Install  
**Location**: `~/.config/opencode/skills/openspec-design/`

**Purpose**: Create technical design documents that answer HOW the system will be built — architecture decisions, data models, component structure, API contracts.

**When to Use**:
- AFTER specs are written (design depends on requirements)
- When architecture decisions need to be documented
- Before implementation starts
- When onboarding new team members to understand the "why" of decisions

**What It Does**:
- Creates design artifacts in `openspec/designs/`
- Documents architecture patterns
- Specifies data models and relationships
- Defines component boundaries
- Explains non-obvious decisions

**Example Usage**:
```bash
# Create design for a change
openspec design --change "honeypot-event-aggregator"
```

**Design Template Covers**:
- Architecture Overview (diagrams, patterns)
- Components (responsibility, location, interface)
- Data Model (schemas, types, migrations)
- API Changes (endpoints, contracts, examples)
- Implementation Notes (rationale, gotchas)
- Risks & Mitigations

**Agent Workflow**:
1. Load skill: `skill openspec-design`
2. Agent reads proposal + specs
3. Examines current codebase architecture
4. Proposes design with rationale
5. Documents decisions, not just what but WHY

---

### 3. openspec-spec

**Status**: 🔧 To Install  
**Location**: `~/.config/opencode/skills/openspec-spec/`

**Purpose**: Write delta specifications that define WHAT the system must do — acceptance criteria, behavioral requirements, testable scenarios.

**When to Use**:
- AFTER `openspec-propose` creates the proposal
- Before design or implementation
- When requirements need clarification
- When you need acceptance criteria for QA

**What It Does**:
- Creates spec artifacts in `openspec/specs/`
- Documents requirements using RFC 2119 keywords (MUST, SHALL, SHOULD, MAY)
- Provides Given/When/Then scenarios
- Defines edge cases and error conditions
- Establishes testable acceptance criteria

**Example Usage**:
```bash
# Write specs for a change
openspec spec --change "threat-correlation-engine"
```

**Spec Structure**:
- Overview (brief description)
- Requirements (numbered, with RFC keywords)
- Scenarios (Given/When/Then format)
- Edge Cases & Error Handling

**Agent Workflow**:
1. Load skill: `skill openspec-spec`
2. Agent reads proposal
3. Defines testable requirements
4. Writes scenarios for QA
5. Spec becomes contract for design & implementation

---

### 4. openspec-tasks

**Status**: 🔧 To Install  
**Location**: `~/.config/opencode/skills/openspec-tasks/`

**Purpose**: Create implementation task checklists from specs and design. Break down work into actionable items.

**When to Use**:
- After specs and design are approved
- Before starting implementation
- When team needs clear task breakdown
- For sprint planning

**What It Does**:
- Creates task artifacts in `openspec/tasks/`
- Breaks specs into discrete, completable tasks
- Links tasks to design components
- Estimates complexity
- Creates tracking checklist

**Example Usage**:
```bash
# Create tasks from specs
openspec tasks --change "event-enrichment-service"
```

**Task Breakdown Includes**:
- Per-component implementation tasks
- Database migration tasks
- API endpoint implementation
- Frontend component tasks
- Test coverage tasks
- Documentation tasks

**Agent Workflow**:
1. Load skill: `skill openspec-tasks`
2. Agent reads specs + design
3. Creates granular tasks
4. Links to code locations
5. Estimates effort per task

---

### 5. openspec-verify

**Status**: 🔧 To Install  
**Location**: `~/.config/opencode/skills/openspec-verify/`

**Purpose**: Validate that implementation matches specs, design, and tasks. Quality gate before merge.

**When to Use**:
- After implementation is complete
- Before creating pull request
- When code review finds issues
- To confirm all tasks are done

**What It Does**:
- Compares code against specs
- Validates design decisions were followed
- Checks all tasks are marked complete
- Runs automated checks
- Generates verification report

**Example Usage**:
```bash
# Verify implementation completeness
openspec verify --change "event-aggregator"
```

**Verification Checks**:
- All tasks completed
- Code matches design
- Tests cover requirements
- No dangling TODOs
- Documentation updated

**Agent Workflow**:
1. Load skill: `skill openspec-verify`
2. Agent compares implementation to artifacts
3. Validates test coverage
4. Confirms design compliance
5. Reports pass/fail with details

---

### 6. skill-creator

**Status**: ✅ Available  
**Location**: `~/.config/opencode/skills/skill-creator/`

**Purpose**: Create new AI agent skills following Agent Skills spec. Use when establishing patterns for recurring tasks.

**When to Use**:
- When a workflow repeats across many changes
- When need to document specialized patterns
- When creating team conventions
- When establishing project-specific automation

**What It Does**:
- Scaffolds new skill structure
- Generates SKILL.md template
- Creates skill metadata
- Documents patterns and conventions

**Example Future Usage**:
```bash
# Create a skill for Cowrie integration pattern
skill create --name "cowrie-event-integration" --type integration
```

---

## Workflow Section

### OpenSpec Development Workflow

This project uses **OpenSpec (OPSX)** to manage changes systematically. Every feature or significant fix follows this flow:

```
1. PROPOSE          (openspec-propose)
      ↓
2. SPECIFY          (openspec-spec)
      ↓
3. DESIGN           (openspec-design)
      ↓
4. CREATE TASKS     (openspec-tasks)
      ↓
5. IMPLEMENT        (openspec-apply-change)
      ↓
6. VERIFY           (openspec-verify)
      ↓
7. MERGE & DEPLOY
```

### When to Use Each Skill

**Starting a New Feature**:
1. Call `openspec-propose` to create initial proposal
2. Wait for stakeholder feedback
3. Move to specifying when approved

**During Specification Phase**:
1. Call `openspec-spec` to write requirements
2. Review with QA/stakeholders
3. Move to design when specs are locked

**During Design Phase**:
1. Call `openspec-design` to document architecture
2. Review with senior engineers
3. Move to tasks when design is approved

**During Implementation Phase**:
1. Call `openspec-tasks` to break work into checklist
2. Call `openspec-apply-change` to implement each task
3. Commit as you complete tasks

**Before Merge**:
1. Call `openspec-verify` to validate completeness
2. Fix any gaps
3. Create PR with verification report

### Agent Handoff Pattern

```
Human (orchestrator)
  ↓
Agent #1: Propose → initial proposal created
  ↓
Human: Review proposal
  ↓
Agent #2: Spec → requirements defined
  ↓
Human: Review & approve spec
  ↓
Agent #3: Design → architecture documented
  ↓
Human: Review & approve design
  ↓
Agent #4: Tasks → checklist created
  ↓
Agent #5: Apply → implementation executed
  ↓
Human: Code review (optional intermediate)
  ↓
Agent #6: Verify → validation complete
  ↓
Human: Merge to main
```

---

## Conventions Section

### Naming Conventions

#### Files & Directories

- **React Components**: PascalCase, one component per file  
  ✅ `src/components/ThreatMap.tsx`, `src/components/EventFilter.tsx`  
  ❌ `src/components/threatmap.tsx`, `src/components/threat-map.tsx`

- **Utility/Service Functions**: camelCase  
  ✅ `src/utils/enrichThreatData.ts`, `src/services/honeypotService.ts`  
  ❌ `src/utils/EnrichThreatData.ts`, `src/utils/enrich-threat-data.ts`

- **Hooks**: `use` prefix, camelCase  
  ✅ `src/hooks/useThreatData.ts`, `src/hooks/useEventFilters.ts`  
  ❌ `src/hooks/threatDataHook.ts`, `src/hooks/use-threat-data.ts`

- **Database Models**: Singular, snake_case  
  ✅ `honeypot_event`, `threat_intelligence`, `attack_surface`  
  ❌ `honeypot_events`, `ThreatIntelligence`, `attack-surface`

- **n8n Workflows**: Descriptive slug format  
  ✅ `cowrie-ssh-event-aggregator`, `threat-ip-enrichment`  
  ❌ `cowrie`, `enrich`, `threat_enrichment`

#### Variables & Functions

- **Constants**: UPPER_SNAKE_CASE  
  ✅ `const MITRE_ATTACK_URL = "https://..."`  
  ❌ `const mItreAttackUrl = "..."`

- **Type/Interface Names**: PascalCase  
  ✅ `type HoneypotEvent = { ... }`, `interface ThreatIntel { ... }`  
  ❌ `type honeypotEvent = { ... }`, `interface threat_intel { ... }`

- **Function Parameters**: camelCase  
  ✅ `function enrichEvent(eventData, maxRetries) { ... }`  
  ❌ `function enrichEvent(event_data, MaxRetries) { ... }`

---

### Folder Structure

```
Automaticacion_n8n_honeypots/
├── .opencode/                          # OpenCode configuration
│   ├── agents.md                       # This file - agent instructions
│   ├── skills/                         # Local skill installations
│   │   ├── openspec-apply-change/
│   │   ├── openspec-design/
│   │   ├── openspec-spec/
│   │   ├── openspec-tasks/
│   │   └── openspec-verify/
│   └── commands/                       # Custom CLI commands (if needed)
│
├── openspec/                           # OpenSpec artifacts (auto-generated)
│   ├── config.yaml                     # OpenSpec configuration
│   ├── changes/                        # Change metadata
│   ├── specs/                          # Written specifications
│   ├── designs/                        # Technical designs
│   └── tasks/                          # Task checklists
│
├── src/                                # Source code
│   ├── frontend/
│   │   ├── components/
│   │   │   ├── common/                 # Reusable UI components
│   │   │   ├── dashboard/              # Dashboard-related components
│   │   │   ├── maps/                   # Map visualizations
│   │   │   └── forms/                  # Form components
│   │   ├── hooks/                      # React custom hooks
│   │   ├── utils/                      # Frontend utilities
│   │   ├── services/                   # Frontend API services
│   │   ├── types/                      # TypeScript types (shared)
│   │   ├── App.tsx
│   │   └── index.tsx
│   │
│   ├── backend/
│   │   ├── api/
│   │   │   ├── controllers/            # Route handlers
│   │   │   ├── routes/                 # Route definitions
│   │   │   ├── middleware/             # Express middleware (auth, errors, etc.)
│   │   │   └── validators/             # Request validation (Joi, Zod)
│   │   ├── services/                   # Business logic layer
│   │   ├── models/                     # Data models/entities
│   │   ├── database/
│   │   │   ├── migrations/             # Database migrations
│   │   │   ├── seeds/                  # Seed data
│   │   │   └── schema.sql              # Current schema
│   │   ├── utils/                      # Shared utilities
│   │   ├── types/                      # TypeScript types
│   │   ├── config/                     # Configuration (env, logger, etc.)
│   │   └── server.ts
│   │
│   └── shared/                         # Shared types & constants
│       ├── types.ts
│       └── constants.ts
│
├── tests/
│   ├── unit/                           # Unit tests (mirrors src/)
│   ├── integration/                    # Integration tests
│   └── e2e/                            # End-to-end tests
│
├── workflows/                          # n8n workflows (as JSON/YAML)
│   ├── honeypot-event-capture.json
│   ├── threat-intelligence-enrichment.json
│   └── mitre-correlation.json
│
├── docs/                               # Documentation
│   ├── ARCHITECTURE.md
│   ├── API.md
│   ├── DATABASE.md
│   ├── WORKFLOWS.md
│   └── SECURITY.md
│
├── docker/                             # Docker-related files
│   ├── Dockerfile.api
│   ├── Dockerfile.frontend
│   └── docker-compose.yml
│
├── .github/workflows/                  # GitHub Actions CI/CD
│   ├── test.yml
│   ├── lint.yml
│   └── deploy.yml
│
├── README.md
├── package.json
└── tsconfig.json
```

---

### Commit Message Format

Follow **Conventional Commits** with this format:

```
<type>(<scope>): <subject>

<body>

<footer>
```

**Types**:
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation
- `style`: Code style (no logic change)
- `refactor`: Code reorganization
- `perf`: Performance improvement
- `test`: Test coverage
- `chore`: Build, config, dependencies
- `ci`: CI/CD configuration

**Scopes** (optional but recommended):
- `api`: Express API changes
- `frontend`: React component/UI changes
- `db`: Database schema/migration changes
- `n8n`: Workflow automation changes
- `security`: Security-related changes
- `infra`: Docker, deployment configuration
- `types`: TypeScript type definitions

**Examples**:

```
feat(api): add threat enrichment POST endpoint

Implement POST /api/threats/enrich to accept raw honeypot events
and return enriched threat intelligence with MITRE mapping.

Closes #42
```

```
fix(db): correct foreign key constraint in honeypot_events

Previous migration had incorrect reference. Corrects to properly
link events to honeypot_sessions.

Fixes #85
```

```
refactor(frontend): extract ThreatMapControls component

Reduces ThreatMap.tsx complexity by 300 lines. Component now
focuses on rendering, controls are separate.
```

```
test(api): add unit tests for IPEnricher service

70% → 95% coverage on IP enrichment logic.
```

---

### Testing Conventions

#### Unit Tests

- **File naming**: `[component].test.ts` or `[component].spec.ts`  
- **Location**: Mirror src/ structure in tests/
- **Coverage target**: 80% for business logic, 60% for UI
- **Framework**: Jest (for both backend and frontend)

Example:
```typescript
// src/backend/services/ThreatEnricher.ts
// tests/unit/backend/services/ThreatEnricher.test.ts

describe("ThreatEnricher", () => {
  describe("enrichEvent()", () => {
    it("should append MITRE ATT&CK mapping when technique_id provided", () => {
      // Arrange
      const event = { technique_id: "T1110" };
      // Act
      const enriched = enricher.enrichEvent(event);
      // Assert
      expect(enriched.mitre).toBeDefined();
    });
  });
});
```

#### Integration Tests

- **File naming**: `[flow].integration.test.ts`
- **Location**: `tests/integration/`
- **Scope**: Test service-to-database flows, API endpoint-to-database
- **Setup**: Use test database fixtures, cleanup after

Example:
```typescript
// tests/integration/api/threatsController.integration.test.ts
describe("POST /api/threats (integration)", () => {
  it("should capture event, enrich, and return intelligence", async () => {
    // Arrange: insert test data
    const event = { src_ip: "192.0.2.1", protocol: "ssh" };
    
    // Act
    const response = await request(app).post("/api/threats").send(event);
    
    // Assert
    expect(response.status).toBe(201);
    expect(response.body.mitre_techniques).toBeDefined();
  });
});
```

#### E2E Tests

- **File naming**: `[flow].e2e.test.ts`
- **Location**: `tests/e2e/`
- **Scope**: Full user flows from UI to database
- **Tool**: Cypress or Playwright

---

### TypeScript Conventions

#### Strict Mode

Always use strict TypeScript settings:

```json
{
  "compilerOptions": {
    "strict": true,
    "strictNullChecks": true,
    "noImplicitAny": true,
    "noImplicitThis": true,
    "alwaysStrict": true
  }
}
```

#### Type Definitions

- **Never use `any`** — use `unknown` if truly unknown, then narrow
- **Prefer interfaces for object contracts**, types for unions/aliases
- **Use discriminated unions** for polymorphic data

```typescript
// ✅ Good
type EventType = "ssh" | "protocol" | "malware";
interface HoneypotEvent {
  type: EventType;
  timestamp: Date;
  source_ip: string;
}

// ❌ Avoid
let event: any = { ... };
```

#### Async/Await

- Always use `async/await` over `.then()` chains
- Always handle errors with try/catch

```typescript
// ✅ Good
async function enrichEvent(event: HoneypotEvent): Promise<EnrichedEvent> {
  try {
    const intel = await threatIntel.fetch(event.source_ip);
    return { ...event, intel };
  } catch (error) {
    logger.error("Enrichment failed", { event, error });
    throw new EnrichmentError(error);
  }
}

// ❌ Avoid
function enrichEvent(event) {
  return threatIntel.fetch(event.source_ip).then(intel => {
    return { ...event, intel };
  });
}
```

---

## Conventions

### Code Style

- **Prettier**: Enforce formatting (configured in `package.json`)
- **ESLint**: Enforce rules (security, performance, correctness)
- **EditorConfig**: Ensure consistency across editors

### Comments

- Comment the "why", not the "what"
- Use JSDoc for public functions/exports
- Link to GitHub issues for non-obvious logic

```typescript
// ✅ Good
/**
 * Enrich honeypot event with threat intelligence
 * @param event - Raw honeypot capture
 * @returns Enriched event with MITRE mapping, IP reputation, etc.
 * @throws {EnrichmentError} If external API fails
 */
async function enrichEvent(event: HoneypotEvent): Promise<EnrichedEvent> {
  // Cowrie events from SSH honeypot need special handling
  // because srcIP is nested under "session" — see #42
  const srcIp = event.session?.source_ip || event.src_ip;
  // ...
}

// ❌ Avoid
// Get the IP address
const srcIp = event.src_ip;
```

---

## Gotchas & Warnings

### Critical Issues

**1. Database Migrations Are One-Way**
- ❌ Never manually edit production data
- ✅ Always create formal migrations
- Rollback strategy: Create reverse migration, don't just DROP

**2. n8n Webhooks Need Secrets**
- ❌ Never hardcode webhook tokens in workflows
- ✅ Use environment variables or n8n vault
- Issue: Exposed tokens = attacker can inject false events

**3. MITRE ATT&CK Mapping Has Gaps**
- ❌ Don't assume all attack vectors map to ATT&CK
- ✅ Document custom technique mapping
- Known issue: APT behaviors may not have official tactics yet

**4. IP Enrichment Rate Limits**
- ❌ Don't bulk-enrich without caching
- ✅ Implement Redis caching for IP lookups
- Issue: VirusTotal/AbuseIPDB have strict rate limits; caching reduces costs

**5. Honeypot Containers Can Leak Resources**
- ❌ Don't leave stopped containers running
- ✅ Use memory limits in docker-compose
- Issue: Dionaea can consume 2GB+ if not capped; impacts host

### Security Considerations

**Authentication & Secrets**
- All API keys stored in environment variables (`dotenv`)
- n8n admin panel requires basic auth (change default credentials immediately)
- Database passwords never in code or git history

**Data Classification**
- Raw honeypot events = sensitive (PII/attack patterns)
- Enriched events = exportable (threat intelligence)
- Logs should not contain raw payloads

**Network Segmentation**
- Honeypots on isolated Docker network (not host)
- n8n accessible only from authenticated users
- PostgreSQL accessible only from backend services

**Input Validation**
- All API endpoints validate request bodies (Zod/Joi)
- All database queries use parameterized statements (never raw SQL)
- All n8n webhooks verify origin/signature

### Common Breakages

| Issue | Cause | Fix |
|-------|-------|-----|
| Events not captured | Cowrie logs not mounted | Verify docker volume: `-v honeypot_logs:/var/lib/cowrie/log` |
| n8n workflows fail silently | Webhook secret mismatch | Re-generate webhook token in n8n UI |
| PostgreSQL won't start | Port 5432 already in use | `lsof -i :5432` then kill process or use different port |
| React won't build | TypeScript errors in production | `npm run type-check` before commit |
| Enrichment timeout | External API down | Add fallback to cache or mark as "pending enrichment" |
| Duplicate events | n8n webhook called twice | Add idempotency key check (request_id) |

### Performance Gotchas

- **N+1 Query Problem**: Loading events without eager-loading source session data
  - Fix: Use SQL JOINs or TypeORM relations
  
- **Unindexed event searches**: Queries on `source_ip` without index
  - Fix: Add compound index on `(timestamp, source_ip)`
  
- **React re-renders on every webhook**: No memoization on ThreatMap
  - Fix: Use `React.memo()` and `useMemo()` for expensive renders
  
- **n8n workflow memory leak**: Long-running workflows accumulate state
  - Fix: Set workflow timeout, use pagination for bulk enrichment

---

## Contact & Support

- **Lead Architect**: Ignacio Navarria (GitHub: @navanacho)
- **Co-Lead**: Germán Marino
- **Issues**: Report via GitHub Issues with `[opencode]` prefix
- **Feedback**: https://github.com/anomalyco/opencode
- **Questions**: Refer to `.opencode/agents.md` or project docs/

---

## Document Version

- **Version**: 1.0
- **Last Updated**: 2026-05-28
- **Status**: Active (all skills installed locally)
