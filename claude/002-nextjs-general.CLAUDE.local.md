# Next.js practices

Applies to TypeScript/JavaScript files under `src/`, `app/`, `pages/`, `components/`, `lib/`.

## Commands
- Install: `pnpm install`
- Dev server: `pnpm dev`
- Build: `pnpm build`
- Test: `pnpm test`
- Lint: `pnpm lint`
- Type-check: `pnpm typecheck`

## Conventions
- British spelling in code, comments, and identifiers (`colour`, `organise`, `behaviour`).
- TypeScript strict mode. No `any` — use `unknown` and narrow, or write the proper type. Shared types live in `types/`.
- Absolute imports via the `@/` alias. Group and sort imports. No circular deps.

## Architecture
- Default to **Server Components**. Add `"use client"` only for browser APIs, event handlers, or local state.
- Mutations go through **Server Actions** or **Route Handlers**. Validate input with **Zod** on the server.
- App Router conventions: `loading.tsx` for Suspense, `error.tsx` for error boundaries, `not-found.tsx` for 404.
- Co-locate by feature: `app/(feature)/components`, `app/(feature)/[route]`, `lib/(feature)/*`.
- Use `next/link` and `next/navigation`. Avoid `window.location` in client code.

## File size & structure
- Target < 350 lines per file. Extract reusable UI to `components/`, hooks and helpers to `lib/`.
- Barrel files (`index.ts`) only where they don't bloat the client bundle.

## Data fetching
- Server `fetch` with an explicit cache mode:
  - Static: `{ cache: 'force-cache' }`
  - Revalidating: `{ next: { revalidate: 60 } }`
  - Dynamic: `{ cache: 'no-store' }`
- After mutations: `revalidateTag` or `revalidatePath`.
- Client-side queries use **TanStack Query** or **SWR**. Don't duplicate the same request server- and client-side.

## Performance
- `next/image` with explicit width and height. `priority` only for above-the-fold images.
- `next/font` over `<link>` tags.
- `dynamic(() => import('...'), { ssr: false })` for heavy client-only modules.
- Memoise (`React.memo`, `useMemo`, `useCallback`) only when a profiler confirms re-renders are the bottleneck. Not preemptively.
- Debounce or throttle expensive handlers. Prefer CSS for animations; `requestAnimationFrame` for JS-driven loops.
- Run bundle analysis before merging anything that adds dependencies. Flag any single client chunk over 200KB gzipped.

## Styling
- Single `ThemeProvider`. Reference theme tokens, never hardcode colours.
- Extract repeated UI (forms, cards, dialogs) into typed components with small prop APIs.
- Support dark mode. Check contrast at AA minimum.

## Security
- Secrets stay on the server. `process.env.*` is only read in server code.
- Validate all external input with Zod.
- Sanitise rendered HTML (`rehype-sanitize` or equivalent).
- CSP and Referrer-Policy headers via middleware in production.

## Accessibility
- Semantic HTML first; ARIA only where semantics fall short.
- Every interactive element keyboard-reachable with a visible focus ring.
- Icon-only buttons need `aria-label`. Inputs need labels and clear error text.

## Testing
- Vitest or Jest for unit and critical logic. Playwright for key user flows.
- Test behaviour, not implementation details.

## Logging & errors
- `console` is for development only. Server-side: a structured logger (e.g. **pino**) with timestamps and levels.
- Log errors with context. Never log PII or secrets.
- Surface errors via `error.tsx` (server boundary) and toasts/snackbars (client actions).

## Documentation
- Short JSDoc on complex components and non-obvious props.
- A README in a feature folder when the layout isn't self-explanatory.

## When in doubt
Clarity first, performance second. Small, well-named components beat clever one-liners.
