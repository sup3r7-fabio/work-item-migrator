<!--
Sync Impact Report - Constitution v1.1.0
========================================
Version Change: 1.0.0 → 1.1.0
Rationale: MINOR version bump - Added new principle (Direct API Implementation) and expanded 
technical constraints (Golang backend specification)

Modified Principles:
  - Principle VI (API Standards & Contracts) → Enhanced with direct implementation requirement
  
Added Sections:
  - NEW Principle VIII: Direct API Implementation (NON-NEGOTIABLE) - No third-party Azure DevOps SDKs
  - Backend language specification: Golang 1.21+ mandatory
  - Go-specific quality gates and testing requirements

Removed Sections: N/A

Templates Requiring Updates:
  ✅ .specify/templates/plan-template.md - Constitution Check must validate no azure-devops-* deps
  ✅ .specify/templates/spec-template.md - Requirements validation aligned
  ✅ .specify/templates/tasks-template.md - Test-first task categorization aligned
  ⚠️  .specify/templates/plan-template.md - Update "Core Technology Stack" guidance to reflect Go backend

Follow-up TODOs:
  - Update plan-template.md Technical Context section to include Go/testing.T guidance
  - Add Go-specific linting rules (golangci-lint) to quality gates documentation

Last Updated: 2025-11-29
-->

# Azure DevOps Work Item Migrator Constitution

## Purpose

The Azure DevOps Work Item Migrator is a robust, user-friendly tool designed to seamlessly migrate work items, boards, and teams between Azure DevOps organizations. It provides rich editing capabilities, bulk migration operations, and extensibility through a plugin system, ensuring organizations can transition smoothly without data loss or service disruption.

## Core Principles

### I. Test-First Development (NON-NEGOTIABLE)

**TDD is mandatory for all features:**
- Tests MUST be written before implementation code
- Tests MUST be reviewed and approved by stakeholders before implementation begins
- Red-Green-Refactor cycle strictly enforced: Write failing test → Implement → Refactor
- No code merged without corresponding test coverage (minimum 80% line coverage)
- Integration tests required for Azure DevOps API interactions and migration workflows

**Rationale**: Migration tools handle critical business data. Test-first development ensures reliability, prevents data loss, and builds confidence in migration operations. Given the complexity of Azure DevOps APIs and the risk of data corruption, comprehensive testing is non-negotiable.

### II. Library-First Architecture

**Every feature starts as a standalone, reusable library:**
- Core migration logic MUST be framework-agnostic and UI-independent
- Libraries MUST be self-contained with clear interfaces and zero UI dependencies
- Each library MUST have comprehensive documentation and usage examples
- Libraries MUST be independently testable without UI or framework overhead
- Clear separation: `core/` (business logic) → `ui/` (presentation layer)

**Rationale**: Whether delivered as a web SPA or Electron app is still under evaluation. Library-first architecture ensures migration logic can be reused across both delivery models, in automation scripts, or as a standalone CLI tool. This also enables easier testing and maintenance.

### III. UX-First Design

**User experience drives all interface decisions:**
- Migration operations MUST provide real-time progress indicators (% complete, ETA)
- All destructive operations MUST require explicit confirmation with clear consequences
- Error messages MUST be actionable (what went wrong + how to fix it)
- Bulk operations MUST support preview mode before execution
- Migration history and audit logs MUST be accessible and exportable
- Undo/rollback capabilities required for critical operations where feasible

**Rationale**: Migrations are high-stakes operations. Users need confidence, visibility, and control. Clear feedback prevents costly mistakes and reduces support burden.

### IV. Performance & Scalability

**The system MUST handle enterprise-scale migrations efficiently:**
- Support migration of 10,000+ work items without UI freezing or timeouts
- Implement batching and pagination for all bulk operations (max 200 items per batch)
- Use background workers/web workers for long-running operations
- Implement rate limiting and retry logic for Azure DevOps API calls (respect 429 responses)
- Memory usage MUST remain constant regardless of migration size (streaming/chunking required)
- Progress persisted to enable resume after interruption or failure

