---
phase: 01-k8s-scaffold-data-foundation
plan: "03"
subsystem: infra
tags: [nextjs, health-endpoints, postgis, haversine, geo, k8s, prisma]

# Dependency graph
requires:
  - phase: 01-k8s-scaffold-data-foundation
    provides: "lib/prisma.ts (prisma client), lib/logger.ts (structured logging), prisma/schema.prisma (Ticket model)"
provides:
  - "K8s liveness probe: GET /api/health/live → { status: 'ok', timestamp }"
  - "K8s readiness probe: GET /api/health/ready → { status: 'ready', db, migrations } or 503"
  - "lib/geo.ts: detectGeoMode(), GEO_MODE, distanceMeters(), buildBboxFilter(), parseBbox()"
  - "lib/api-response.ts: ok(), apiError(), requireSession()"
  - "scripts/migrate-and-start.js: full 6-step startup sequence with PostGIS detection"
affects: ["02-authentication-sessions", "03-public-portal", "all subsequent phases"]

# Tech tracking
tech-stack:
  added: ["pg (direct client for startup geo detection)"]
  patterns:
    - "GEO_MODE global: globalThis used to survive Next.js hot-reload; module-level var shadows global"
    - "Dual-path geo: detectGeoMode() sets GEO_MODE at startup; all geo queries branch on this"
    - "Health endpoint separation: /live (no DB) vs /ready (DB ping) — canonical K8s probe pattern"
    - "Startup script geo detection via pg client (not Prisma) — avoids Prisma bootstrap cost at boot"

key-files:
  created:
    - lib/geo.ts
  modified:
    - scripts/migrate-and-start.js
  already-present-from-prior-work:
    - app/api/health/live/route.ts
    - app/api/health/ready/route.ts
    - lib/api-response.ts
    - lib/auth.ts
    - app/page.tsx

key-decisions:
  - "PostGIS detection uses pg client directly in boot script (not Prisma) — avoids full ORM bootstrap cost and keeps the detection self-contained"
  - "GEO_MODE stored on globalThis so it survives Next.js hot-module replacement in dev"
  - "lib/auth.ts stub created in Phase 1 to allow lib/api-response.ts to compile — full Auth.js implementation deferred to Phase 2"
  - "X-Frame-Options intentionally absent from next.config.ts headers — Pivota Preview embeds app in iframe (confirmed from plan 01-01)"

patterns-established:
  - "Health probes: /live never touches DB; /ready executes SELECT 1 and returns 503 on failure"
  - "Geo queries: always check GEO_MODE before deciding between PostGIS raw SQL and Haversine math"
  - "API responses: use ok()/apiError() from lib/api-response.ts for consistent envelope"
  - "Startup sequence: waitForDb → migrate deploy → optional seed → detectGeoMode → next start"

# Metrics
duration: 3min
completed: 2026-07-07
---

# Phase 1 Plan 3: Health Endpoints, Geo Detection & API Helpers Summary

**K8s liveness/readiness probes, PostGIS detection with Haversine fallback (lib/geo.ts), and shared API response helpers — completes Phase 1 foundation**

## Performance

- **Duration:** 3 min
- **Started:** 2026-07-07T20:18:26Z
- **Completed:** 2026-07-07T20:21:27Z
- **Tasks:** 2
- **Files modified:** 3 (1 created, 1 updated, 1 type-check artifact)

## Accomplishments

- Created `lib/geo.ts` implementing PostGIS detection (`detectGeoMode()`), Haversine formula (`distanceMeters()` with R=6371000m), bbox filtering (`buildBboxFilter()`), and bbox parsing (`parseBbox()`)
- Updated `scripts/migrate-and-start.js` with step 5: PostGIS detection via direct `pg` client before spawning Next.js, passing `GEO_MODE` env var to the child process
- Verified all Task 1 artifacts (health endpoints, api-response.ts, page.tsx, auth stub) were present and correct from prior work in the initial commit

