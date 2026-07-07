---
phase: 01-k8s-scaffold-data-foundation
plan: "01"
subsystem: infra
tags: [next.js, prisma, typescript, postgresql, tailwindcss, next-auth]

# Dependency graph
requires: []
provides:
  - "package.json with all runtime deps (Next.js 15, React 19, Prisma 6, next-auth beta)"
  - "tsconfig.json for Next.js 15 TypeScript compilation"
  - "next.config.ts with no X-Frame-Options (Pivota iframe safe)"
  - "infrastructure.json declaring postgres sidecar on port 3000"
  - "scripts/migrate-and-start.js K8s pod entrypoint"
  - "lib/prisma.ts PrismaClient singleton"
  - "lib/logger.ts structured JSON logger"
  - "types/domain.ts core domain types"
  - "types/auth.ts NextAuth session augmentation"
affects:
  - "02-k8s-scaffold-data-foundation (Prisma schema depends on lib/prisma.ts)"
  - "all subsequent phases (depend on package.json, tsconfig, next.config.ts)"

# Tech tracking
tech-stack:
  added:
    - "next@^15.0.0 (App Router, React 19)"
    - "react@^19.0.0"
    - "@prisma/client@^6.0.0 + prisma@^6.0.0"
    - "next-auth@beta (v5 credentials provider)"
    - "tailwindcss@^4.0.0 + @tailwindcss/postcss@^4.0.0"
    - "pg@^8.12.0 (native Postgres client for boot script)"
    - "leaflet@^1.9.4 + react-leaflet@^4.2.1"
    - "recharts@^2.12.0"
    - "vitest@^2.1.0 + playwright@^1.47.0"
    - "zod@^3.23.0"
  patterns:
    - "PrismaClient singleton via globalThis hot-reload safety"
    - "Structured JSON stdout logging (no PII)"
    - "K8s boot entrypoint: validate env → wait for DB → migrate → start"
    - "next.config.ts (.ts extension, no X-Frame-Options)"
    - "infrastructure.json sidecar declaration"

key-files:
  created:
    - "package.json"
    - "tsconfig.json"
    - "next.config.ts"
    - "infrastructure.json"
    - "app/layout.tsx"
    - "app/globals.css"
    - "types/domain.ts"
    - "types/auth.ts"
    - "scripts/migrate-and-start.js"
    - "lib/prisma.ts"
    - "lib/logger.ts"
  modified:
    - ".gitignore (added !scripts/*.js exception)"

key-decisions:
  - "No X-Frame-Options in next.config.ts — Pivota Preview embeds app in iframe; intentionally omitted"
  - "next.config.ts uses .ts extension (required by Next.js 15+, not .js)"
  - "0.0.0.0 binding in boot script — required for K8s health probe proxy"
  - "next-auth@beta (not ^5.0.0 — semver range doesn't match beta tags)"
  - "--legacy-peer-deps for npm install — react-leaflet v4 has React 18 peer dep, React 19 in use"
  - ".gitignore !scripts/*.js exception — legacy *-*.js pattern was blocking migrate-and-start.js"

patterns-established:
  - "PrismaClient singleton: globalForPrisma via globalThis, dev-only hot-reload safety"
  - "Structured logger: write() with level filtering via LOG_LEVEL env, JSON to stdout"
  - "Boot entrypoint pattern: env-validate → DB wait with exponential backoff → migrate → seed? → start"
  - "infrastructure.json at repo root declares sidecar_requirements for Pivota platform"

# Metrics
duration: 5 min
completed: 2026-07-07
---

# Phase 1 Plan 01: K8s Scaffold — Package & Boot Foundation Summary

**Next.js 15 greenfield scaffold with Prisma 6 singleton, structured JSON logger, and K8s-native boot entrypoint that validates env, waits for Postgres, migrates, and starts on 0.0.0.0:3000**

## Performance

- **Duration:** 5 min
- **Started:** 2026-07-07T19:35:34Z
- **Completed:** 2026-07-07T19:40:19Z
- **Tasks:** 2
- **Files modified:** 11 created + 1 modified

## Accomplishments

- Greenfield Next.js 15 TypeScript scaffold alongside legacy PHP (no PHP files touched)
- infrastructure.json declaring postgres sidecar enables Pivota platform provisioning
- Boot entrypoint validates DATABASE_URL + AUTH_SECRET, waits for DB with exponential backoff, runs prisma migrate deploy, binds Next.js to 0.0.0.0:3000
- Complete domain type system (types/domain.ts, types/auth.ts) from TechArch spec

## Task Commits

Each task was committed atomically:

1. **Task 1: Project scaffold** - `89b0c1d` (feat)
2. **Task 2: Boot entrypoint + Prisma singleton + structured logger** - `556e4be` (feat)

## Files Created/Modified

