# Testomniac UI

> **Git policy — never auto-commit or auto-push.** Leave your work in the working tree.
> Run `git commit`, `git push`, `gh pr create`, or `scripts/push_all.sh` **only when the user
> explicitly asks in that turn**. Approval for an earlier change does not carry forward, and
> finishing a task is not permission to commit it.

Shared dashboard UI library for Testomniac apps.

**Package**: `@sudobility/testomniac_ui` (restricted, BUSL-1.1)

## What this is

The environment-scoped dashboard pages + components extracted from
`testomniac_app` so they can be reused by `testomniac_extension` (and any other
host). It contains the **presentation layer only** — no routing, no i18n, no app
constants.

## Architecture: host-agnostic via two contexts

The library NEVER imports `react-router-dom`, `@sudobility/building_blocks`, or
`@/config/constants`. Anything host-specific is injected:

- **`src/context/config.tsx`** — `TestomniacUiProvider` carries `{ networkClient, token, apiUrl, SeoHead? }`.
  - `useTestomniacApi()` → `{ networkClient, token, baseUrl }` (replaces the app's `useApi()` + `CONSTANTS.API_URL`).
  - `<SEOHead/>` delegates to the host `SeoHead`, or renders nothing (extension).
- **`src/context/routing.tsx`** — `RoutingProvider` carries `{ params, pathname, navigate, currentLanguage, switchLanguage? }`.
  - `useRouteParams<T>()` replaces react-router `useParams`.
  - `<Redirect to=… />` replaces react-router `<Navigate>`.
  - `src/hooks/useLocalizedNavigate.ts` and `useEnvBasePath.ts` read this context.

Routing (route definitions + the real `navigate`) lives in the **host app**, not
here — the host wires its router into `RoutingProvider` per route.

## Structure

```
src/
├── index.ts          # barrel: providers, hooks, components, pages
├── context/          # config + routing contexts (host integration)
├── hooks/            # useLocalizedNavigate, useEnvBasePath, useDashboardEnvironmentContext
├── components/       # cells, data, scanner, states, forms, scripts, scenarios,
│                     #   bundles, dashboard (sidebar), pages (views), navigation,
│                     #   apiKeys, credentials, scan-settings, ui
├── config/           # priorityConfig
├── utils/            # formatDateTime
└── pages/            # 33 dashboard page modules (named exports)
```

## Commands

```bash
bun install        # scoped @sudobility deps resolve from GitHub Packages (see .npmrc)
bun run build      # tsc -p tsconfig.build.json -> dist/ (ESM + .d.ts)
bun run typecheck  # tsc --noEmit
bun run lint       # eslint src   (lint:fix to autofix)
bun run format     # prettier --write (format:check to verify)
bun run test:unit  # vitest run --passWithNoTests  (test:watch for watch mode)
bun run verify     # typecheck + lint + format:check + test:unit + build (full gate)
```

There is **no** bare `bun run test` script — use `test:unit` / `test:watch`.

## Conventions

- **Pages are named exports** (`export function FooPage()`), consumed via the barrel.
- Pages get data through `@sudobility/testomniac_client` / `_lib` hooks, passing
  `{ networkClient, token, baseUrl }` from `useTestomniacApi()`.
- Use `useRouteParams`, `useLocalizedNavigate`, `useEnvBasePath`, `<SEOHead/>`,
  `<Redirect/>` — never `react-router-dom` directly.
- Styling is Tailwind class names from `@sudobility/components` / `@sudobility/design`;
  the library emits no CSS.

## Gotchas

- **Registry**: `@sudobility/*` deps (incl. this package) live on GitHub Packages, not
  npmjs — `.npmrc` maps the scope to `npm.pkg.github.com`. Install/publish needs a GitHub
  token with `read:packages` (CI publishes via `publish-target: github`).
- **No bundler / no CSS output**: build is `tsc` only. The library ships class names; the
  host's Tailwind must include `node_modules/@sudobility/testomniac_ui` in its `content` globs.
- **Barrel is manual**: every public component/page/hook must be re-exported from
  `src/index.ts` (with `.js` extensions) or it won't ship.
- Peer deps are pinned-ish ranges; `@sudobility/*` versions move fast — keep `package.json`
  peer + dev versions in sync.

## Consumers

- **testomniac_app** — wraps pages with a react-router `UiRoute` adapter (see its `src/lib/testomniacUi.tsx`).
- **testomniac_extension** — (planned) state-based routing adapter, no `SeoHead`.

## Related projects

- **testomniac_client** — API client + query hooks
- **testomniac_lib** — business logic (managers, analysis, formatting, priority, graph layout)
- **testomniac_types** — shared types

## Git Workflow

- Do not use feature branches for code changes. Always stay on the current branch.
