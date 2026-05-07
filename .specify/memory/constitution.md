<!--
SYNC IMPACT REPORT
==================
Version change: (none) → 1.0.0
Bump rationale: Initial ratification of project constitution from template; establishes
  foundational principles, constraints, and governance for the project-management-app.

Modified principles:
  - [PRINCIPLE_1_NAME] → I. User-Centric Collaboration
  - [PRINCIPLE_2_NAME] → II. Modular, Domain-Driven Architecture
  - [PRINCIPLE_3_NAME] → III. Test-First (NON-NEGOTIABLE)
  - [PRINCIPLE_4_NAME] → IV. Real-Time Reliability & Data Integrity
  - [PRINCIPLE_5_NAME] → V. Observability, Security & Performance

Added sections:
  - Additional Constraints & Quality Standards
  - Development Workflow & Quality Gates

Removed sections: None (all template placeholders resolved).

Templates requiring updates:
  - ✅ .specify/templates/plan-template.md (reviewed; Constitution Check section is generic
    and remains compatible with new principles — no edits required)
  - ✅ .specify/templates/spec-template.md (reviewed; remains compatible — no edits required)
  - ✅ .specify/templates/tasks-template.md (reviewed; remains compatible — no edits required)
  - ✅ .specify/templates/checklist-template.md (reviewed; remains compatible — no edits required)
  - ⚠ README.md / docs/quickstart.md: not present in repo yet; create when project docs
    are introduced and ensure they reference these principles.

Follow-up TODOs:
  - TODO(RATIFICATION_DATE): Confirm official adoption date with project owner; placeholder
    set to today's date (2026-05-07) on initial ratification.
-->

# project-management-app Constitution

## Core Principles

### I. User-Centric Collaboration
The product exists to help teams plan, execute, and report on work together. Every feature
MUST be evaluated against concrete collaboration use cases (task boards, Gantt planning,
time tracking, file sharing, milestone tracking, dashboards). Specifically:

- Every user-facing capability MUST define the roles affected (e.g., Admin, Project Manager,
  Member, Viewer) and the permissions model BEFORE implementation begins.
- Multi-user flows (assignment, comments, mentions, notifications, shared files) MUST be
  designed for concurrent edits and MUST specify conflict-resolution behavior.
- Accessibility (WCAG 2.1 AA) and keyboard-first navigation are MANDATORY for all primary
  workflows (board, Gantt, task detail, dashboard).

Rationale: A project management tool that ignores collaboration semantics or accessibility
fails its core audience.

### II. Modular, Domain-Driven Architecture
The system MUST be organized around clear bounded contexts: Projects, Tasks/Boards,
Scheduling/Gantt, Time Tracking, Collaboration (comments/mentions/notifications), Files,
Milestones, and Reporting. Each context:

- MUST expose a well-defined API contract (REST and/or GraphQL) with versioning.
- MUST be independently testable; cross-context coupling MUST go through contracts, not
  shared internals.
- MUST own its data model; cross-context reads happen via APIs or read models, not direct
  database joins across contexts.
- New cross-cutting concerns (e.g., audit, search) MUST be implemented as shared services,
  not duplicated per context.

Rationale: Project management apps grow in feature surface; modular boundaries are the only
sustainable way to evolve boards, Gantt, time, and reporting independently.

### III. Test-First (NON-NEGOTIABLE)
Test-Driven Development is mandatory. For every change:

1. Write failing contract/integration tests for the API or component contract first.
2. Write failing unit tests for domain logic (scheduling, dependency resolution, time
   aggregation, permission checks) before implementation.
3. Implement only enough code to make tests pass; refactor with green tests.

Coverage gates:
- Domain logic (Gantt dependency calculations, recurrence, time rollups, RBAC) MUST have
  ≥ 90% line and branch coverage.
- All public API endpoints MUST have contract tests.
- Critical user journeys (create project → plan tasks → assign → track time → close
  milestone → view report) MUST have end-to-end tests.

Rationale: Scheduling and permission bugs in PM tools cause data corruption and trust loss;
TDD is the cheapest way to prevent regressions across a feature-rich domain.

### IV. Real-Time Reliability & Data Integrity
Boards, Gantt views, comments, and dashboards are inherently collaborative and time-sensitive:

- All write operations on shared entities MUST be transactional and idempotent where
  feasible (use idempotency keys for retried client calls).
- Real-time updates (board moves, task edits, comments, presence) MUST use a documented
  channel (e.g., WebSocket/SSE) with reconnection and replay-from-cursor semantics.
- Time tracking entries MUST be append-only with explicit edit history; deletions MUST be
  soft-deletes with audit trail.
- Gantt schedules MUST validate dependency cycles, working-calendar constraints, and
  milestone consistency before persisting.
