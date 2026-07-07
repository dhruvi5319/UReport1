---
pivota_spec_state_version: 1.0
milestone: v1.0
milestone_name: milestone
status: executing
stopped_at: Completed 01-k8s-scaffold-data-foundation-02-PLAN.md
last_updated: "2026-07-07T19:54:42.587Z"
last_activity: "2026-07-07 — Plan 01-01 complete: Next.js 15 scaffold, Prisma singleton, boot entrypoint"
progress:
  total_phases: 7
  completed_phases: 0
  total_plans: 8
  completed_plans: 2
  percent: 25
---

# Project State

## Project Reference

See: .planning/PROJECT.md (updated 2026-07-06)

**Core value:** City constituents can report municipal issues and staff can manage the full ticket lifecycle — all from one responsive, accessible web app running as a single Kubernetes pod with a Postgres sidecar.
**Current focus:** Phase 1 — K8s Scaffold & Data Foundation

## Current Position

Phase: 1 of 7 (K8s Scaffold & Data Foundation)
Plan: 2 of 3 in current phase (01-01, 01-02 complete)
Status: In progress
Last activity: 2026-07-07 — Plan 01-02 complete: Full Prisma schema, 3 migration files, FTS triggers, seed data

Progress: [███░░░░░░░] 25%

## Performance Metrics

**Velocity:**

- Total plans completed: 2
- Average duration: 7.5 min
- Total execution time: 0.25 hours

**By Phase:**

| Phase | Plans | Total | Avg/Plan |
|-------|-------|-------|----------|
| 01-k8s-scaffold-data-foundation | 2/3 | 15 min | 7.5 min |

**Recent Trend:**

- Last 5 plans: 01-01 (5 min), 01-02 (10 min)
- Trend: stable

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
- [Phase 01-k8s-scaffold-data-foundation]: search_vector tsvector NOT in Prisma schema — managed by trigger migration SQL to avoid unsupported type errors
- [Phase 01-k8s-scaffold-data-foundation]: PostGIS conditional DO $$ block in migration — safe no-op on plain Postgres 16; geog column only added when postgis extension present

### Pending Todos

None yet.

### Blockers/Concerns

None yet.

## Session Continuity

Last session: 2026-07-07T19:54:42.585Z
Stopped at: Completed 01-k8s-scaffold-data-foundation-02-PLAN.md
Resume file: None
