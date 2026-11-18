# Payload Template Project

Internal starter template for building customer websites, landing pages, and SaaS
applications on a consistent stack. Bootstrap new projects from this repo instead of
starting from scratch — auth, CMS, storage, and UI conventions are already wired up.

## Stack

- **Framework**: [Next.js](https://nextjs.org) 16 (App Router)
- **CMS**: [Payload CMS](https://payloadcms.com) 3
- **Database**: PostgreSQL (via `@payloadcms/db-postgres`)
- **Auth**: [Clerk](https://clerk.com)
- **Storage**: S3-compatible object storage (media uploads)
- **Email**: [Resend](https://resend.com)
- **UI**: Tailwind CSS 4, [shadcn/ui](https://ui.shadcn.com), Lucide icons, Motion

## Requirements

- Node.js `^22.14.0` or `>=24.10.0`
- pnpm `^9`, `^10`, `^11`, or `^12`
- A PostgreSQL connection string (local, Docker, or hosted)

## Setup

1. Clone the repo and install dependencies:

   ```bash
   pnpm install
   ```

2. Copy the environment file and fill in the values:

   ```bash
   cp .env.example .env
   ```

   | Variable | Purpose |
   | --- | --- |
   | `DATABASE_URL` | PostgreSQL connection string (`postgres://user:pass@host:5432/db`) |
   | `PAYLOAD_SECRET` | Payload's signing secret |
   | `PREVIEW_SECRET` | Secret used for live preview links |
   | `CRON_SECRET` | Secret for authenticating scheduled/cron jobs |
   | `NEXT_PUBLIC_SERVER_URL` | Public URL the app is served from |
   | `CLERK_SECRET_KEY` / `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY` | Clerk API keys |
   | `CLERK_WEBHOOK_SIGNING_SECRET` | Verifies incoming Clerk webhooks |
   | `NEXT_PUBLIC_CLERK_SIGN_IN_URL`, `..._SIGN_IN_FALLBACK_REDIRECT_URL`, `..._SIGN_UP_FALLBACK_REDIRECT_URL` | Clerk auth flow routing |
   | `S3_BUCKET`, `S3_ACCESS_KEY_ID`, `S3_ACCESS_KEY_SECRET`, `S3_REGION`, `S3_ENDPOINT` | Media storage (S3-compatible) |
   | `RESEND_API_KEY`, `RESEND_FROM_EMAIL`, `RESEND_FROM_NAME` | Transactional email |

3. Apply the database schema:

   ```bash
   pnpm payload migrate
   ```

   The Postgres adapter runs with `push: false` (`src/payload.config.ts`), so tables are
   created only by migrations. Skip this on a fresh database and every query fails with
   `relation "pages" does not exist` (Postgres `42P01`), which appears during
   `pnpm build` as `Failed to collect page data for /[slug]`. The command is safe to
   re-run: applied migrations are tracked in `payload_migrations`.

4. Start the dev server:

   ```bash
   pnpm dev
   ```

5. Open `http://localhost:3000/admin`. There is no local first-user screen: the Users
   collection disables the local strategy and authenticates through Clerk. Create your
   user in Clerk, set its public metadata to `{ "role": "admin" }`, then sign in at
   `/admin`. The default role is `user`, which cannot enter the admin panel; the Clerk
   login strategy (or the `user.created` webhook, when reachable) creates the matching
   Payload record with the role from that metadata.

### Local PostgreSQL (optional)

`docker-compose.yml` deploys the application itself and expects an external
PostgreSQL instance. For local development you can run PostgreSQL with Docker:

1. Start a PostgreSQL container:

   ```bash
   docker run --name payload-postgres -e POSTGRES_PASSWORD=postgres -p 5432:5432 -d postgres:17
   ```

2. Set `DATABASE_URL` in `.env`, e.g.
   `postgres://postgres:postgres@127.0.0.1:5432/payload`.
3. Apply the schema: `pnpm payload migrate`.

### Database Migrations

The Postgres adapter runs with `push: false`, so schema changes are applied through
migrations rather than automatic dev-time pushes. After changing any collection,
global, or field config:

1. Generate a migration and review it:

   ```bash
   pnpm payload migrate:create
   ```

2. Commit the new files under `src/migrations/`.
3. Apply pending migrations to the database:

   ```bash
   pnpm payload migrate
   ```

CI applies migrations before building the image
(`.github/workflows/push-to-ghcr.yml`), and the build prerenders routes against the
database, so migrations must be committed before the workflow runs.

### First run and empty databases

Run `pnpm payload migrate` once before the first `pnpm dev` or `pnpm build`, and again
after recreating the database or switching `DATABASE_URL` to a database that has not
been migrated. Verify with `pnpm payload migrate:status` (the initial migration is
`20260925_210034`).

If a table appears to be missing, confirm you migrated the database you are actually
connected to, not just one in Docker or on another port.

Symptoms of an unmigrated database:

- `error: relation "pages" does not exist` (`code: '42P01'`)
- `Failed to collect page data for /[slug]` during `pnpm build`
- `relation "payload_migrations" does not exist` from `migrate:status`

## Available Scripts

| Script | Purpose |
| --- | --- |
| `pnpm dev` | Start the Next.js dev server |
| `pnpm build` | Production build |
| `pnpm start` | Serve the production build |
| `pnpm lint` | Run ESLint |
| `pnpm payload` | Run the Payload CLI |
| `pnpm generate:types` | Regenerate Payload's TypeScript types from the config |
| `pnpm generate:importmap` | Regenerate Payload's admin import map |
| `pnpm payload migrate:create` | Generate a migration after config changes |
| `pnpm payload migrate` | Apply pending migrations |
| `pnpm payload migrate:status` | Show applied and pending migrations |

Run `pnpm generate:types` after changing any collection, global, or field config so
generated types stay in sync, and `pnpm payload migrate:create` so the database schema
stays in sync with the config.

## Project Structure

- `src/` — application code (Next.js routes, Payload config, collections, components)
- `context/` — living documentation (architecture, UI tokens/rules, code standards,
  build plan, progress tracker) used to keep new projects built from this template
  consistent
- `docker-compose.yml` — production compose for the app (expects an external
  PostgreSQL instance)

## Collections

- **Users** — auth-enabled collection with admin panel access, backed by Clerk.
- **Media** — uploads collection with pre-configured image sizes and focal point
  support, backed by S3-compatible storage.

See the [Payload Collections docs](https://payloadcms.com/docs/configuration/collections)
to extend either.

## Using This as a Starter

When bootstrapping a new internal project from this template:

1. Update `package.json` name/description and this README's title.
2. Review `context/` and update it to describe the new project's purpose, scope, and
   architecture — it should not still describe this template once customized.
3. Keep the Clerk, Payload, S3, and Resend wiring unless the new project has a reason
   to diverge — the point of the template is a consistent baseline across projects.

## Support

Internal questions: ask in the team channel. For upstream framework issues, see the
[Payload Discord](https://discord.com/invite/payload) or
[Payload GitHub discussions](https://github.com/payloadcms/payload/discussions).