- `package.json` - All runtime + dev dependencies (Next.js 15, React 19, Prisma 6, next-auth beta, Tailwind v4, vitest, playwright)
- `tsconfig.json` - Next.js 15 TypeScript config with bundler module resolution, strict mode
- `next.config.ts` - Next.js 15 config with safe security headers (no X-Frame-Options for Pivota iframe)
- `infrastructure.json` - `{ "sidecar_requirements": ["postgres"], "port": 3000 }` for Pivota platform
- `app/layout.tsx` - Minimal root layout required by Next.js 15 App Router
- `app/globals.css` - Tailwind v4 `@import "tailwindcss"` import
- `types/domain.ts` - Core domain types: TicketSummary, TicketDetail, PersonRecord, CategoryRecord, PaginatedResponse, etc.
- `types/auth.ts` - NextAuth v5 session augmentation for staff/admin roles
- `scripts/migrate-and-start.js` - K8s pod entrypoint: env validation → DB wait → prisma migrate deploy → next start -H 0.0.0.0
- `lib/prisma.ts` - PrismaClient singleton with dev hot-reload safety via globalThis
- `lib/logger.ts` - Structured JSON stdout logger with LOG_LEVEL control, no PII
- `.gitignore` - Added `!scripts/*.js` exception for legacy `*-*.js` blocking rule

## Decisions Made

- **next-auth@beta**: `^5.0.0` semver range doesn't match beta pre-release tags; using `beta` dist-tag to get latest v5 beta (5.0.0-beta.31)
- **No X-Frame-Options**: Intentionally omitted per Pivota platform constraint; app must be embeddable in preview iframe
- **next.config.ts extension**: Next.js 15+ requires `.ts` extension for config; `.js` is not supported
- **0.0.0.0 binding**: K8s health probes connect over IPv4; binding to `localhost` can resolve to IPv6 `::1` and fail
- **--legacy-peer-deps**: react-leaflet@4.2.1 declares React 18 peer dep; React 19 in use; install proceeds with legacy resolution

## Deviations from Plan

### Auto-fixed Issues

**1. [Rule 3 - Blocking] next-auth@^5.0.0 semver range resolves nothing**
- **Found during:** Task 1 (npm install)
- **Issue:** `next-auth@^5.0.0` finds no matching package — v5 is published as pre-release (`5.0.0-beta.x`), which semver `^5.0.0` excludes
- **Fix:** Changed `"next-auth": "^5.0.0"` → `"next-auth": "beta"` to resolve to `5.0.0-beta.31`
- **Files modified:** `package.json`
- **Verification:** npm install succeeded
- **Committed in:** 89b0c1d (Task 1 commit)

**2. [Rule 3 - Blocking] react-leaflet@4.2.1 peer dep conflict with React 19**
- **Found during:** Task 1 (npm install)
- **Issue:** `react-leaflet@4.2.1` requires `peer react@^18.0.0` but project uses React 19; npm ERESOLVE error
- **Fix:** Added `--legacy-peer-deps` flag to npm install
- **Files modified:** None (install flag only)
- **Verification:** npm install succeeded with 508 packages
- **Committed in:** 89b0c1d (Task 1 commit)

**3. [Rule 3 - Blocking] Legacy .gitignore *-*.js pattern blocked migrate-and-start.js**
- **Found during:** Task 2 (git status after writing scripts/migrate-and-start.js)
- **Issue:** Legacy PHP `.gitignore` rule `*-*.js` matched `migrate-and-start.js` (contains hyphen); file was silently excluded from git tracking
- **Fix:** Added `!scripts/*.js` exception before the Pivota-managed block; used `git add -f` for initial stage
- **Files modified:** `.gitignore`
- **Verification:** `git ls-files scripts/migrate-and-start.js` shows file tracked
- **Committed in:** 556e4be (Task 2 commit)

---

**Total deviations:** 3 auto-fixed (3 blocking)
**Impact on plan:** All fixes necessary for installation and version resolution. No scope creep. Plan artifacts delivered exactly as specified.

## Issues Encountered

None beyond the auto-fixed blocking deviations above.

## User Setup Required

None — no external service configuration required. The Postgres sidecar is platform-provided via `DATABASE_URL` and `PIVOTA_DB_MODE=sidecar-postgres` when deployed to Pivota Kubernetes. For local development, set `DATABASE_URL` pointing to a local Postgres instance.

## Next Phase Readiness

- **Plan 01-02 needs from this plan:** `lib/prisma.ts` (PrismaClient export), `package.json` (prisma CLI in devDependencies for `prisma generate` and schema commands), `tsconfig.json` (TypeScript resolution for generated Prisma types)
- **All subsequent plans need:** `package.json` for dependency resolution, `tsconfig.json` for TypeScript compilation, `next.config.ts` for Next.js configuration
- **Blocker:** None — all artifacts delivered and verified

## Self-Check: PASSED

All 11 key files found on disk. Both task commits (89b0c1d, 556e4be) verified in git log.

---
*Phase: 01-k8s-scaffold-data-foundation*
*Completed: 2026-07-07*
