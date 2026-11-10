## Architecture

Layers, top down:

- **Routes** — `app/routes.ts` is the explicit route config; files in `app/routes/` are matched by it, not by filename convention. Each route's `loader`/`action` handles session and access, then calls services. `api.*.ts` routes are resource routes with no UI.
- **Services** — `app/services/*Service.ts` own all database access. Functions are synchronous (better-sqlite3) and take positional parameters, the project convention. Each has a colocated `*.test.ts`.
- **Lib** — `app/lib/` holds pure domain logic (`ppp.ts` pricing, `ratings.ts`, `validation.ts`) and `*.server.ts` server-only helpers.
- **DB** — `app/db/schema.ts` is the single schema; `app/db/index.ts` exports a lazily-opened `db`. Keep `app/db/index.ts` side-effect free: a module-scope connection would pull better-sqlite3 into the client bundle.

Auth is a cookie session (`app/lib/session.ts`) storing `userId`. There are no passwords in dev: `api/switch-user` sets the session user, and `app/components/dev-ui.tsx` exposes the switcher. User roles come from `UserRole` in the schema.

## Tests

Service tests mock `~/db` with an in-memory database built from the real `drizzle/` migrations:

```ts
let testDb: ReturnType<typeof createTestDb>;
vi.mock("~/db", () => ({ get db() { return testDb; } }));

beforeEach(() => {
  testDb = createTestDb();
  base = seedBaseData(testDb);
});
```

`createTestDb` and `seedBaseData` live in `app/test/setup.ts`. A schema change reaches tests only once its migration is generated into `drizzle/`.

## Other directories

- `request-logger/` — course tooling, not app code. It has its own `CLAUDE.md`.
- `.sandcastle/` — sandboxed agent runner config.
- `_internal/` — `feature-ideas.md`, a backlog of feature ideas sized by effort.
