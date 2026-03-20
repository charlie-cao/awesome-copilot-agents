---
applyTo: "**/*.{ts,tsx}"
description: "GitHub Copilot instructions for Next.js 14+ App Router projects with TypeScript, Tailwind CSS, and modern React patterns."
---

# Next.js App Router (TypeScript)

## Project Convention

- Use Next.js **App Router** exclusively (`src/app/` directory). Do not use the Pages Router.
- All components default to **React Server Components** (RSC). Add `"use client"` only when the component requires browser APIs, event handlers, or React hooks.
- Collocate related files: keep `page.tsx`, `layout.tsx`, `loading.tsx`, `error.tsx`, and `not-found.tsx` together inside their route segment folder.
- Use `src/` directory layout: `src/app/`, `src/components/`, `src/lib/`, `src/hooks/`, `src/types/`.

## TypeScript

- Enable strict mode (`"strict": true` in tsconfig). Never use `any` — prefer `unknown` with type narrowing.
- Export a named `type` or `interface` for every component's props. Prefer `interface` for object shapes; use `type` for unions and primitives.
- Use `satisfies` operator when inferring types from object literals to preserve narrower types.
- Avoid non-null assertions (`!`). Use optional chaining (`?.`) and nullish coalescing (`??`) instead.

## Components

- One component per file. Filename matches the exported component name in PascalCase.
- Prefer composition over prop drilling. Extract reusable UI into `src/components/ui/`.
- Use `children: React.ReactNode` (not `JSX.Element`) as the prop type for wrapper components.
- Avoid `React.FC` — write plain function declarations: `export function MyComponent({ prop }: Props)`.
- For Client Components that accept Server-rendered children, keep the parent a Server Component and pass children via slots.

## Data Fetching

- Fetch data in **Server Components** using `async`/`await` directly — not `useEffect`.
- Cache control: use `fetch` with `{ cache: "force-cache" }` for static, `{ next: { revalidate: N } }` for ISR, and `{ cache: "no-store" }` for dynamic data.
- For mutations, use **Server Actions** (`"use server"` directive) — not API routes when possible.
- Use `React.cache()` to deduplicate expensive reads shared across a request.

## Routing & Navigation

- Use `useRouter` from `next/navigation` (not `next/router`).
- Prefer `<Link>` from `next/link` for all internal navigation.
- Dynamic segments: `[slug]`, catch-all: `[...slug]`, optional catch-all: `[[...slug]]`.
- Use route groups `(group-name)/` to share layouts without affecting URL structure.

## Styling

- Use **Tailwind CSS** utility classes as the primary styling approach.
- Global styles only in `src/app/globals.css`. Avoid scoped CSS modules unless isolated animation or complex selectors are required.
- Responsive design: mobile-first — `sm:`, `md:`, `lg:`, `xl:` breakpoints.
- Use `cn()` utility (clsx + tailwind-merge) for conditional class merging.

## Performance

- Use `next/image` (`<Image>`) for all images — never raw `<img>` tags.
- Use `next/font` to load fonts — never `@import` in CSS.
- Code-split large client bundles with `dynamic(() => import(...), { ssr: false })`.
- Avoid large dependencies in the critical path. Check bundle size with `@next/bundle-analyzer`.

## Error Handling

- Every route segment that fetches data must have a sibling `error.tsx` (must be a Client Component).
- Use `notFound()` from `next/navigation` for 404 responses in server components.
- Wrap third-party calls in `try/catch`; return typed `{ data, error }` objects from service functions.

## Environment Variables

- Server-only secrets: plain `NEXT_PUBLIC_` prefix is **never** used — keep secrets server-side only.
- Client-accessible values: prefix with `NEXT_PUBLIC_`.
- Access via `process.env.VAR_NAME` — do not destructure `process.env`.
- Validate all required env vars at startup with zod or a custom validator in `src/lib/env.ts`.

## Forbidden Patterns

- Do not use `getServerSideProps`, `getStaticProps`, or `getStaticPaths` — these are Pages Router APIs.
- Do not import Server Components into Client Components.
- Do not use `useEffect` for data fetching — use Server Components or React Query / SWR on the client.
- Do not use `<a>` for internal navigation — always use `<Link>`.
- Do not put business logic in `page.tsx` — delegate to service functions in `src/lib/` or `src/services/`.
