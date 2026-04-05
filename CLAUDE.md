# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev       # Start dev server (Next.js)
npm run build     # Production build
npm run lint      # Run ESLint
npx prisma migrate dev   # Run DB migrations
npx prisma studio        # Open Prisma Studio
npx prisma generate      # Regenerate Prisma client
```

## Architecture

**Stack:** Next.js 16 (App Router) · React 19 · TypeScript · Tailwind CSS v4 · Prisma 7 (PostgreSQL via `pg` adapter) · Clerk (auth) · shadcn/ui components

### Auth flow (Clerk + org-based access)

- All routes are protected by Clerk middleware in [src/proxy.ts](src/proxy.ts), which is the Next.js middleware file (note: non-standard filename — the `matcher` config exports the middleware correctly).
- Unauthenticated users are redirected to `/sign-in` or `/sign-up` (Clerk hosted UI routes under `[[...sign-in]]` / `[[...sign-up]]`).
- Authenticated users **without an org** are redirected to `/org-selection`, which uses Clerk's `OrganizationList`. The app is org-scoped — every resource (`Voice`, `Generation`, `AudioJob`) carries an `orgId`.
- After org selection, users land on `/` (currently a placeholder with `OrganizationSwitcher` + `UserButton`).

### Database

- Prisma schema is at [prisma/schema.prisma](prisma/schema.prisma); generated client outputs to `src/generated/prisma/`.
- The singleton Prisma client lives in [src/lib/db.ts](src/lib/db.ts) and uses the `@prisma/adapter-pg` driver adapter (not the default binary engine).
- Core models: `Voice` (system or custom, org-scoped), `Generation` (TTS job output, links to a Voice), `AudioJob` (stub, org-scoped).

### Environment variables

Validated at startup via `@t3-oss/env-nextjs` in [src/lib/env.ts](src/lib/env.ts). Required:
- `DATABASE_URL` — PostgreSQL connection string
- Clerk keys (`NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY`, `CLERK_SECRET_KEY`) — referenced by Clerk SDK automatically

### UI

- Components live in `src/components/ui/` — these are shadcn/ui primitives (Base UI + Radix UI underneath).
- `cn()` utility from [src/lib/utils.ts](src/lib/utils.ts) combines `clsx` + `tailwind-merge`.
- Global styles and CSS variables are in [src/app/globals.css](src/app/globals.css).
- `sonner` is wired as a global `<Toaster>` in the root layout.
