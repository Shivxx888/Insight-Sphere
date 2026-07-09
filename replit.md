# InsightSphere

A premium full-stack blogging website — "Learn Something New Every Day." Covers AI, technology, finance, make money online, business, education, digital marketing, cybersecurity, mobile apps, and how-to guides.

## Run & Operate

- `pnpm --filter @workspace/api-server run dev` — run the API server (port 8080, proxied at `/api`)
- `pnpm --filter @workspace/insight-sphere run dev` — run the frontend (port 23030, proxied at `/`)
- `pnpm run typecheck` — full typecheck across all packages
- `pnpm run build` — typecheck + build all packages
- `pnpm --filter @workspace/api-spec run codegen` — regenerate API hooks and Zod schemas from the OpenAPI spec
- `pnpm --filter @workspace/db run push` — push DB schema changes (dev only)
- `pnpm --filter @workspace/scripts run seed` — seed sample data into the database
- Required env: `DATABASE_URL` — Postgres connection string, `SESSION_SECRET` — session secret

## Stack

- pnpm workspaces, Node.js 24, TypeScript 5.9
- Frontend: React + Vite, Tailwind CSS v4, shadcn/ui components, wouter routing, framer-motion, recharts
- API: Express 5, Pino logging
- DB: PostgreSQL + Drizzle ORM
- Validation: Zod (`zod/v4`), `drizzle-zod`
- API codegen: Orval (from OpenAPI spec → React Query hooks + Zod schemas)
- Build: esbuild (CJS bundle)

## Where things live

- `lib/db/src/schema/` — DB schema (categories, authors, posts, comments, newsletter, contacts)
- `lib/db/src/index.ts` — DB connection + all schema re-exports
- `lib/api-spec/` — OpenAPI 3.1 YAML spec (source of truth for API contracts)
- `lib/api-zod/src/generated/` — generated Zod schemas from the spec
- `lib/api-client-react/src/generated/` — generated React Query hooks from the spec
- `artifacts/api-server/src/routes/` — Express route handlers
- `artifacts/insight-sphere/src/pages/` — All frontend page components
- `artifacts/insight-sphere/src/components/` — Shared UI components (Layout, PostCard)
- `artifacts/insight-sphere/src/lib/theme.tsx` — Dark/light mode ThemeProvider + useTheme hook
- `scripts/src/seed.ts` — Database seed script

## Architecture decisions

- **Contract-first API:** OpenAPI spec in `lib/api-spec` is the source of truth; Zod schemas and React Query hooks are generated from it via `orval`.
- **Monorepo libs:** Shared code lives in `lib/*` packages (db, api-spec, api-zod, api-client-react). Leaf artifacts never import from each other.
- **Admin auth:** Simple token-based auth — token `insight-sphere-admin-token-secure-2024` stored in localStorage, passed via `Authorization: Bearer` header using `setAuthTokenGetter` from the API client.
- **No React Router:** Using `wouter` for lightweight client-side routing.
- **Theme:** ThemeProvider adds `.dark` class to `<html>`, CSS variables handle all color switching. ThemeContext is consumed via `useTheme()`.

## Product

**Public Blog:**
- Home page with hero featured posts, latest articles, trending sidebar, categories, authors, newsletter subscribe, recent comments
- `/blog` — paginated article listing with category filter
- `/blog/:slug` — full article with reading progress bar, table of contents, comments, like/bookmark, share buttons
- `/categories` — category grid with icons
- `/categories/:slug` — category article listing
- `/about` — team profiles
- `/contact` — contact form
- `/privacy`, `/terms`, `/disclaimer`, `/cookie-policy` — legal pages

**Admin Dashboard** (`/admin`):
- `/admin/login` — Email: shivxx888@gmail.com, Password: Admin@InsightSphere2024
- `/admin` — Dashboard with charts (top posts bar chart, category breakdown pie chart), stats, recent activity
- `/admin/posts` — Post listing with edit/delete/view actions
- `/admin/posts/new` — Rich post editor (title, slug, excerpt, content, SEO, tags, featured flag)
- `/admin/posts/:slug/edit` — Edit existing post
- `/admin/categories` — Category management with color picker
- `/admin/newsletter` — Newsletter subscriber list

## User preferences

_Populate as you build — explicit user instructions worth remembering across sessions._

## Gotchas

- Featured posts must have `featured: true` AND `status: "published"` to appear in the hero
- The `/posts/featured`, `/posts/trending`, `/posts/latest` routes must come BEFORE `/posts/:slug` in Express (they do — already correct)
- Admin auth uses `setAuthTokenGetter` in `main.tsx` — reads from `localStorage.getItem("admin_token")`
- CSS: Google Fonts `@import url()` must be first line in `index.css` (before `@import "tailwindcss"`)
- After seeding, run the postCount update script to sync category/author post counts

## Pointers

- See the `pnpm-workspace` skill for workspace structure, TypeScript setup, and package details
- Seed: `pnpm --filter @workspace/scripts run seed`
- Codegen: `pnpm --filter @workspace/api-spec run codegen`
