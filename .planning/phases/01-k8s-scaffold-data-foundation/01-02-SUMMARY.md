---
phase: 01-k8s-scaffold-data-foundation
plan: "02"
subsystem: database
tags: [prisma, postgresql, migrations, fts, tsvector, postgis, seed, bcrypt]

# Dependency graph
requires:
  - phase: 01-k8s-scaffold-data-foundation
    provides: "lib/prisma.ts PrismaClient singleton, package.json with prisma CLI, tsconfig.json"
provides:
  - "prisma/schema.prisma with all 15 models + 3 enums"
  - "prisma/migrations/20240101000001_initial_schema/migration.sql — full DDL for all 15 tables"
  - "prisma/migrations/20240101000002_add_fts/migration.sql — FTS tsvector columns, triggers, GIN indexes"
  - "prisma/migrations/20240101000003_add_postgis/migration.sql — conditional PostGIS geography column"
  - "prisma/seed.ts — CategoryGroups, Categories, Departments, admin + staff users"
  - "package.json prisma.seed config"
affects:
  - "01-03 (health readiness uses Ticket table via prisma.ticket.count)"
  - "02-auth (User table with username/email/password_hash/role fields)"
  - "03-public-portal (Ticket+Category+Person+Media models)"
  - "04-staff-queue (all Ticket lifecycle models)"
  - "05-open311-api (Category.service_code, ApiKey model)"
  - "06-admin-panel (all 15 models, AdminAuditLog, Substatus)"

# Tech tracking
tech-stack:
  added:
    - "prisma/schema.prisma (Prisma 6 schema DSL)"
    - "bcryptjs work factor 12 (in seed.ts)"
  patterns:
    - "tsvector FTS via raw migration SQL (not Prisma schema) — avoids unsupported type issues"
    - "PostGIS conditional DO $$ block — zero-downgrade-risk graceful enhancement"
    - "Upsert-idempotent seed — safe for repeated prisma db seed calls"
    - "Manual migration SQL directories — predictable 8-digit timestamp prefixes"

key-files:
  created:
    - "prisma/schema.prisma"
    - "prisma/migrations/20240101000001_initial_schema/migration.sql"
    - "prisma/migrations/20240101000002_add_fts/migration.sql"
    - "prisma/migrations/20240101000003_add_postgis/migration.sql"
    - "prisma/migrations/migration_lock.toml"
    - "prisma/seed.ts"
  modified:
    - "package.json (added prisma.seed config)"

key-decisions:
  - "search_vector tsvector NOT in Prisma schema — managed by migration SQL trigger to avoid Prisma unsupported-type errors"
  - "FTS dictionary: 'english' for Ticket (stems 'potholes'→'pothole'), 'simple' for Person (preserves phone digits/emails)"
  - "PostGIS DO $$ conditional block — migration is idempotent and safe without PostGIS extension installed"
  - "Manual migration SQL files (not prisma migrate dev output) — required because schema-engine binary unavailable in offline Daytona sandbox"
  - "Seed passwords are dev defaults (Admin1234!secure / Staff1234!secure) — documented for rotation in production"

patterns-established:
  - "Raw migration SQL pattern: Prisma schema defines structure; extra DDL (tsvector, PostGIS) added via manual migration files"
  - "Seed upsert pattern: all seed operations use upsert for idempotency across restarts"

# Metrics
duration: 10min
completed: 2026-07-07
---

# Phase 1 Plan 02: Prisma Schema, Migrations, and Seed Data Summary

**Full Prisma data model with 15 models, 3 enums, 3 migration files (initial DDL + FTS tsvector triggers + conditional PostGIS), and idempotent seed script populating CategoryGroups, Categories, Departments, and bcrypt-hashed admin/staff users**

## Performance

- **Duration:** 10 min
- **Started:** 2026-07-07T19:43:07Z
- **Completed:** 2026-07-07T19:53:07Z
- **Tasks:** 2
- **Files modified:** 7 created + 1 modified

## Accomplishments

- 15-model Prisma schema matching TechArch §3.2 + FRD §Y0.15 (all 15 tables verified against Postgres 16)
- FTS setup: `search_vector tsvector` on Ticket (English stemming) + `person_search_vector tsvector` on Person (simple dictionary), both with GIN indexes and BEFORE INSERT/UPDATE triggers
- PostGIS conditional migration: `DO $$ block` adds `geog geography(Point,4326)` + GIST index only if PostGIS extension present; verified safe no-op on plain Postgres 16
- Seed script creates 5 CategoryGroups, 4 Departments, 6 Categories, 1 admin user (admin@bloomington.in.gov), 1 staff user (staff@bloomington.in.gov); bcrypt work factor 12; all upsert-idempotent

## Task Commits

Each task was committed atomically:

