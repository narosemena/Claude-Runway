# CLAUDE.md Examples by Stack

Concrete examples of well-structured CLAUDE.md files for common stacks.
Each demonstrates the principles: under 300 lines, no platitudes, project-specific
only, with executable verification commands.

---

## Example 1: Next.js 15 + TypeScript + Supabase

```markdown
# WidgetDash — Customer Analytics Dashboard

Real-time analytics dashboard for tracking widget usage metrics.
Internal tool, not public-facing.

## Stack
- Next.js 15.1 (App Router)
- TypeScript 5.6 (strict mode)
- Supabase (auth + PostgreSQL + realtime subscriptions)
- Zustand 5.x for client state
- Tailwind CSS 4.x
- Vitest + React Testing Library

## Architecture
- `app/` — App Router pages. All pages are Server Components by default.
- `app/(dashboard)/` — Authenticated route group. Layout handles auth check.
- `components/` — Reusable UI. Split into `components/ui/` (primitives) and `components/features/` (domain).
- `lib/supabase/` — Supabase client config. `server.ts` for RSC, `client.ts` for client components. Never import server client in a 'use client' file.
- `lib/queries/` — All database queries as standalone async functions. No inline SQL in components.
- `stores/` — Zustand stores. One store per domain (userStore, metricsStore).

## Verification
Before completing any task:
- `pnpm typecheck` — must pass with zero errors
- `pnpm test` — all tests must pass
- `pnpm lint` — zero warnings
- `pnpm build` — must complete successfully

## Testing
- Framework: Vitest + React Testing Library
- Location: `__tests__/` mirroring `app/` and `components/` structure
- Approach: Test-first for business logic and data queries. Test-after for UI components.
- Run: `pnpm test` (all) or `pnpm test [file]` (specific)

## Terminology
- "widget" → the customer's product being tracked, NOT a UI component. UI components are called "components".
- "event" → a usage event from the tracking SDK, stored in `public.events` table
- "org" → customer organization, maps to `public.organizations` table

## Do NOT
- Use `useEffect` for data fetching. Fetch in Server Components or use Supabase realtime subscriptions.
- Use class components. Functional components with hooks only.
- Put Supabase queries directly in components. All queries go through `lib/queries/`.
- Use `any` type. If the type is genuinely unknown, use `unknown` and narrow.

## References
- Before modifying auth: read `@docs/auth-flow.md`
- Before modifying database schema: read `@docs/schema.md`
- Before adding a new dashboard panel: read `@docs/panel-pattern.md`
```

---

## Example 2: Python FastAPI + PostgreSQL

```markdown
# OrderEngine — Order Processing API

REST API for processing and tracking customer orders.
Production service deployed via Docker to AWS ECS.

## Stack
- Python 3.12
- FastAPI 0.115
- SQLAlchemy 2.x (async, mapped classes)
- Alembic for migrations
- PostgreSQL 16
- pytest + httpx for testing
- Pydantic v2 for all request/response models

## Architecture
- `app/api/` — Route handlers grouped by domain (orders/, products/, customers/)
- `app/models/` — SQLAlchemy ORM models. One file per table.
- `app/schemas/` — Pydantic models. Separate Create/Update/Response schemas per entity.
- `app/services/` — Business logic. Handlers call services, services call repositories.
- `app/repositories/` — Database access. All SQL goes here. Never in services or handlers.
- `app/core/` — Config, security, database session management.
- `migrations/` — Alembic migration files. Auto-generated, then reviewed.

## Verification
Before completing any task, run ALL of these as a batch:
```bash
python -m pytest tests/ -x -q && \
python -m mypy app/ --strict && \
python -m ruff check app/ && \
python -m ruff format --check app/
```

## Terminology
- "order" → customer purchase order, `orders` table. NOT a sorting directive.
- "fulfillment" → the shipping/delivery process AFTER order confirmation
- "SKU" → stock keeping unit, unique product identifier in `products.sku` column

## Do NOT
- Use synchronous database operations. All DB calls use async sessions.
- Put business logic in route handlers. Handlers validate input → call service → return response.
- Use raw SQL strings. Use SQLAlchemy query builder or mapped class queries.
- Create circular imports between models. Use `TYPE_CHECKING` for type hints if needed.
- Run migrations without generating a review migration first: `alembic revision --autogenerate -m "description"`, then inspect before `alembic upgrade head`.

## References
- Before modifying the order state machine: read `@docs/order-states.md`
- Before adding a new API endpoint: read `@docs/endpoint-pattern.md`
- Before modifying database schema: read `@docs/migration-protocol.md`
```

---

## Example 3: Lightweight Script (Minimal CLAUDE.md)

```markdown
# csv-cleaner

CLI tool that reads messy CSV exports and outputs clean, standardized CSVs.
Personal utility, not production.

## Stack
- Python 3.12, no framework
- pandas for data manipulation
- click for CLI interface

## Verification
- `python -m pytest tests/ -q`
- `python cli.py --help` (must not error)

## Do NOT
- Add a web interface. This is CLI only.
- Use any database. Input and output are CSV files.
```

---

## Key Patterns to Notice

1. **Every example starts with a 1-2 sentence identity statement.** The agent needs to know
   what it's working on before anything else.

2. **Stack sections list exact versions.** "React" is not helpful. "React 19.1 with Next.js 15
   App Router" tells the agent exactly which APIs and patterns to use.

3. **Architecture sections describe YOUR decisions, not framework defaults.** Don't explain
   what App Router is — the model knows. Explain that YOU chose to put queries in `lib/queries/`
   instead of inline.

4. **Verification commands are copy-paste executable.** Never "run the tests" — always
   `pnpm test` or `python -m pytest tests/ -x -q`.

5. **Terminology maps prevent the single most common agent error:** misinterpreting a
   domain term as a technical term or vice versa.

6. **Anti-patterns are specific and reasoned.** Not "write good code" but "don't use useEffect
   for data fetching because we use Server Components."

7. **The lightweight example is 15 lines.** Resist the urge to over-document simple projects.
