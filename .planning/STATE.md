---
pivota_spec_state_version: 1.0
milestone: v1.0
milestone_name: milestone
status: planning
stopped_at: Completed 01-k8s-scaffold-data-foundation-01-PLAN.md
last_updated: "2026-07-07T19:41:26.221Z"
last_activity: 2026-07-06 — Roadmap created; all 59 v1 requirements mapped across 7 phases
progress:
  total_phases: 7
  completed_phases: 0
  total_plans: 8
  completed_plans: 1
  percent: 13
---

# Project State

## Project Reference

See: .planning/PROJECT.md (updated 2026-07-06)

**Core value:** City constituents can report municipal issues and staff can manage the full ticket lifecycle — all from one responsive, accessible web app running as a single Kubernetes pod with a Postgres sidecar.
**Current focus:** Phase 1 — K8s Scaffold & Data Foundation

## Current Position

Phase: 1 of 7 (K8s Scaffold & Data Foundation)
Plan: 1 of 3 in current phase (01-01 complete)
Status: In progress
Last activity: 2026-07-07 — Plan 01-01 complete: Next.js 15 scaffold, Prisma singleton, boot entrypoint

Progress: [█░░░░░░░░░] 13%

## Performance Metrics

**Velocity:**

- Total plans completed: 1
- Average duration: 5 min
- Total execution time: 0.1 hours

**By Phase:**

| Phase | Plans | Total | Avg/Plan |
|-------|-------|-------|----------|
| 01-k8s-scaffold-data-foundation | 1/3 | 5 min | 5 min |

**Recent Trend:**

- Last 5 plans: 01-01 (5 min)
- Trend: —

*Updated after each plan completion*

## Accumulated Context

### Decisions

Decisions are logged in PROJECT.md Key Decisions table.
Recent decisions affecting current work:

- [Init]: Full-stack Next.js 15 single process — one port maps cleanly to K8s single-pod; avoids dual-process complexity
- [Init]: Prisma ORM + PostgreSQL 16 sidecar — `prisma migrate deploy` at boot, idempotent
- [Init]: Postgres FTS (tsvector+GIN) replaces Solr — eliminates second stateful service
- [Init]: PostGIS as enhancement, not hard dep — graceful Haversine fallback
- [Init]: Auth.js credentials provider (no OAuth) — three roles: public/staff/admin
- [Phase 01-k8s-scaffold-data-foundation]: No X-Frame-Options in next.config.ts — Pivota Preview embeds app in iframe
- [Phase 01-k8s-scaffold-data-foundation]: next-auth@beta used (^5.0.0 doesn't match pre-release beta tags)
- [Phase 01-k8s-scaffold-data-foundation]: infrastructure.json declares postgres sidecar at port 3000 for Pivota K8s platform

### Pending Todos

None yet.

### Blockers/Concerns

None yet.

## Session Continuity

Last session: 2026-07-07T19:41:26.219Z
Stopped at: Completed 01-k8s-scaffold-data-foundation-01-PLAN.md
Resume file: None