- File uploads MUST be virus-scanned (or queued for scanning) and MUST enforce per-project
  storage quotas and content-type allow-lists.

Rationale: Users trust PM tools as the system of record for work; silent data loss,
re-ordered events, or invalid schedules destroy that trust.

### V. Observability, Security & Performance
Every feature MUST ship with operability and security baked in:

- **Observability**: Structured JSON logging with request/correlation IDs; metrics for
  latency, error rate, and saturation per endpoint; traces across service boundaries; user
  action audit log for security-sensitive operations.
- **Security**: Authentication via standard protocols (OAuth 2.0 / OIDC); authorization
  enforced server-side via RBAC at the project and resource level; secrets MUST NOT be
  committed; all PII and file content encrypted in transit (TLS 1.2+) and at rest.
- **Performance budgets** (defaults; revise per release in plan.md):
  - Board view initial load: P95 ≤ 2.0s for ≤ 500 tasks.
  - Task create/update API: P95 ≤ 300ms.
  - Gantt render for ≤ 1,000 tasks: P95 ≤ 3.0s.
  - Dashboard report queries: P95 ≤ 5.0s.
- Any feature exceeding these budgets MUST document the deviation and mitigation in the
  plan's Complexity Tracking section.

Rationale: PM tools are used daily by entire organizations; security failures and slow
boards are existential product risks.

## Additional Constraints & Quality Standards

- **Tech stack discipline**: Stack choices (frontend framework, backend runtime, database,
  message bus, real-time layer) are decided in the first plan.md and MUST NOT be changed
  without an amendment to this constitution or an explicit ADR linked from plan.md.
- **Database**: A relational database (PostgreSQL or equivalent) is the default system of
  record. Schema migrations MUST be backward compatible within a minor version; destructive
  migrations require an explicit deprecation window.
- **API versioning**: Public APIs follow semantic versioning. Breaking changes require a
  new major version path (e.g., `/api/v2/...`) with at least one minor-version deprecation
  notice.
- **Internationalization**: All user-facing strings MUST go through an i18n layer from day
  one; no hardcoded English UI strings in production code paths.
- **Time & calendars**: Times MUST be stored in UTC; user-facing displays MUST honor the
  user's timezone and locale. Working-day/working-hour calendars are first-class entities
  for Gantt and time tracking.
- **Privacy & compliance**: Personal data handling MUST follow GDPR-style principles
  (purpose limitation, data export, account deletion). Audit logs are retained per the
  documented retention policy.

## Development Workflow & Quality Gates

- **Specify → Plan → Tasks → Implement**: All non-trivial features follow the spec-kit
  workflow. `/speckit.specify` defines WHAT and WHY (no implementation detail);
  `/speckit.plan` defines HOW (stack, contracts, data model); `/speckit.tasks` defines
  ordered work; implementation follows tasks.
- **Constitution Check**: Every plan.md MUST pass the Constitution Check gate before
  Phase 0 research and again before Phase 1 design. Violations MUST be justified in the
  Complexity Tracking section or the plan is revised.
- **Code review**: Every change MUST be reviewed by at least one engineer who did not
  author it. Reviews MUST verify: tests exist and fail without the change; principles I–V
  are upheld; security and performance budgets are not regressed.
- **CI gates** (mandatory before merge): lint, type-check, unit tests, contract tests,
  integration tests for affected contexts, and a build of the deployable artifact.
- **Definition of Done** for a user-visible feature: spec accepted; plan + tasks committed;
  tests green at required coverage; contract tests for any new API; telemetry (logs,
  metrics, traces) added; docs updated; accessibility checks performed for UI; security
  review for any change touching auth, files, or PII.

## Governance

This constitution supersedes ad-hoc practices and prior conventions. It applies to every
contributor, automated agent, and downstream integration in this repository.

- **Amendment procedure**: Proposed amendments are submitted as a PR modifying this file
  with a Sync Impact Report. Amendments require approval from the project owner and at
  least one other maintainer. Once merged, dependent templates and docs MUST be updated in
  the same PR or an immediately following PR linked from the amendment.
- **Versioning policy** (semantic):
  - **MAJOR**: Backward-incompatible governance changes or removal/redefinition of a
    principle.
  - **MINOR**: Addition of a new principle or materially expanded section.
  - **PATCH**: Clarifications, wording, or non-semantic refinements.
- **Compliance review**: Maintainers conduct a quarterly review verifying that recent
  features, plans, and tasks align with this constitution. Findings MUST be filed as
  issues; recurring violations trigger an amendment proposal.
- **Runtime guidance**: Day-to-day agent and contributor guidance lives in `CLAUDE.md` and
  any future `docs/quickstart.md`. Those documents MUST defer to this constitution on any
  conflict.

**Version**: 1.0.0 | **Ratified**: 2026-05-07 | **Last Amended**: 2026-05-07