## Task Commits

Each task was committed atomically:

1. **Task 1: Health endpoints — /api/health/live, /api/health/ready, lib/api-response.ts, app/page.tsx** - Pre-existing in `db37fb6` (feat(phase-1): execution complete) — artifacts verified correct, no changes needed
2. **Task 2: PostGIS detection + Haversine fallback + startup integration** - `3a85c0b` (feat(01-03))

**Plan metadata:** pending docs commit (docs(01-03))

## Files Created/Modified

- `lib/geo.ts` — PostGIS detection, Haversine formula, bbox filter helpers (GeoMode type, 4 exports)
- `scripts/migrate-and-start.js` — Added `detectGeoMode()` function and step 5 in startup sequence
- `app/api/health/live/route.ts` — K8s liveness probe (already correct from initial commit)
- `app/api/health/ready/route.ts` — K8s readiness probe with prisma.$queryRaw and 503 fallback (already correct)
- `lib/api-response.ts` — ok(), apiError(), requireSession() helpers (already correct)
- `lib/auth.ts` — Phase 1 stub returning null (enables api-response.ts compilation)
- `app/page.tsx` — Minimal placeholder landing page (already correct)

## Decisions Made

- **PostGIS detection uses `pg` client directly** in the boot script (not Prisma) to avoid the full ORM bootstrap cost and keep detection self-contained at startup
- **`GEO_MODE` stored on `globalThis`** so it survives Next.js hot-module replacement in dev; module-level `let GEO_MODE` shadows and initializes from globalThis
- **lib/auth.ts stub** created in Phase 1 to allow lib/api-response.ts (which imports `auth`) to compile — full Auth.js config deferred to Phase 2
- **X-Frame-Options verification note**: The grep check for "X-Frame-Options" matched on *comments* in next.config.ts explaining its absence — the header is correctly not set in the actual headers array

## Deviations from Plan

None - plan executed exactly as written. Task 1 artifacts were pre-created as part of the initial repository setup commit (`db37fb6`); all were verified correct and matched the plan specifications exactly. Task 2 (lib/geo.ts + migrate-and-start.js update) was the primary new work.

## Issues Encountered

- **False positive verification alert**: The `grep -q "X-Frame-Options"` check in the plan verification script matches the comment text "DO NOT add X-Frame-Options" in next.config.ts. The actual headers array correctly omits X-Frame-Options. Not a real issue — the config is correct.

## Phase 1 Overall: All 5 Success Criteria Met ✓

| Criterion | Status | Evidence |
|-----------|--------|----------|
| SC1: App starts port 3000, single process (INFRA-01) | ✓ | scripts/migrate-and-start.js spawns `next start -p 3000 -H 0.0.0.0` |
| SC2: infrastructure.json declares postgres sidecar (INFRA-03) | ✓ | `{"sidecar_requirements":["postgres"],"port":3000}` |
| SC3: All Prisma models present, migrations run cleanly (DATA-01) | ✓ | 15 models in schema.prisma, 3 migration files |
| SC4: search_vector GIN trigger + PostGIS conditional migration (DATA-02, DATA-03) | ✓ | `ticket_search_vector_trigger` in migration, DO $$ PostGIS block |
| SC5: Seed creates CategoryGroups + Categories + admin + staff (DATA-04) | ✓ | prisma/seed.ts has CategoryGroup, UserRole enum |

## Next Phase Readiness

Phase 2 (Authentication & Sessions) can proceed immediately. It needs:
- `lib/prisma.ts` — for User table queries ✓
- `lib/logger.ts` — for auth error logging ✓
- `lib/api-response.ts` — for `requireSession()` helper ✓ (stub auth currently returns null)
- `prisma/schema.prisma` User model — for bcrypt + token_version checks ✓

**Full Auth.js implementation in Phase 2 will replace the `lib/auth.ts` stub.**

---
*Phase: 01-k8s-scaffold-data-foundation*
*Completed: 2026-07-07*
