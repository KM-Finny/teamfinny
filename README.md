# BLUE_FINNY

Minimal README for running, building and managing voucher prefixes in the KM_FINNY_MERGE project.

## Project overview
A Node + React (Vite) application with a Postgres database (Drizzle ORM). Admins can centrally manage voucher prefixes (Expense / Toll) via the Settings page. Prefixes are stored in the `voucher_prefixes` DB table.

## Prerequisites
- Node 18+ and npm
- PostgreSQL server (accessible from app)
- Optional: psql CLI or pgAdmin for DB inspection and manual SQL
- On macOS: Homebrew to install psql (`brew install libpq` then add bin to PATH)

## Environment
Create a `.env` in project root or set env vars in your system:
```
DATABASE_URL=postgresql://dbuser:dbpass@host:5432/km_finny
PORT=3000
NODE_ENV=development
# any other env used by your repo
```

## Install dependencies
```bash
# fresh install
npm ci
# or
npm install
```

## Run in development
1. Start backend (example):
```bash
npm run dev
```
2. Start frontend (if separate):
```bash
npm run dev:client
# or the repo's script that runs Vite
```

Open app at http://localhost:<PORT>.

## Build for production
Ensure dependencies (including dev tools) are installed before running build:
```bash
npm ci
npm run build
```
If you see `'vite' is not recognized`, make sure `node_modules` exist (run `npm ci`) or run `npx vite build` in CI/build scripts.

## Database: migrations & schema
- Prefer migrations (deterministic) over `drizzle-kit push`.
- Generate/apply migrations:
```bash
# apply pending migrations
npm run db:migrate

# generate a migration (if using drizzle-kit generate)
npx drizzle-kit generate
```
If migrate errors occur complaining about existing relations (42P07) or primary-key constraint (42P16), the DB and migration history may be out of sync. See Troubleshooting.

## Seed / create voucher_prefixes rows (manual)
If `voucher_prefixes` exists but is empty, insert safe defaults:

Run these in pgAdmin Query Tool or via psql:
```sql
-- safe insert if row not present
INSERT INTO voucher_prefixes (type, prefix, updated_by, created_at, updated_at)
SELECT 'expense', 'KM2526-EV-', 'manual_seed', now(), now()
WHERE NOT EXISTS (SELECT 1 FROM voucher_prefixes WHERE type = 'expense');

INSERT INTO voucher_prefixes (type, prefix, updated_by, created_at, updated_at)
SELECT 'toll', 'KM2526-TV-', 'manual_seed', now(), now()
WHERE NOT EXISTS (SELECT 1 FROM voucher_prefixes WHERE type = 'toll');
```

Optional: add a unique index to allow ON CONFLICT upserts later:
```sql
CREATE UNIQUE INDEX IF NOT EXISTS voucher_prefixes_type_idx ON voucher_prefixes (type);
```

Alternative two-step upsert (no index required):
```sql
BEGIN;
UPDATE voucher_prefixes
SET prefix = 'KM2527-EV-', updated_by = 'admin_user', updated_at = now()
WHERE type = 'expense';

INSERT INTO voucher_prefixes (type, prefix, updated_by, updated_at)
SELECT 'expense','KM2527-EV-','admin_user', now()
WHERE NOT EXISTS (SELECT 1 FROM voucher_prefixes WHERE type='expense');
COMMIT;
```

## Manage voucher prefixes (UI)
- Open Settings -> Application -> Voucher Prefixes.
- Only admin users can edit/save.
- The Settings page issues PUT requests:
  - PUT /api/voucher-prefixes/expense
  - PUT /api/voucher-prefixes/toll
- After save, clients broadcast a timestamp to localStorage (`voucher_prefixes_updated`) and pages refetch to reflect changes. For cross-device real-time updates, use SSE/WebSockets or rely on polling.

## Typical troubleshooting

- "psql: command not found"
  - Install psql (macOS via Homebrew: `brew install libpq` and add `/opt/homebrew/opt/libpq/bin` to PATH).

- Drizzle migrate errors like `relation "activities" already exists` (42P07)
  - DB already has tables that migrations expect. Options:
    - Mark existing migrations as applied in the migrations table (non-destructive baseline).
    - Or, if safe, drop public schema and re-run migrations (destructive).
  - Check the DB tables and `drizzle_migrations` (or similar) table to align state.

