# Practical Next.js coding rules for performance, readability, and maintainability

Applies to:
- `src/**/*.{ts,tsx,js,jsx}`
- `app/**/*.{ts,tsx,js,jsx}`
- `pages/**/*.{ts,tsx,js,jsx}`
- `components/**/*.{ts,tsx,js,jsx}`
- `lib/**/*.{ts,tsx,js,jsx}`

0. British Spelling

# Structure & size
1. Keep each file under **350 lines**; extract UI and hooks to reusable components in `components/` and `lib/`.
2. Prefer **TypeScript** (`.ts/.tsx`) and strict mode; add types instead of `any`. Put shared types in `types/`.
3. Co-locate by feature: `app/(feature)/components`, `app/(feature)/[route]`, `lib/(feature)/*`.
4. Use **barrel files** (`index.ts`) only when they don't bloat the client bundle.

# React / Next.js best practices
5. Default to **Server Components**; add `"use client"` only for browser APIs/event handlers/local state.
6. For Client Components, minimise re-renders: lift state, split components, and memoise **sparingly** (`React.memo`, `useMemo`, `useCallback`) on hot paths.
7. Use **Server Actions** or **Route Handlers** for mutations. Validate with **Zod** on the server.
8. Follow App Router conventions: `loading.tsx` (Suspense), `error.tsx` (error boundary), `not-found.tsx` (404).
9. Use `next/link` + `next/navigation`; avoid `window.location` in client code.

# Data fetching & caching
10. Prefer server `fetch` with explicit caching:
    - Static: `fetch(url, { cache: 'force-cache' })`
    - Revalidate: `fetch(url, { next: { revalidate: 60 } })`
    - Dynamic: `fetch(url, { cache: 'no-store' })`
11. Invalidate with `revalidateTag` / `revalidatePath` after mutations.
12. For client queries, use **SWR** or **TanStack Query**; don't duplicate server + client fetch for the same data.

# Performance
13. Use **`next/image`** with width/height; set `priority` only for above-the-fold images.
14. Use **`next/font`** over custom `<link>` for fonts.
15. **Dynamic import** heavy client-only modules: `dynamic(() => import('...'), { ssr: false })` when needed.
16. Keep bundles lean: tree-shake, prefer native APIs, avoid large utility libs. Check bundle analysis.
17. Debounce/throttle expensive handlers; prefer CSS for animations; use `requestAnimationFrame` for smooth loops.

# Styling & theming (MUI-friendly)
18. Keep a single **ThemeProvider**; don't hardcode colours—use theme tokens.
19. Extract repeated UI into **reusable components** (forms/cards/dialogs). Keep prop APIs small and typed.
20. Support **dark mode** and check contrast.

# API & security
21. Never expose secrets to the client. Read `process.env.*` only on the server.
22. Validate inputs with **Zod**; sanitise HTML before rendering (e.g., `rehype-sanitize`).
23. Add security headers (CSP, Referrer-Policy) via middleware in production.

# Accessibility & UX
24. Use semantic HTML; ARIA only when needed. All interactive elements must be keyboard-reachable with visible focus.
25. Icon-only buttons need `aria-label`; inputs need labels and clear error text.

# Testing & quality
26. Test critical logic (Vitest/Jest) and key flows (Playwright). Test behaviour, not implementation.
27. Enforce ESLint + Prettier. Avoid `eslint-disable`—if needed, justify with a comment.
28. Import hygiene: absolute imports via `@/` alias; group/sort imports; avoid circular deps.

# Logging & errors
29. Use `console` for dev only. On server, use a tiny logger (timestamps/levels). Log errors with context, never PII.
30. Surface errors via `error.tsx` and user toasts/snackbars for client actions.

# Documentation
31. Add short JSDoc for complex components and props. Add a README in feature folders when non-obvious.

# When in doubt
32. Optimise for **clarity first**, then performance. Small, well-named components beat clever one-liners.