**Rationale**: Enterprise migrations involve thousands of work items. Poor performance leads to timeouts, data loss, and user frustration. Scalability ensures the tool remains viable for organizations of any size.

### V. Security & Compliance

**Data protection and authentication are mandatory:**
- All Azure DevOps credentials MUST use OAuth 2.0 or Personal Access Tokens (PAT)
- Credentials MUST NEVER be logged or stored in plain text
- Use OS-level credential managers (Windows Credential Manager, macOS Keychain, Linux Secret Service)
- All API calls MUST use HTTPS; reject insecure connections
- Audit logs MUST capture all migration operations with user identity, timestamp, and scope
- Implement least-privilege access: only request necessary Azure DevOps scopes

**Rationale**: Work items contain sensitive business data. Credential leaks or unauthorized access could expose confidential information. Compliance with security best practices protects users and their organizations.

### VI. API Standards & Contracts

**Azure DevOps API interactions MUST follow strict contracts:**
- All API models MUST match Azure DevOps REST API v7.0+ schemas exactly
- Implement retry logic with exponential backoff for transient failures (3 retries, 1s/2s/4s delays)
- Validate all API responses against expected schemas before processing
- Handle API versioning explicitly; fail fast if incompatible version detected
- Rate limiting MUST respect Azure DevOps service limits (default: 200 requests per hour per resource)
- Cache read-only data (project metadata, work item types) to reduce API calls
- All API contracts MUST be defined using strongly-typed Go structs with JSON tags

**Rationale**: Azure DevOps APIs are the foundation of this tool. Robust API handling prevents data corruption, reduces errors, and ensures compatibility across Azure DevOps updates. Direct implementation ensures full control and transparency.

### VII. Observability & Diagnostics

**All operations MUST be traceable and debuggable:**
- Structured logging required for all migration operations (JSON format with correlation IDs)
- Log levels: ERROR (failures), WARN (retries/fallbacks), INFO (milestones), DEBUG (API calls)
- Performance metrics captured: API latency, batch processing time, memory usage
- Export diagnostic bundles on error (logs, configuration, API responses with sensitive data redacted)
- Telemetry for usage patterns (opt-in): feature adoption, error rates, performance bottlenecks

**Rationale**: When migrations fail, users need detailed diagnostics to resolve issues quickly. Observability enables rapid troubleshooting and continuous improvement through data-driven insights.

### VIII. Direct API Implementation (NON-NEGOTIABLE)

**NO third-party Azure DevOps SDK libraries permitted:**
- ALL Azure DevOps API interactions MUST be implemented directly using HTTP clients
- Backend MUST use Go's standard `net/http` package for API calls
- NO dependencies on `azure-devops-node-api`, `azure-devops-extension-api`, or any similar SDKs
- API request/response models MUST be defined as Go structs in the codebase
- HTTP client configuration MUST be explicit and customizable (timeouts, retries, headers)
- OpenAPI/Swagger specs MAY be used to generate initial struct definitions but MUST be reviewed and maintained manually

**Rationale**: Third-party SDKs introduce dependencies that:
- Obscure API behavior and reduce control over request/response handling
- Create version lock-in and delayed updates when Azure DevOps APIs change
- Add unnecessary abstraction layers that complicate debugging
- Limit customization for performance optimization and retry strategies
- Increase attack surface and maintenance burden

Direct implementation ensures maximum control, transparency, testability, and maintainability. The Azure DevOps REST API is well-documented and stable, making direct implementation straightforward.

## Technical Constraints

### Technology & Platform Decisions

**Delivery Model (To Be Determined):**
- **Option A**: Web SPA (React/Vue + TypeScript) with cloud backend
- **Option B**: Electron app (desktop-first, offline-capable)
- **Decision Criteria**: Cross-platform requirements, offline support needs, enterprise deployment preferences
- **Current Stance**: Architecture MUST support both options (library-first principle enforces this)