- drizzle-kit push error `column "id" is in a primary key` (42P16)
  - Means a DDL tried to modify a PK column; prefer creating migrations that alter constraints in safe steps or apply manual SQL fixes.

- Build error `'vite' is not recognized`
  - Ensure dev deps installed: `npm ci`
  - Or change build scripts to use `npx vite build` in CI.

## Adding a new DB column (example)
If you added a new column (e.g., `year_token`), create a migration SQL file:
```sql
ALTER TABLE voucher_prefixes ADD COLUMN year_token varchar(32);

-- optional backfill from existing prefix
UPDATE voucher_prefixes
SET year_token = regexp_replace(prefix, '^.*(KM\\d{4}-).*$','\1')
WHERE prefix IS NOT NULL;
```
Then run:
```bash
npm run db:migrate
```
Also update Drizzle schema and server code to use the new column and rebuild.

## Production deploy (notes)
- If using GitHub Actions, ensure workflow installs devDependencies before build.
- Build step should run `npm ci` then `npm run build`.
- Provide environment secrets (DATABASE_URL, SSH key, etc.) to your CI.
- Restart process manager (pm2/systemd) or deploy service after building.

## Useful commands
- Run server in dev: `npm run dev`
- Apply migrations: `npm run db:migrate`
- Push schema (not recommended for production): `npm run db:push`
- Build: `npm run build`
- Seed prefixes manually: use SQL above in pgAdmin or psql

## If something still doesn't update
- Verify PUT request succeeds (Network tab). Expected 200 and a JSON response with updated data.
- Check backend logs for errors during save.
- Verify DB rows with:
```sql
SELECT id, type, prefix, updated_by, created_at, updated_at FROM voucher_prefixes ORDER BY id;
```

## Contact / help
- For code changes: edit client file `client/src/pages/Settings.tsx`.
- Server route: `server/routes/voucher-prefix.ts`.
- DB config: `drizzle.config.ts` and `server/db.ts`.
- If you want me to create migration/seed files or a GitHub Actions workflow file, respond with which file you want added.

```// filepath: /README.md
# KM_FINNY_MERGE

Minimal README for running, building and managing voucher prefixes in the KM_FINNY_MERGE project.

## Project overview
A Node + React (Vite) application with a Postgres database (Drizzle ORM). Admins can centrally manage voucher prefixes (Expense / Toll) via the Settings page. Prefixes are stored in the `voucher_prefixes` DB table.

## Prerequisites
- Node 18+ and npm
- PostgreSQL server (accessible from app)
- Optional: psql CLI or pgAdmin for DB inspection and manual SQL
- On macOS: Homebrew to install psql (`brew install libpq` then add bin to PATH)

## Environment
Create a `.env` in project root or set env vars in your system:
```
DATABASE_URL=postgresql://dbuser:dbpass@host:5432/km_finny
PORT=3000
NODE_ENV=development
# any other env used by your repo
```

## Install dependencies
```bash
# fresh install
npm ci
# or
npm install
```

## Run in development
1. Start backend (example):
```bash
npm run dev
```
2. Start frontend (if separate):
```bash
npm run dev:client
# or the repo's script that runs Vite
```

Open app at http://localhost:<PORT>.

## Build for production
Ensure dependencies (including dev tools) are installed before running build:
```bash
npm ci
npm run build
```
If you see `'vite' is not recognized`, make sure `node_modules` exist (run `npm ci`) or run `npx vite build` in CI/build scripts.

## Database: migrations & schema
- Prefer migrations (deterministic) over `drizzle-kit push`.
- Generate/apply migrations:
```bash
# apply pending migrations
npm run db:migrate

# generate a migration (if using drizzle-kit generate)
npx drizzle-kit generate
```
If migrate errors occur complaining about existing relations (42P07) or primary-key constraint (42P16), the DB and migration history may be out of sync. See Troubleshooting.

## Seed / create voucher_prefixes rows (manual)
If `voucher_prefixes` exists but is empty, insert safe defaults:

Run these in pgAdmin Query Tool or via psql:
```sql
-- safe insert if row not present
INSERT INTO voucher_prefixes (type, prefix, updated_by, created_at, updated_at)
SELECT 'expense', 'KM2526-EV-', 'manual_seed', now(), now()
WHERE NOT EXISTS (SELECT 1 FROM voucher_prefixes WHERE type = 'expense');

