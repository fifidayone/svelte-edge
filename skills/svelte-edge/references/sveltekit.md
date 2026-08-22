# SvelteKit Defaults, State Safety, Routing, Forms, and Stable Architecture

Use this file for SvelteKit architecture and behavior.
Remote functions are a separate opt-in concern — see `references/remote-functions.md`.
This file covers **SvelteKit 2** only. Once a project resolves `@sveltejs/kit@3.0.0-next.*`, or the user explicitly asks about SvelteKit 3 preview or a SvelteKit 2→3 migration, switch to `references/sveltekit-3-preview.md` — several APIs described below (config location, shallow routing, `invalidateAll`, env vars) change shape on that line. The SvelteKit 2→3 migration workflow lives in `references/migration.md`.

## Contents

- [Stable architecture and server safety](#stable-default-architecture)
- [Types, loading, and state](#types-and-props)
- [Forms and server routes](#form-actions)
- [Project configuration and environment variables](#project-configuration)
- [Navigation and UI state](#navigation-apis)
- [Errors and redirects](#error-and-redirect-helpers)

## Stable default architecture

For most SvelteKit apps, default to:
- `load` functions for route data
- form actions for form submissions and mutations tied to forms
- `+server` files for HTTP endpoints and server-only request handling
- `$app/state` for page/navigation state access in modern SvelteKit
- generated `./$types` for route-safe typing

Do not present remote functions as the baseline answer for all projects.

## Server-safety doctrine

### Avoid shared mutable state on the server
Do not store per-request or per-user mutable state in top-level server module variables.
A SvelteKit server process can handle many users, so shared mutable module state can leak data between requests.

Prefer:
- `event.locals`
- cookies
- request-scoped data
- DB/session lookups
- server-only helpers that read from request context

### No side effects in `load`
Treat `load` as data loading.
Do not do writes, background mutations, or stateful side effects in `load`.

If you need side effects or writes, use:
- form actions
- `+server` handlers
- explicit server functions where appropriate

### Global state vs context vs request scope

Use shared module state only for client-side state or data that is not specific to an individual user/request.
Use `createContext()` when state should be scoped to a subtree and passed through a component tree without prop drilling.
Use `event.locals`, cookies, DB/session lookups, or other request-scoped state for user-specific server data.

If state must not leak between users, do not put it in a shared server module.

## Types and props

Use generated route types. Note that SvelteKit types the `form` prop (returned from Form Actions) as `ActionData | null`. SvelteKit passes `null` on the initial GET request. When forwarding the `form` prop to child components that expect optional properties (`ActionData | undefined`), convert `null` to `undefined` explicitly (`form={form ?? undefined}`).

```svelte
<script lang="ts">
	import type { PageProps } from './$types';

	let { data, form }: PageProps = $props();
</script>
```

For route params in components, use `page.params` from `$app/state` when needed.
In **SvelteKit 2.24+**, pages also receive a generated `params` prop through `PageProps`; before that, do not invent a page `params` prop.

In newer SvelteKit, route params with matchers are narrowed more precisely in `$app/types`, `$app/state`, and hooks. Prefer the generated types instead of hand-rolled param unions.

## Loading data

Use `+page.server.ts` / `+layout.server.ts` when data needs secrets, private env vars, cookies, or server-only modules.
Use universal `+page.ts` / `+layout.ts` when the load can safely run on server and client.

Key load rules:
- `load` is for reads, not writes or side effects.
- Use `depends(...)` to declare custom invalidation keys and `$app/navigation` `invalidate(...)` / `invalidateAll()` to rerun load.
- A rerun updates `data` but does not recreate the component; local component state is preserved unless you reset it or use `{#key ...}`.
- Streaming promises are useful for slow non-critical data, but do not hide auth or required validation behind late streams.
- In `load`, prefer `url.searchParams.get('key')` over reading `url.search` or `url.href`. SvelteKit tracks individual `.get()` calls, preventing unnecessary reruns when unrelated query parameters change.
- When returning raw promises in server `load` for streaming, attach a `.catch(() => {})` to each. An unhandled rejection on a lazy promise before rendering starts will crash the Node process.

`invalidateAll()` is the SvelteKit 2 API for rerunning every active load function.

### Auth and parent data

Do not assume a layout `load` protects every child route.
Layout loads do not rerun on every client navigation, and layout/page loads run concurrently unless a child explicitly awaits `parent()`.

For protected data, prefer:
- hooks for broad route families before any load runs
- route-specific `+page.server.ts` guards for page-only protection
- explicit `await parent()` only when the child truly depends on parent data

Never put authorization-critical logic in a layout load while child loads can run protected work independently.

When using `await parent()`, initiate independent fetches first and await `parent()` only where subsequent logic depends on its result. Placing it at the top of a load function serializes all work into a waterfall.

### SSR fetch and cookies

During SSR, SvelteKit's `fetch` forwards cookies automatically for same-origin and more-specific-subdomain requests, but drops them for sibling subdomains (e.g. `app.example.com` → `api.example.com`) and parent domains — the server can't infer a cookie's original domain scope safely. If you forward manually in `handleFetch` (e.g. to a sibling API), scope it to a hardcoded allowlist of first-party origins you operate, forwarding only the specific cookie that API needs via `event.cookies.get(...)` — never the whole `Cookie` header to a request-derived, rewritten, or third-party URL.

## `$app/state`

Prefer `$app/state` in modern SvelteKit for route/page state instead of older patterns.
Do not introduce deprecated `$app/stores` in new Svelte 5 code. Keep it only for Svelte 4 compatibility or an untouched legacy file.

```svelte
<script lang="ts">
	import { page } from '$app/state';
</script>

<p>Current slug: {page.params.slug}</p>
```

## Form actions

Use form actions as the stable default for user-submitted mutations tied to forms.
This is usually the cleanest baseline for auth, profile updates, settings, and CRUD forms.

Use `fail(...)` for validation errors that should return form data and status.
Use `redirect(...)` for successful flows that should move the user.
`use:enhance` progressively enhances POST forms that target `+page.server` actions; do not use it for GET forms or arbitrary `+server` endpoints.
If an action sets or deletes cookies used by `handle` to populate `event.locals`, update `event.locals` inside the action too. `handle` does not rerun between the action and the following load.
Set cookie attributes explicitly: `httpOnly: true`, `secure: true` (HTTPS), `sameSite: 'lax'`, `path: '/'`. Form actions are not an auth mechanism — validate inputs and authenticate/authorize yourself; SvelteKit's default production origin check covers form-style CSRF, not blanket API CSRF.

When implementing a custom `onsubmit` handler without `use:enhance`, use `deserialize(await response.text())` from `$app/forms` instead of `response.json()`. SvelteKit serializes action data with `devalue`, which supports types that JSON does not (Dates, BigInts, cyclical references).

The default action runs on a plain `POST` to the page path with no query string. `?/` (empty action name) 404s, and `?/default` 500s because `default` is a reserved action name — do not use either when triggering the default action via raw HTTP or in tests.

## `+server` routes

Use `+server.ts` / `+server.js` for server-only handlers, APIs, webhooks, and custom HTTP behavior. `+server` routes get no automatic authentication or API CSRF protection — they are raw handlers, so validate input and authenticate/authorize in the handler (typically via `event.locals` populated in `hooks.server.js`).

To read an imported asset from the filesystem at runtime on the server (e.g. a bundled text or binary file), use `read(asset)` from `$app/server` (**SvelteKit 2.4.0+**). It returns a `Response`; this is a general server utility, not a remote-function feature.

## Security floors

Because this file teaches the APIs they protect (form actions, `+server`, content negotiation), the SvelteKit 2 security floors are surfaced here too; the authoritative inventory stays in `SKILL.md` → Critical Security Patch Floors.

- Cross-site request (CSRF) and remote-function origin checks: require **SvelteKit 2.70.0+** — builds with a non-production `NODE_ENV` previously compiled origin enforcement out (fix, sveltejs/kit#16313). `csrf.checkOrigin` defaults to `true`; it is deprecated in favor of `csrf.trustedOrigins`, but the check stays on by default.
- `Accept`-header content negotiation: require **SvelteKit 2.70.2+** (quadratic backtracking / ReDoS — CVE-2026-66062, GHSA-29g2-3rmr-qm68; affects ≤2.70.1).


## Project configuration

SvelteKit 2 still supports `svelte.config.js`. Since **2.62+**, configuration may instead live directly in the `sveltekit({...})` Vite plugin call, and `sv@0.16+` uses that layout for new projects.

```ts
// vite.config.ts
import adapter from '@sveltejs/adapter-auto';
import { sveltekit } from '@sveltejs/kit/vite';
import { defineConfig } from 'vite';

export default defineConfig({
	plugins: [sveltekit({ adapter: adapter() })]
});
```

In plugin configuration, SvelteKit options such as `adapter` sit alongside Svelte compiler options; there is no outer `kit` property. If configuration is passed to `sveltekit(...)`, any `svelte.config.js` is ignored rather than merged.

- New project on SvelteKit 2.62+ -> prefer the current `sv` scaffold layout in `vite.config`
- Existing project -> preserve its coherent config location unless migration is requested
- Never split options across both files
- Do not call `svelte.config.js` removed or invalid on SvelteKit 2 — that removal only happens on the SvelteKit 3 preview line; read `references/sveltekit-3-preview.md` for that line's config shape rather than guessing forward from this section

## Server-only modules and environment variables

Use `$lib/server` for server-only code that must never enter the client bundle.

### SvelteKit 2 default

Use `$env/static/private` / `$env/static/public` for build-time values and `$env/dynamic/private` / `$env/dynamic/public` for runtime values. Only use public env modules for values safe to expose to the browser; public means client-readable.

### Explicit environment variables (experimental)

SvelteKit **2.63+** offers an opt-in preview of the SvelteKit 3 model:

- enable `experimental.explicitEnvironmentVariables`
- declare variables in `src/env.ts` with `defineEnvVars` from `@sveltejs/kit/env` (canonical since **SvelteKit 2.70+**; the earlier `@sveltejs/kit/hooks` re-export still works but is deprecated — import from `@sveltejs/kit/env` in new code)
- import declared values from `$app/env/private` or `$app/env/public`
- use `$app/env` instead of `$app/environment`
- configure validation/transforms, `public`, `static`, and descriptions per variable

This is an edge-mode recommendation only. `$env/*` and `$app/environment` remain the SvelteKit 2 default, but official docs say they will be removed in SvelteKit 3. Explain that migration horizon without pretending the experimental replacement is already mandatory. For the actual SvelteKit 3 preview `defineEnvVars` shape, read `references/sveltekit-3-preview.md` — do not extrapolate its exact API from this SvelteKit 2 opt-in section.

Never import private env or `$lib/server` modules from client/shared code.

## Build and prerender additions

- With **SvelteKit 2.66+**, adapter precompression also covers prerendered `.md` and `.mdx` files.
- With **SvelteKit 2.67+**, use `prerender.handleInvalidUrl` to fail, warn, ignore, or custom-handle URLs the crawler cannot parse. Do not misuse `handleHttpError` for this case.
- **Prerender Crawler Rule:** Do not use relative hash links (e.g., `href="#work"`) inside shared layout headers/footers. During static crawling/prerendering, SvelteKit resolves relative links against the current path (e.g., `/projects/01#work` on page `/projects/01`). If the target ID does not exist on that subpage, the build will crash. Use root-relative links instead (e.g., `href="/#work"`).

## Navigation APIs

Use `$app/navigation` for programmatic routing and preloading when needed.
Relevant functions include:
- `goto(...)`
- `preloadData(...)`
- `preloadCode(...)`
- `invalidate(...)`

For external URLs, use normal browser navigation instead of `goto(...)`.

## Shallow routing

Use shallow routing when UI state should update browser history without full navigation.
This is especially useful for modal/detail flows.

Core APIs:
- `pushState(...)`
- `replaceState(...)`
- `page.state`

Use this when the user needs back-button-friendly UI state like in-page modals.

## Snapshots

Use snapshots to preserve ephemeral DOM state such as unsaved form input across navigation/back-forward flows.
Export `snapshot` from `+page.svelte` or `+layout.svelte` with `capture` and `restore`.

```svelte
<script lang="ts">
	let comment = $state('');

	export const snapshot = {
		capture: () => comment,
		restore: (value: string) => (comment = value)
	};
</script>

<textarea bind:value={comment}></textarea>
```

## Error and redirect helpers

Use `error(...)` and `redirect(...)` from `@sveltejs/kit` for server-side route control flow.
Do not fake these with ordinary thrown strings or custom response objects when the built-ins are the right fit.

### Route-level vs component-level errors

Use `+error.svelte` and `error(...)` for route-level/server request failures.
Use `<svelte:boundary>` for component-level async or rendering failures inside a subtree.

With **SvelteKit 2.54+** and **Svelte 5.53+**, `kit.experimental.handleRenderingErrors` can wrap route components in server rendering boundaries. This is experimental opt-in, not the default. Rendering errors go through `handleError` and the nearest `+error.svelte`; because rendering may already be in progress, read the passed `error` prop rather than expecting `page.error` to update.

Route-level errors remain the default answer for request/load failures. Do not enable rendering-error handling merely because a project lacks the flag.

## Remote functions

Remote functions are an opt-in SvelteKit feature for projects that explicitly enable `kit.experimental.remoteFunctions: true`.
They are not the stable default and are not covered here.

→ Read `references/remote-functions.md` when the user asks about remote functions, or the project contains `.remote.ts` / `.remote.js` files, or `kit.experimental.remoteFunctions` is enabled.

→ On the SvelteKit 3 preview line, also cross-check `references/sveltekit-3-preview.md` in addition to `remote-functions.md` once a project is on `3.0.0-next.*`.

## `getRequestEvent()`

Use `getRequestEvent()` (**SvelteKit 2.20+**) in server-side contexts — `+server.ts`, hooks, and server-only helpers — when it meaningfully simplifies access to the active request event.
It is request-scoped server logic, not a general client-side primitive.
For `getRequestEvent()` behavior inside remote functions specifically, see `references/remote-functions.md`.

## Link options

Remember that SvelteKit supports link options like preloading behavior on navigational links.
Use them when route transitions benefit from prefetching, but do not spam them everywhere by default.

## Hard reminders

- Stable first: `load`, actions, `+server`, `$app/state`
- New Svelte 5 project -> `$app/state`, never deprecated `$app/stores`
- Avoid shared mutable server state
- Keep `load` pure and side-effect-free
- Use shallow routing and snapshots when UI state/history behavior matters
- Remote functions are opt-in only — see `references/remote-functions.md`
- Keep stable `$env/*` defaults for SvelteKit 2; present explicit env vars only as an opt-in preview of the SvelteKit 3 model
- Keep all configuration in one place; Vite-plugin config wins and causes `svelte.config.js` to be ignored
- Project resolving `3.0.0-next.*`, or an explicit SvelteKit 2→3 migration ask -> stop applying this file's SvelteKit-2-specific API shapes and read `references/sveltekit-3-preview.md` (migration workflow: `references/migration.md`)
- Always map `form` props using `form ?? undefined` to prevent strict type check mismatches with child component optional props
- Never use relative anchor links (`href="#section"`) in global layouts; use root-relative links (`href="/#section"`) to prevent prerender crawler failures on subpages
