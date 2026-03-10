<!--
  Sync Impact Report
  ==================
  Version change: N/A → 1.0.0 (initial ratification)

  Added principles:
    - I. Security-First Architecture
    - II. AI Agent Governance
    - III. Developer Experience
    - IV. AI Agent Design
    - V. API Design Standards
    - VI. Testing Discipline
    - VII. Error Handling & Observability

  Added sections:
    - Technical Stack & Data Layer
    - Domain Architecture & Module Responsibilities
    - Development Workflow & Operational Standards

  Removed sections: N/A (initial version)

  Templates requiring updates:
    - .specify/templates/plan-template.md — ✅ compatible (Constitution Check section aligns)
    - .specify/templates/spec-template.md — ✅ compatible (user stories, requirements, success criteria align)
    - .specify/templates/tasks-template.md — ✅ compatible (phase structure, parallel markers, test-first align)
    - .specify/templates/agent-file-template.md — ✅ compatible (technology extraction will pull from this constitution)

  Follow-up TODOs: None
-->

# CyberAwareness Platform Constitution

## Core Principles

### I. Security-First Architecture

- **Zero Trust**: All requests MUST be validated; trust nothing by default.
  Assume breach at every layer boundary.
- **Defense in Depth**: Every service MUST implement multiple security layers.
  No single point of failure is acceptable.
- **Least Privilege**: All services, users, and AI agents MUST operate with
  minimal permissions required for their function.
- **Audit Everything**: All security events and AI decisions MUST be logged
  with structured, correlation-ID-tagged entries.

### II. AI Agent Governance

- **Explainability**: All AI decisions MUST be traceable and auditable.
  Agent outputs MUST include reasoning metadata.
- **Human-in-the-Loop**: Critical security actions (quarantine, block, escalate)
  MUST require human confirmation before execution.
- **Graceful Degradation**: The system MUST remain functional when AI services
  are unavailable, falling back to rule-based analysis.
- **Bias Awareness**: AI recommendations MUST be regularly evaluated for
  fairness; evaluation sets MUST be maintained alongside prompts.

### III. Developer Experience

- **Spec-Driven Development**: Feature specifications are the source of truth.
  All implementation MUST trace back to a ratified spec.
- **Type Safety**: TypeScript strict mode is mandatory. No `any` types in
  production code. All public interfaces MUST have explicit types.
- **Test Coverage**: Minimum 80% coverage for all modules; 100% coverage
  for security-critical paths (auth, RBAC, encryption, agent actions).
- **Documentation as Code**: APIs MUST be self-documenting via OpenAPI/Swagger.
  Living documentation MUST stay in sync with implementation.

### IV. AI Agent Design

- **Single Responsibility**: Each agent type (threat_analyst, training_coach,
  phishing_sim, incident_resp, security_advisor) MUST have a focused domain.
- **Composable Tools**: Agents MUST share a tool registry; tools MUST be
  reusable across agent types without duplication.
- **Stateless Execution**: Agent state MUST live in the database, not in
  the agent process. Agents MUST be horizontally scalable.
- **Idempotent Actions**: All agent-triggered operations MUST be repeat-safe.
- **Observable**: All agent actions MUST emit telemetry events via
  OpenTelemetry for tracing and cost tracking.

### V. API Design Standards

- **REST Conventions**: Resources use plural nouns; actions use POST on
  sub-resources (e.g., `POST /threats/:id/analyze`).
- **Response Envelope**: All responses MUST use the standard `ApiResponse<T>`
  envelope with `data`, `meta.requestId`, and `meta.timestamp`.
- **Error Contract**: Errors MUST return `ApiError` with machine-readable
  `code`, human-readable `message`, and `requestId`.
- **Versioning**: URL-based versioning (`/api/v1/`). Deprecated versions
  MUST include a `Sunset` header and receive 6 months of support.

### VI. Testing Discipline

- **Test Pyramid**: 60% unit, 30% integration, 10% E2E. Unit tests cover
  business logic; integration tests cover API + DB; E2E covers critical
  user journeys.
- **AI Testing**: LLM responses MUST be mocked in unit/integration tests.
  Evaluation sets MUST exist for AI quality (separate from CI).
  Prompt changes MUST have regression tests.
- **Security Tests**: Auth, RBAC, rate limiting, and data encryption MUST
  have dedicated test suites with 100% path coverage.

### VII. Error Handling & Observability

- **Error Hierarchy**: All domain errors MUST extend `AppError` with a
  machine-readable `code`, HTTP `statusCode`, and `isOperational` flag.
- **Global Handler**: All errors MUST be logged with correlation ID.
  Stack traces MUST NOT be exposed in production responses.
- **Structured Logging**: Pino with JSON output. All log entries MUST
  include `requestId`, `timestamp`, and `level`.
- **Distributed Tracing**: OpenTelemetry MUST instrument all service
  boundaries, database calls, and AI agent invocations.