INSERT INTO voucher_prefixes (type, prefix, updated_by, created_at, updated_at)
SELECT 'toll', 'KM2526-TV-', 'manual_seed', now(), now()
WHERE NOT EXISTS (SELECT 1 FROM voucher_prefixes WHERE type = 'toll');
```

Optional: add a unique index to allow ON CONFLICT upserts later:
```sql
CREATE UNIQUE INDEX IF NOT EXISTS voucher_prefixes_type_idx ON voucher_prefixes (type);
```

Alternative two-step upsert (no index required):
```sql
BEGIN;
UPDATE voucher_prefixes
SET prefix = 'KM2527-EV-', updated_by = 'admin_user', updated_at = now()
WHERE type = 'expense';

INSERT INTO voucher_prefixes (type, prefix, updated_by, updated_at)
SELECT 'expense','KM2527-EV-','admin_user', now()
WHERE NOT EXISTS (SELECT 1 FROM voucher_prefixes WHERE type='expense');
COMMIT;
```

## Manage voucher prefixes (UI)
- Open Settings -> Application -> Voucher Prefixes.
- Only admin users can edit/save.
- The Settings page issues PUT requests:
  - PUT /api/voucher-prefixes/expense
  - PUT /api/voucher-prefixes/toll
- After save, clients broadcast a timestamp to localStorage (`voucher_prefixes_updated`) and pages refetch to reflect changes. For cross-device real-time updates, use SSE/WebSockets or rely on polling.

## Typical troubleshooting

- "psql: command not found"
  - Install psql (macOS via Homebrew: `brew install libpq` and add `/opt/homebrew/opt/libpq/bin` to PATH).

- Drizzle migrate errors like `relation "activities" already exists` (42P07)
  - DB already has tables that migrations expect. Options:
    - Mark existing migrations as applied in the migrations table (non-destructive baseline).
    - Or, if safe, drop public schema and re-run migrations (destructive).
  - Check the DB tables and `drizzle_migrations` (or similar) table to align state.

- drizzle-kit push error `column "id" is in a primary key` (42P16)
  - Means a DDL tried to modify a PK column; prefer creating migrations that alter constraints in safe steps or apply manual SQL fixes.

- Build error `'vite' is not recognized`
  - Ensure dev deps installed: `npm ci`
  - Or change build scripts to use `npx vite build` in CI.

## Adding a new DB column (example)
If you added a new column (e.g., `year_token`), create a migration SQL file:
```sql
ALTER TABLE voucher_prefixes ADD COLUMN year_token varchar(32);

-- optional backfill from existing prefix
UPDATE voucher_prefixes
SET year_token = regexp_replace(prefix, '^.*(KM\\d{4}-).*$','\1')
WHERE prefix IS NOT NULL;
```
Then run:
```bash
npm run db:migrate
```
Also update Drizzle schema and server code to use the new column and rebuild.

## Production deploy (notes)
- If using GitHub Actions, ensure workflow installs devDependencies before build.
- Build step should run `npm ci` then `npm run build`.
- Provide environment secrets (DATABASE_URL, SSH key, etc.) to your CI.
- Restart process manager (pm2/systemd) or deploy service after building.

## Useful commands
- Run server in dev: `npm run dev`
- Apply migrations: `npm run db:migrate`
- Push schema (not recommended for production): `npm run db:push`
- Build: `npm run build`
- Seed prefixes manually: use SQL above in pgAdmin or psql

## If something still doesn't update
- Verify PUT request succeeds (Network tab). Expected 200 and a JSON response with updated data.
- Check backend logs for errors during save.
- Verify DB rows with:
```sql
SELECT id, type, prefix, updated_by, created_at, updated_at FROM voucher_prefixes ORDER BY id;
```

## Contact / help
- For code changes: edit client file `client/src/pages/Settings.tsx`.
- Server route: `server/routes/voucher-prefix.ts`.
- DB config: `drizzle.config.ts` and `server/db.ts`.
- If you want me to create migration/seed files or a GitHub Actions workflow file, respond with which file you want added.
