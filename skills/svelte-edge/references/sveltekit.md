# SvelteKit 3

**Status:** current generation — stable since **2026-10-01** (`@sveltejs/kit@3.0.0`, official announcement "SvelteKit 3 is here"). This is the canonical SvelteKit reference: default to it for new projects and for any project resolving `@sveltejs/kit@3.*`. SvelteKit 2 guidance lives in `references/sveltekit-legacy.md`; the 2→3 workflow lives in `references/migration.md`.

Facts verified against the published `@sveltejs/kit@3.0.0` package (exports map and type declarations) and current official docs. Verification method: `references/maintenance.md`.

## Contents

- [Requirements](#requirements)
- [Configuration](#configuration)
- [Project layout](#project-layout)
- [Environment variables](#environment-variables)
- [Navigation and state](#navigation-and-state)
- [Reloading data](#reloading-data)
- [Snapshots](#snapshots)
- [Forms and errors](#forms-and-errors)
- [Remote functions (experimental)](#remote-functions-experimental)
- [Service workers and manifest](#service-workers-and-manifest)
- [Server and runtime behavior](#server-and-runtime-behavior)
- [Generation fences](#generation-fences)
- [Sync and check noise](#sync-and-check-noise)

## Requirements

- Node **22.17+**, TypeScript **6+**, Svelte **^5.57.1**, Vite **^8.0.12** (Rolldown 1.0), `@sveltejs/vite-plugin-svelte` **7+**
- a kit-3-line adapter major: `adapter-auto` 8, `adapter-node` 6, `adapter-static` 4, `adapter-vercel` 7, `adapter-cloudflare` 8, `adapter-netlify` 7 — these peer on `@sveltejs/kit@^3.0.0-next.0`, so SvelteKit 2 projects must stay on the older adapter majors
- `@opentelemetry/api ^1.0.0` as an optional peer, consumed by `kit.tracing`
- On Windows, require Vite **8.0.16+**: the `^8.0.12` peer floor allows 8.0.12–8.0.15, which are vulnerable to CVE-2026-53571 (`server.fs.deny` bypass via NTFS ADS / 8.3-name forms)
- 3.0.0 inherits every current SvelteKit security fix — the per-fix version floors in `SKILL.md` (origin checks, Accept-header ReDoS) apply to the SvelteKit 2 line
- Upgrade framework, adapter, and toolchain in one branch. After upgrading: `svelte-kit sync`, `npx sv check`, unit/component tests, E2E, production build, deployment smoke test

## Configuration

All SvelteKit and Svelte configuration lives in the `sveltekit({...})` Vite plugin call — `svelte.config.js` is no longer supported. If configuration is passed to `sveltekit(...)`, any leftover `svelte.config.js` is ignored rather than merged; never split options across both.

```ts
// vite.config.ts
import { sveltekit } from '@sveltejs/kit/vite';
import { defineConfig } from 'vite';

export default defineConfig({
	plugins: [sveltekit({
		csrf: { trustedOrigins: ['https://checkout.stripe.com'] },
		paths: { origin: 'https://example.com' },
		tracing: { server: true },
		output: { linkHeaderPreload: true }
	})]
});
```

- `paths.origin` sets the deployment origin used for prerendering and origin checks (replaces the adapter-node `ORIGIN` env var).
- `csrf.trustedOrigins`: explicit allowlist of full origins (protocol + host) permitted to submit cross-origin forms. CSRF checks run only in production. Remote functions strictly require same-origin and ignore `trustedOrigins` — remote endpoints are internal implementation details.
- `tracing` is a top-level option (OpenTelemetry span emission; requires `@opentelemetry/api`).
- `router.resolution: 'server'` is incompatible with `router.type: 'hash'` or `output.bundleStrategy: 'inline' | 'single'`.
- `csp.directives`: setting `'require-trusted-types-for': ['script']` requires `'svelte-trusted-html'` in `'trusted-types'` (and `'sveltekit-trusted-url'` if `serviceWorker.register` is true) — but only when client-side code is actually shipped: builds where all pages have `csr: false` don't need the policy.
- A universal `config` export wins over a server-only `config` export on the same route.

TypeScript config extends the generated `$app/tsconfig` (written to `node_modules/$app/tsconfig.json`); the project owns its `include` array, and `compilerOptions.types` must include `$app/types` when set.

## Project layout

### Subpath imports (`#lib`)

The `$lib` alias and `kit.files.lib` are gone. Declare `#lib` in `package.json` using Node's built-in subpath imports:

```json
{
	"imports": {
		"#lib": "./src/lib/index.js",
		"#lib/*": "./src/lib/*"
	}
}
```

Import as `import Component from '#lib/Component.svelte'` — specifiers must use a Node-style `.js` extension even when the target is a `.ts` file (`#lib/data.js`). TypeScript maps `.js` specifiers to `.ts` sources but does not probe extensions on `imports`-map results: under the generated `moduleResolution: "bundler"`, an extensionless import fails `svelte-check` with `Cannot find module '#lib/...'`. SvelteKit writes no `#lib` entry into the generated `paths` — the `package.json` `imports` map is the only mechanism. Keep the barrel target (`./src/lib/index.js`) existing, or drop that entry if the project has no barrel.

### Param matchers (`src/params.ts`)

All param matchers live in a single `src/params.ts` (or `.js`) file using the `defineParams` helper:

```ts
// src/params.ts
import { defineParams } from '@sveltejs/kit/params';
import * as v from 'valibot';

export const params = defineParams({
	// Standard Schema variant (e.g. Valibot, Zod)
	integer: v.pipe(v.string(), v.toNumber()),
	// Function variant — return the param to accept, undefined to reject
	fruit: (param: string) => (param === 'apple' || param === 'orange' ? param : undefined)
});
```

When a matcher returns a parsed value or uses a Standard Schema, route `params` are typed with the parsed output type. Callable Standard Schemas are not treated as function param matchers.

### Server-only directories

A `server/` path segment makes a directory server-only anywhere inside the project (except `src/routes` and `$lib`). `$lib/server` remains the standard home for server-only code; never import it (or `$app/env/private`) from client or shared code.

## Environment variables

Declare variables in `src/env.ts` with `defineEnvVars` from `@sveltejs/kit/env`; import them from `$app/env/private` or `$app/env/public` (never both server/client-crossing). Use `$app/env` in place of the old `$app/environment`.

```ts
// src/env.ts
import { defineEnvVars } from '@sveltejs/kit/env';
import { building } from '$app/env';
import * as v from 'valibot';

export const variables = defineEnvVars({
	// Secret, evaluated at startup, required
	POSTGRES_URL: {},

	// Safe for browser exposure
	PUBLIC_KEY: { public: true, schema: v.string() },

	// Inlined at build time for dead-code elimination
	BUILD_FLAG: { public: true, static: true, schema: v.boolean() },

	// Optional during build, required at runtime
	SECRET: { schema: building ? v.optional(v.string()) : v.string() },

	// Function validators are supported in addition to Standard Schemas
	TOKEN: { schema: (v) => (typeof v === 'string' && v.length >= 32 ? v : undefined) }
});
```

Variables declared `static` must be present at build time — the build fails with `env_invalid` naming the missing variable (harness-verified); non-static secrets are validated at startup instead. Variables that may legitimately be absent during build need the optional-`building` validator shape shown above. Never import from `$env/*` in new code: it exists only as deprecated compatibility aliases for dependencies that still import it.

## Navigation and state

```ts
import { goto } from '$app/navigation';

// open a modal without a full navigation, keep history entry
goto('/photos/42', { shallow: true, state: { showModal: true } });

// same, but restore state after a reload
goto('/photos/42', { shallow: true, state: { showModal: true }, persistState: true });

// replace history entry instead of pushing a new one
goto('/photos/42', { shallow: true, state: { showModal: true }, replace: true });
```

- Default preserves scroll/focus; `reset: true` intentionally resets them. State survives reload only with `persistState: true`.
- `goto` rejects destinations that don't resolve to an internal route — use `window.location.href` for external navigation.
- `page.state` is strictly typed: declare each key on `App.PageState` in `app.d.ts`.
- `page.url` and its search parameters are readonly — copy with `new URL(page.url)` before mutating.
- Navigation `delta` exists only for `popstate` navigations — guard it before use.
- Link attributes disable with `false`, not `"off"`: `data-sveltekit-*="false"`. `data:` protocol URLs count as external.
- Re-navigating to the current URL re-runs all `load` functions and queries.

## Reloading data

```ts
import { refreshAll, invalidate, preloadCode } from '$app/navigation';

await invalidate('app:posts');                          // rerun only load/queries depending on key
await refreshAll();                                      // rerun every active load function, query, and remote function
await refreshAll({ includeLoadFunctions: false });       // refresh remote functions only, skip load reruns

await preloadCode('/blog/[slug]');                       // takes a Route ID, without paths.base
```

`refreshAll()` does not reset `page.state`; deprecated `invalidateAll()` does. `preloadData(...)` can resolve to `{ type: 'error', status, error }` — handle the error result. In load functions, reruns also fire when the number of values of a tracked search parameter changes, not only when a value differs.

## Snapshots

Use the `snapshot()` helper from `$app/navigation`:

```svelte
<script lang="ts">
	import { snapshot } from '$app/navigation';

	let comment = $state('');

	snapshot({
		capture: () => comment,
		restore: (value: string) => (comment = value)
	});
</script>

<textarea bind:value={comment}></textarea>
```

`snapshot({ id?, capture, restore, reset? })` must run during component initialization and stays active while the component is mounted. Register several per component — unique per call site or via explicit `id` — and the optional `reset` callback runs on navigations with no captured value. The page-level `export const snapshot` is deprecated.

## Forms and errors

```ts
import { error, redirect } from '@sveltejs/kit';

// error() requires a string message as 2nd argument; details require an App.Error extension
error(404, 'Post not found', { code: 'POST_NOT_FOUND' });

// External redirects MUST specify { external: true }
throw redirect(307, 'https://checkout.stripe.com', { external: true });
```

- custom keys in the `details` object require extending `App.Error` in `app.d.ts` — without the extension the third parameter type collapses to `never` and `svelte-check` rejects the call.
- `handleError` can return `{ status, message }` to override the response status code; `App.Error` always includes `status: number`.
- `redirect(...)` to an external URL throws unless `{ external: true }` or the origin is listed in `csrf.trustedOrigins`.
- Cross-origin form submissions without a `Content-Type` header are rejected.
- Form action `fail(...)` status codes surface directly as the HTTP response status. The page `form` prop's `error` is typed `App.Error | undefined`.
- `invalid(...issues)` throws a `ValidationError` (accepts strings or `StandardSchemaV1.Issue` objects). `isValidationError` and `ValidationError` are imported from the root `@sveltejs/kit` package.
- Enhanced cross-page form actions navigate to the action page on success **and** failure, matching native form behavior; enhanced form results cannot navigate to another origin unless they are redirects.
- Every error runs through the `handleError` hook; stack traces for internal errors like 404s are hidden. Rendering errors are handled unconditionally: route components are wrapped in server rendering boundaries, the nearest `+error.svelte` renders at the depth it occupies (typed via generated `ErrorProps`), and a failed `<svelte:boundary>` resets on client navigation.
- `RequestEvent` and `Cookies` are imported from the root `@sveltejs/kit` package.

## Remote functions (experimental)

Remote functions are **experimental on the stable line**: they require `kit.experimental.remoteFunctions: true` plus `compilerOptions.experimental.async: true`, and `.remote.ts`/`.remote.js` files without the flag are a build error. Stabilizing them is the top post-3.0 priority per the release announcement. See `references/remote-functions.md` for the shared semantics; the SvelteKit 3 deltas:

- Every remote form field must be created with `form.fields.foo.as(...)` — hand-built field objects fail validation. `.as('radio', value, checked)` and `.as('checkbox', value, checked)` accept the checked state as a third argument so these inputs reset to it after submission.
- Inside a `query`, `event.url`/`event.params`/`event.route` are inaccessible.
- `.run()` does not exist — use `await` or async iteration.
- `handleValidationError` does not exist — remote validation errors reach `handleError` with `kind: 'validation'`.
- Remote function types (`query`, `form`, `command`, `prerender`, `requested`, `RemoteQuery`, `RemoteForm`, …) are exported from `$app/server`; `isValidationError` comes from the root `@sveltejs/kit` package.
- Remote form `validate({ all })` validates untouched fields too; `field.touched()` reports per-field dirtiness.
- A client-requested single-flight mutation errors when the server does not accept it via `requested(...)`; the server can explicitly ignore refreshes.
- `query.live` streams carry periodic SSE keep-alive comments and an explicit `Accept` header so proxies do not buffer them; stream cancellation is observable via the generator's `request.signal`.

## Service workers and manifest

```ts
// src/service-worker/index.ts
import { self } from '$app/service-worker';
import { version } from '$app/env';
import { assets, immutable, prerendered, routes } from '$app/manifest';
```

- `$app/service-worker` exports `self` (typed `ServiceWorkerGlobalScope`) — nothing else. Asset metadata comes from `$app/manifest`: `assets`, `immutable`, `prerendered`, and `routes` (route entries carry `page` and `endpoint` booleans; there is no `build`/`files` pair — account for the new shapes rather than renaming old imports).
- `version` comes from `$app/env` (alongside `browser`, `building`, `dev`).
- Service workers register as modules (`type: 'module'`) — use module imports, not `importScripts(...)`. `$app/paths` is importable inside service workers.
- A TypeScript service worker is its own TS project: `src/service-worker/tsconfig.json` extending `$app/tsconfig/service-worker`, excluded from the root tsconfig; move a flat `src/service-worker.ts` to `src/service-worker/index.ts`.

## Server and runtime behavior

- Query parameters beginning with `x-sveltekit-` are rejected — renamespace custom params that collide with this reserved prefix.
- `json(...)` and `text(...)` from `@sveltejs/kit` are deprecated — use `Response.json(...)` and `new Response(...)`.
- `+server.js` supports the `QUERY` HTTP method.
- `getRequest()` and `setResponse()` from `@sveltejs/kit/node` are synchronous.
- `cookies.parse(...)` parses raw cookie headers; cookie options are optional (cookie v2 defaults apply, names ASCII-only, default `path: '/'`).
- Production sourcemaps are supported.
- Dev-server CORS for static assets is delegated to Vite — configure `server.cors.origin` in `vite.config` for cross-origin dev access.
- Deployment-change detection polls hourly by default (`version.pollInterval: 3600000`) and also triggers on data/remote/form-action responses, tab focus, and visibility change.

## Generation fences

Never generate SvelteKit 2-only APIs in SvelteKit 3 code: `invalidateAll`, `pushState`, `replaceState`, `$service-worker`, `$lib`, `kit.csrf.checkOrigin`, `kit.prerender.origin`, `adapter-node` `ORIGIN`, `noScroll`, `keepFocus`, `data-sveltekit-*-off`, `alias` config option, `$env/*`, `$app/environment`, `@sveltejs/kit/node/polyfills`, `preloadStrategy`, `handleRenderingErrors`, `kit.experimental.explicitEnvironmentVariables`, `base`/`assets`/`resolveRoute` from `$app/paths`, page-level `export const snapshot`, `handleValidationError`, or root-package imports of `Page`, the `Navigation*` types, and `ActionResult`/`SubmitFunction`.

Never generate SvelteKit 3-only APIs in SvelteKit 2 code: `refreshAll`, `goto({ shallow, persistState, reset })`, `$app/manifest`, `$app/service-worker`, `#lib`, `paths.origin`, `$app/env/*`, `Path`/`AssetPath` (no leading slash), `resolve()`, `asset()`, `defineParams`, `snapshot()` from `$app/navigation`, the `@sveltejs/kit/params`/`@sveltejs/kit/hooks`/`@sveltejs/kit/env` subpaths, `$app/forms`/`$app/state` type imports, or universal-over-server `config` precedence.

`Path`/`AssetPath` strings have no leading `/` (`asset('foo.png')`), and `resolve()`/`asset()` take constrained literal types — cast through the `$app/types` union (`RouteId`, `Path`, `AssetPath`) when the value is dynamic. `Path` collapses to `never` when the project has no routes.

## Sync and check noise

`svelte-kit sync` may print validator warnings about overwritten tsconfig options against the generated `$app/tsconfig.json` — noise, not diagnostics; do not "fix" them by redefining `paths` in the project tsconfig. Real `svelte-check` errors out of a stray `build/` output folder are a genuine `tsconfig.json` exclude gap; see `references/cli.md` → `sv check`.