## Technical Stack & Data Layer

### Core Framework

| Component | Technology | Version |
|-----------|-----------|---------|
| Runtime | Node.js LTS | 20+ |
| Framework | NestJS | 10+ |
| Language | TypeScript (strict) | 5+ |
| Package Manager | pnpm | latest |

### Data Layer

| Component | Technology | Purpose |
|-----------|-----------|---------|
| Primary DB | PostgreSQL 15+ | Security events, user data, training |
| Cache | Redis 7+ | Sessions, rate limiting, real-time state |
| Vector Store | pgvector | Threat embeddings, semantic search |
| Message Queue | BullMQ | Async jobs, AI agent coordination |

### AI/ML Infrastructure

| Component | Technology | Purpose |
|-----------|-----------|---------|
| LLM Provider | Anthropic Claude API | Primary AI backbone |
| Embeddings | OpenAI text-embedding-3-small | Threat analysis vectors |
| Agent Framework | Custom NestJS orchestration | Agent lifecycle management |
| RAG | LangChain.js | Document retrieval |

### Security & Observability

| Component | Technology |
|-----------|-----------|
| Auth | Passport.js + JWT (15min) + Refresh tokens |
| RBAC | CASL.js (role hierarchy: super_admin > org_admin > manager > user) |
| Logging | Pino (structured JSON) |
| Tracing | OpenTelemetry |
| Metrics | Prometheus + Grafana |

## Domain Architecture & Module Responsibilities

### Bounded Contexts

1. **Threat Intelligence**: Ingest/normalize threat feeds (STIX/TAXII,
   MITRE ATT&CK), AI-powered analysis, real-time alerts, pattern detection.
2. **Training & Assessment**: Adaptive learning paths, phishing simulations,
   AI-generated assessments, progress tracking, competency mapping.
3. **User Management**: Multi-tenant orgs, RBAC, user risk scoring,
   behavioral analytics, SSO/SAML integration.
4. **AI Agent Orchestration (Shared Kernel)**: Agent lifecycle, tool/function
   coordination, memory management, rate limiting, cost tracking.

### Project Structure

```
src/
├── main.ts
├── app.module.ts
├── common/          # decorators, filters, guards, interceptors, pipes, utils
├── config/          # app, database, AI configuration
├── modules/
│   ├── threat-intelligence/
│   ├── training/
│   ├── users/
│   └── ai-orchestration/
├── database/        # migrations, seeds
└── jobs/            # background job processors
```

### Performance Targets

| Metric | Target | Critical Threshold |
|--------|--------|--------------------|
| API Response (p95) | < 200ms | < 500ms |
| AI Agent Response | < 5s | < 15s |
| Threat Alert Latency | < 30s | < 60s |
| Concurrent Users | 10,000 | 5,000 |
| Uptime | 99.9% | 99.5% |

### Rate Limits

| Scope | Window | Max Requests |
|-------|--------|-------------|
| API (general) | 1 minute | 100 |
| Auth (failed attempts) | 15 minutes | 5 |
| AI Agent (per user) | 1 hour | 50 |

## Development Workflow & Operational Standards

### Spec-Driven Development (SDD)

1. `/speckit.constitution` — Establish project principles
2. `/speckit.specify` — Define feature behavior
3. `/speckit.clarify` — Resolve ambiguities
4. `/speckit.plan` — Generate implementation plan
5. `/speckit.tasks` — Break into atomic tasks
6. `/speckit.implement` — Execute against spec
7. Validate — Tests + spec compliance check

### Git Workflow

- `main` — Production-ready code
- `develop` — Integration branch
- `feature/XXX-description` — Feature branches
- `fix/XXX-description` — Bug fix branches

### Commit Convention

```
<type>(<scope>): <description>
Types: feat, fix, docs, refactor, test, chore
Scopes: threat, training, users, ai, infra
```

### Health Checks

- `/health` — Basic liveness (`{ status: 'ok' }`)
- `/health/ready` — Full readiness (database, redis, AI service, threat feed)

### Disaster Recovery

- Database: Point-in-time recovery, 30-day retention
- Redis: AOF persistence, replica failover
- AI: Fallback to rule-based analysis when LLM unavailable

## Governance

- This constitution is the authoritative source for architectural decisions
  and development standards. All PRs MUST verify compliance.
- **Amendments** require: (1) documented rationale, (2) impact analysis on
  dependent templates, (3) version bump per semantic versioning.
- **Version policy**: MAJOR for principle removals/redefinitions, MINOR for
  new principles or material expansions, PATCH for clarifications.
- **Compliance review**: Every feature plan MUST include a Constitution Check
  gate verifying adherence to all applicable principles.
- **Complexity justification**: Any deviation from these principles MUST be
  documented in the plan's Complexity Tracking table with rationale.

**Version**: 1.0.0 | **Ratified**: 2026-03-11 | **Last Amended**: 2026-03-11