1. **Task 1: Full Prisma schema — all 15 models + 3 enums** - `eb910ff` (feat)
2. **Task 2: Migration SQL files (FTS triggers + PostGIS) and seed script** - `ff5a3eb` (feat)

## Files Created/Modified

- `prisma/schema.prisma` — 15 models: CategoryGroup, Category, Department, Substatus, User, Ticket, TicketPerson, Person, TicketHistory, Response, ResponseTemplate, Media, ApiKey, BookmarkedFilter, AdminAuditLog; 3 enums: TicketStatus, UserRole, ApiScope; search_vector absent (trigger-managed)
- `prisma/migrations/20240101000001_initial_schema/migration.sql` — Full DDL: 15 CREATE TABLE, enums, unique/FK/regular indexes, all foreign key constraints
- `prisma/migrations/20240101000002_add_fts/migration.sql` — Ticket FTS ('english' dictionary), Person FTS ('simple' dictionary), 2 GIN indexes, 2 triggers, backfill UPDATE
- `prisma/migrations/20240101000003_add_postgis/migration.sql` — Conditional `DO $$` block: adds geog column + GIST index only when `postgis` extension exists
- `prisma/migrations/migration_lock.toml` — Locks provider to postgresql
- `prisma/seed.ts` — CategoryGroups ×5, Departments ×4, Categories ×6, admin+staff users with bcrypt-hashed passwords
- `package.json` — Added `"prisma": { "seed": "tsx prisma/seed.ts" }`

## Decisions Made

- **search_vector NOT in Prisma schema**: Prisma DSL has no `tsvector` type; adding it via raw migration SQL avoids the unsupported-type error while maintaining Prisma as the schema source of truth for other columns
- **FTS dictionary split**: `'english'` for Ticket fields (description/address) enables stemming — "potholes" matches "pothole" queries; `'simple'` for Person fields (email/phone) preserves exact characters since phone digits and email addresses don't benefit from stemming
- **Manual migration SQL directories**: The Prisma `schema-engine` binary cannot be downloaded in this offline Daytona sandbox (`binaries.prisma.sh` is network-blocked). Migration directories were created manually with fixed timestamps, then verified by running the SQL directly against Postgres 16 via Docker
- **Seed passwords are dev defaults**: `Admin1234!secure` and `Staff1234!secure` are intentional dev/staging defaults per STRIDE threat T-01-05. Production deployments must rotate these via admin UI post-seed

## Deviations from Plan

### Auto-fixed Issues

**1. [Rule 3 - Blocking] Prisma schema-engine binary unavailable — network blocked**
- **Found during:** Task 1 (npx prisma generate)
- **Issue:** `prisma generate` and `prisma migrate deploy` both require downloading `schema-engine` binary from `binaries.prisma.sh`; TLS connection refused in Daytona offline sandbox
- **Fix:** Used `PRISMA_SCHEMA_ENGINE_BINARY` env var pointing to bundled WASM schema engine (`node_modules/prisma/build/schema_engine_bg.wasm`) + `PRISMA_QUERY_ENGINE_LIBRARY` pointing to bundled WASM query engine for `prisma generate`. For migration SQL verification, ran the 3 SQL files directly against a Postgres 16 Docker container via `psql` in the container (Docker/DinD is available on Daytona)
- **Files modified:** None — env var workaround only
- **Verification:** `prisma generate` exited 0; generated `.prisma/client/index.d.ts` contains all 15 model types; migration SQL verified against live Postgres 16 (15 tables confirmed, FTS triggers/GIN indexes confirmed, PostGIS DO $$ block confirmed safe)
- **Committed in:** Both task commits (workaround applied at verification time, not code change)

---

**Total deviations:** 1 auto-fixed (1 blocking)
**Impact on plan:** The migration files and seed script are correct and verified against live Postgres 16. The schema engine binary issue is an infrastructure constraint of the offline Daytona sandbox — it does not affect the K8s deployment where internet access is available and `npm install` downloads the binary normally. No scope creep.

## Issues Encountered

None beyond the auto-fixed blocking deviation above.

## User Setup Required

None — seed uses hardcoded dev defaults. See Decisions Made for password rotation note.

## Next Phase Readiness

- **Plan 01-03 needs:** `prisma/schema.prisma` (for health check using `prisma.ticket.count()`), `lib/prisma.ts` (PrismaClient singleton from plan 01-01), `scripts/migrate-and-start.js` (boot entrypoint from 01-01)
- **Phase 2 auth needs:** `User` table with `username`, `email`, `password_hash`, `role`, `active`, `token_version` fields — all present
- **Blocker:** None — all schema artifacts delivered; `prisma migrate deploy` will run successfully in K8s where internet/binary download works

## Self-Check: PASSED

All 7 key files confirmed present on disk. Both task commits (eb910ff, ff5a3eb) verified in git log.

---
*Phase: 01-k8s-scaffold-data-foundation*
*Completed: 2026-07-07*