**Cross-Platform Support (REQUIRED):**
- MUST run on Windows, macOS, and Linux
- If web SPA: Modern browsers (Chrome, Edge, Firefox, Safari - last 2 versions)
- If Electron: Native desktop experience with OS-specific integrations

**Core Technology Stack:**
- **Backend Language**: Go 1.25+ (REQUIRED, NON-NEGOTIABLE)
  - HTTP Client: Go standard library `net/http` package
  - JSON Handling: Go standard library `encoding/json`
  - Testing: Go standard library `testing` package + `testify` for assertions
  - No third-party Azure DevOps SDKs permitted (see Principle VIII)
- **Frontend Language**: TypeScript (strict mode enabled)
- **Frontend Testing**: Jest for unit tests, Playwright for E2E tests
- **State Management**: Redux Toolkit or Zustand (for UI layer)
- **Build System**: 
  - Backend: Go modules with `go build`
  - Frontend: Vite or Webpack with tree-shaking enabled

### Performance Baselines

- **Bulk Operations**: Process 200 work items per batch, max 2s per batch
- **UI Responsiveness**: Main thread MUST remain responsive during migrations (max 16ms blocking)
- **Memory Budget**: Max 500MB RAM for 10,000 work item migration
- **API Efficiency**: Max 100 API calls per 1,000 work items migrated (via batching and caching)

### Extensibility Requirements

**Plugin System (REQUIRED for v2.0+):**
- Support custom field transformations via plugin API
- Enable custom validation rules before migration
- Allow post-migration hooks for custom integrations
- Plugin sandboxing to prevent system corruption
- Well-documented plugin development guide and examples

## Quality Standards

### Code Quality Gates

**All code MUST pass these gates before merge:**
- **Backend (Go)**:
  - Linting: `golangci-lint` with default rules + `govet`, `staticcheck`, `errcheck` (zero warnings)
  - Formatting: `gofmt` and `goimports` (enforced via pre-commit hooks)
  - Type Safety: No `interface{}` without documented justification
  - Test Coverage: Minimum 80% line coverage, 90% for core migration logic and API client code
- **Frontend (TypeScript)**:
  - Linting: ESLint with strict TypeScript rules (zero warnings)
  - Formatting: Prettier with project config (enforced via pre-commit hooks)
  - Type Safety: TypeScript strict mode with no `any` types (exceptions require justification)
  - Test Coverage: Minimum 80% line coverage
- **Code Review**: Minimum one approval from maintainer; security-sensitive changes require two approvals

### Definition of Done (DoD)

**A feature is complete when:**
1. Tests written and passing (unit + integration)
2. Documentation updated (API docs, user guides, migration guides)
3. Performance validated against baselines
4. Security review completed (if authentication or data handling involved)
5. Accessibility validated (if UI component): WCAG 2.1 AA compliance
6. Migration rollback plan documented (for breaking changes)

## Governance

**Constitution Authority:**
- This constitution supersedes all other development practices and guidelines
- All pull requests MUST be reviewed for constitutional compliance
- Violations MUST be justified with documented rationale and approved by two maintainers
- Technical complexity MUST be justified against business value; prefer simplicity

**Amendment Process:**
- Constitution changes require proposal with rationale and impact analysis
- Amendments require approval from project lead and majority of core contributors
- Breaking changes require migration plan and deprecation timeline
- Version bumps follow semantic versioning: MAJOR (breaking principles), MINOR (new principles), PATCH (clarifications)

**Enforcement:**
- Pre-commit hooks enforce formatting and linting
- CI pipeline enforces test coverage and type safety
- Code review checklist includes constitutional compliance verification
- Quarterly constitution review to ensure relevance and effectiveness

**Version**: 1.1.0 | **Ratified**: 2025-11-29 | **Last Amended**: 2025-11-29
