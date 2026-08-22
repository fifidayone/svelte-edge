# SvelteKit 3 Preview

**Status:** prerelease — in release-candidate phase since 2026-08-13, with no further breaking changes planned before stable (official announcement). Verified against `@sveltejs/kit@3.0.0-next.0` through `next.25`. Use for a project resolving `3.0.0-next.*` — this is the SvelteKit 3 knowledge reference for writing and reviewing SvelteKit 3 code — or when verifying a SvelteKit 2→3 migration (the migration workflow itself lives in `references/migration.md`). Never apply this file to a SvelteKit 2 project.

## Contents

- [Entry gate](#1-entry-gate)
- [Writing SvelteKit 3 code](#2-writing-sveltekit-3-code)
- [Migration and verification](#3-migration-and-verification)
- [Non-mixing rules](#4-non-mixing-rules)
- [Coverage note](#coverage-note)

## 1. Entry gate

Required floors: Node 22.17+, TypeScript 6+, Svelte 5.56.4+, Vite `^8.0.12`+ (Rolldown 1.0), `@sveltejs/vite-plugin-svelte` 7+, matching adapter prerelease. On Windows, also require Vite **8.0.16+** — the `^8.0.12` peer floor allows 8.0.12–8.0.15, which are vulnerable to CVE-2026-53571 (`server.fs.deny` bypass via NTFS ADS / 8.3-name forms).

Move onto the `next` line with the `sv@next` CLI: scaffold new projects with `npx -y sv@next create` (SvelteKit 3 directly), and convert existing SvelteKit 2 projects with `npx -y sv@next migrate sveltekit-3 --tasks all`. Exact syntax and workflow live in `references/migration.md` and `references/cli.md`. Everything below assumes that jump already happened; this file doesn't own getting there.

Upgrade framework, adapter, and toolchain in one branch. After upgrading: `svelte-kit sync`, `npx sv check`, unit/component tests, E2E, production build, deployment smoke test. Treat every step as required, not optional — this is a prerelease, not a patch bump.

## 2. Writing SvelteKit 3 code

Use these as the default shape for new SvelteKit 3 code. Training data has no reliable SvelteKit 3 knowledge — rely on these sections instead of SvelteKit 2 habits; each entry states when to use it and what replaces it.

### Configuration

Since **SvelteKit 3.0.0**, configuration must be passed directly to the `sveltekit` Vite plugin — `svelte.config.js` is no longer supported at all (SvelteKit 2.62+ made this optional; SvelteKit 3 makes it mandatory).

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

- `paths.origin` replaces removed `prerender.origin` and `adapter-node`'s `ORIGIN` env var.
- `csrf.trustedOrigins`: explicit allowlist of full origins (protocol + host) permitted to submit cross-origin forms. `csrf.checkOrigin` is removed. CSRF checks run only in production. (Note: Remote functions strictly require same-origin and ignore `trustedOrigins` because remote endpoints are internal implementation details).
- `router.resolution`: setting to `'server'` is incompatible with `router.type: 'hash'` or `output.bundleStrategy: 'inline' | 'single'`.
- `csp.directives`: setting `'require-trusted-types-for': ['script']` requires `'svelte-trusted-html'` in `'trusted-types'` (and `'sveltekit-trusted-url'` if `serviceWorker.register` is true).
- A universal `config` export wins over a server-only `config` export on the same route.
- TypeScript config extends the generated `$app/tsconfig` (`.svelte-kit/tsconfig.json` is gone); the project owns its `include` array, and `compilerOptions.types` must include `$app/types` when set.

### Subpath imports (`#lib`)

The `$lib` alias and `kit.files.lib` configuration option are **removed**. Declare `#lib` in `package.json` using Node's built-in subpath imports:

```json
{
	"imports": {
		"#lib": "./src/lib/index.js",
		"#lib/*": "./src/lib/*"
	}
}
```

Import as `import Component from '#lib/Component.svelte'`. Both Vite and TypeScript resolve Node subpath imports natively. Since **next.20**, SvelteKit no longer writes its own `#lib` entry into the generated `paths` — the `package.json` `imports` map is the only mechanism, and its target paths must use explicit module extensions (the example above already does).

### Param matchers (`src/params.ts`)

Folder-based matchers (`src/params/*.ts`) are **removed**. Declare all param matchers in a single `src/params.ts` (or `src/params.js`) file using the `defineParams` helper:

```ts
// src/params.ts
	import { defineParams } from '@sveltejs/kit/params'; // next.19+ — moved from the root package
import * as v from 'valibot';

export const params = defineParams({
	// Standard Schema variant (e.g. Valibot, Zod)
	integer: v.pipe(v.string(), v.toNumber()),
	// Function variant
	fruit: (param) => (param === 'apple' || param === 'orange' ? param : undefined)
});
```

When a matcher returns a parsed value or uses a Standard Schema, route `params` are typed with the parsed output type.

### Environment variables

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

	// Optional during build step, required at runtime
	SECRET: { schema: building ? v.optional(v.string()) : v.string() }
});
```

Import from `$app/env/private` or `$app/env/public` (never both server/client-crossing); use `$app/env` in place of `$app/environment`. Do not import from `$env/*` in new SvelteKit 3 code — both exist only as deprecated compatibility aliases, removed entirely once SvelteKit 3 stabilizes. `defineEnvVars` lives in `@sveltejs/kit/env` (`next.10`+). Env-related *types* moved to `@sveltejs/kit/env` as well at **next.20** — do not import them from the root package. On the SvelteKit 3 line this model needs no experimental flag — explicit environment variables are no longer experimental (unlike the SvelteKit 2 opt-in).

### Navigation and state

```ts
import { goto } from '$app/navigation';

// open a modal without a full navigation, keep history entry
goto('/photos/42', { shallow: true, state: { showModal: true } });

// same, but restore state after a reload
goto('/photos/42', { shallow: true, state: { showModal: true }, persistState: true });

// replace history entry instead of pushing a new one
goto('/photos/42', { shallow: true, state: { showModal: true }, replace: true });
```

Use when: back-button-friendly UI state (modals, detail panels) without leaving the current route. Default preserves scroll/focus; pass `reset: true` only when the navigation should intentionally reset them. State is not restored after a page reload unless `persistState: true` is set. Avoid when: the target is a genuinely different page — use a normal `goto(url)` instead. This `goto()` shape ships in **3.0.0-next.13** and replaces `pushState`/`replaceState`/`noScroll`/`keepFocus`; those two option names collapse into the single `reset` option in the same release.

Since **next.19**, these types no longer live in the root `@sveltejs/kit` package: import `Page`, `ReadonlyURL`, and `ReadonlyURLSearchParams` from `$app/state`, and `BeforeNavigate`, `OnNavigate`, `AfterNavigate`, `Navigation`, `NavigationTarget`, `NavigationType`, `GotoOptions`, and the `Navigation*` variant types from `$app/navigation`.

- `page.url` and its search parameters are readonly — copy with `new URL(page.url.href)` before mutating.
- Navigation `delta` exists only for `popstate` navigations — guard it before use.
- `goto` rejects destinations that don't resolve to an internal route — use `window.location.href` for external navigation.
- Link attributes disable with `false`, not `"off"`: `data-sveltekit-*="false"`.

### Paths API

The deprecated `base`, `assets`, and `resolveRoute` exports are **removed** from `$app/paths`. Use:

```ts
import { resolve, asset } from '$app/paths';

// resolve route or pathname to relative/base-prefixed path
const postPath = resolve('/blog/[slug]', { slug: 'hello-world' });

// resolve static asset path
const logoUrl = asset('logo.png'); // NOTE: no leading slash
```

Type names `Pathname` and `Asset` are renamed to `Path` and `AssetPath`. **Path strings passed to `asset()` and typed in `Path`/`AssetPath` have no leading `/`** (e.g. `asset('foo.png')`, not `asset('/foo.png')`). `resolve()` and `asset()` take constrained literal types, not plain `string` — `resolve` accepts the `RouteIdWithSearchOrHash | PathnameWithSearchOrHash` union and `asset` accepts `AssetPath`, so a `string` variable or a template literal with `string` interpolation fails typecheck (a variable typed as the whole `RouteId`/`Path` union also fails — sveltejs/kit#15536). Cast to the matching `$app/types` type (`RouteId`, `Path`, or `AssetPath`).

### Reloading data

```ts
import { refreshAll, invalidate, preloadCode } from '$app/navigation';

await invalidate('app:posts');                          // rerun only load/queries depending on key
await refreshAll();                                      // rerun every active load function, query, and remote function
await refreshAll({ includeLoadFunctions: false });       // refresh remote functions only, skip load reruns

await preloadCode('/blog/[slug]');                       // takes Route ID (next.14), without paths.base
```

Use `invalidate(key)` when specific data changed. Use `refreshAll()` after a mutation that could affect anything on the page — it refreshes active remote functions and reruns `load` functions unless `includeLoadFunctions: false` is passed. `refreshAll()` ships in **3.0.0-next.8** and does **not** reset `page.state`, unlike deprecated `invalidateAll()`.

In **3.0.0-next.14**, re-navigating to the current URL re-runs all `load` functions and queries, and `preloadCode` accepts a Route ID (e.g. `/blog/[slug]`) rather than a pathname, without `paths.base` prefixing. `preloadData(...)` can resolve to `{ type: 'error', status, error }` — handle the error result instead of assuming every result is loaded data.

### Snapshots

Since **next.17**, the page-level `export const snapshot` is **deprecated** in favor of the `snapshot()` helper from `$app/navigation`:

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

`snapshot({ id?, capture, restore, reset? })` must run during component initialization and stays active while the component is mounted. You can register several snapshots per component — unique per call site, or pass an explicit `id` — and the optional `reset` callback runs on navigations that have no captured value to restore.

### Errors, redirects, and forms

```ts
import { error, redirect } from '@sveltejs/kit';

// NOTE: error() requires a string message as 2nd argument (next.13+)
error(404, 'Post not found', { code: 'POST_NOT_FOUND' });

// External redirects MUST specify { external: true }
throw redirect(307, 'https://checkout.stripe.com', { external: true });
```

- `error(status, message, details)` requires a string `message` as the second argument (`next.13`+).
- `handleError` can return `{ status, message }` to override the response status code, and `App.Error` always includes `status: number`.
- `redirect(...)` to an external URL throws unless `{ external: true }` or listed in `csrf.trustedOrigins`.
- Cross-origin form submissions without a `Content-Type` header are rejected.
- Form action `fail(...)` status codes surface directly as the HTTP response status.
- `invalid(...issues)` throws a `ValidationError` (accepts strings or `StandardSchemaV1.Issue` objects, wrapping strings as `{ message: issue }`).
- The `kit.experimental.handleRenderingErrors` flag is removed (dropped in **3.0.0-next.8**) — rendering-boundary error handling is unconditional.
- Enhanced cross-page form actions now navigate to the action page on success **and** failure, matching native form behavior (`next.17`+).
- Every error now runs through the `handleError` hook (`next.19`+).
- `ActionResult` and `SubmitFunction` moved from `@sveltejs/kit` to `$app/forms` (`next.19`+).
- Hooks-related types moved to `@sveltejs/kit/hooks` (`next.20`+); `RequestEvent` and `Cookies` moved from `@sveltejs/kit` to `$app/server` (`next.21`+).

### Other behavior changes

- `json(...)` and `text(...)` from `@sveltejs/kit` are deprecated — use `Response.json(...)` and `new Response(...)`.
- `+server.js` supports the `QUERY` HTTP method (**next.24**+).
- `getRequest()` and `setResponse()` from `@sveltejs/kit/node` are now synchronous — remove `await`.
- Directories ending in `/server/` are server-only everywhere, not only under `src/routes`.
- Cookie v2 renames its option types: `CookieSerializeOptions` → `SerializeOptions`, `CookieParseOptions` → `ParseOptions` (only for imports from the `cookie` package).
- Dev-server CORS for static assets is delegated to Vite — configure `server.cors.origin` in `vite.config` if you rely on cross-origin dev access.

### Service workers and manifest

```ts
// service-worker.ts
import { build, files, version } from '$app/service-worker';
import { assets, prerendered, routes } from '$app/manifest';
```

`$service-worker` is **removed**. Use `$app/service-worker` for build/version/file lists inside the worker, and `$app/manifest` for build-manifest details. `$app/paths` is importable inside service workers (`next.12`+). Service workers register as modules (`type: 'module'`) — replace `importScripts(...)` with module imports. A TypeScript service worker gets its own project: `src/service-worker/tsconfig.json` extending `$app/tsconfig/service-worker`, excluded from the root tsconfig (move a flat `src/service-worker.ts` to `src/service-worker/index.ts`). Import `self` from `$app/service-worker` for a typed worker context (`ServiceWorkerGlobalScope` — accurate `fetch` event types).

### Remote functions

Gate on `experimental.remoteFunctions`; the official migration guide documents `.remote.ts`/`.remote.js` files without the flag as an error on this line — but do not treat the build as your only guard: on the SvelteKit 2 line the equivalent guard proved unreliable in practice (clean builds, silent no-op calls, server code reaching the client bundle), and this preview line has not been independently re-verified to hard-fail. Check the flag directly whenever remote files exist. Every remote form field must be created with `form.fields.foo.as(...)` (`next.10`+, #16331). Inside a query, `event.url`/`event.params`/`event.route` are inaccessible (`next.10`+, #16452). In **next.14**, the deprecated `.run()` method on live queries is **removed** entirely — use `await` or async iteration. `handleValidationError` is removed at **next.19** — remote validation errors reach `handleError` with `kind: 'validation'`. Remote function types moved to `$app/server` at **next.20** and again to `@sveltejs/kit/remote` (with `isValidationError`) at **next.21** — on `next.21`+, import them from `@sveltejs/kit/remote`.

## 3. Migration and verification

The migration is automated on the `sv@next` line: `npx -y sv@next migrate sveltekit-3 --tasks all` (workflow and cleanup in `references/migration.md`). The CLI rewrites the affected code and writes `MIGRATION_TASKS.md` for the steps it cannot automate — work through that file, using the official migration guide (https://next.svelte.dev/docs/kit/migrating-to-sveltekit-3) as the reference for each change.

### Test checklist

- `svelte-kit sync` + `npx sv check` clean
- Every route touched by the config-precedence flip re-verified
- Shallow-routing flows: back button, reload, focus/scroll behavior
- Form actions: status codes on failure, cross-origin submissions, redirects
- Service worker: install/update cycle, cache contents via `$app/manifest` if used
- Deployment-update UX with the new 1-hour default poll and response-triggered checks
- Adapter-specific smoke test (below)

### Adapter verification

Upgrade the adapter together with SvelteKit, not independently — check its exact prerelease peer range. Run a production build and a disposable deploy on the target platform; test SSR, prerendering, form actions, redirects, cookies, server-only imports, and static assets on that deploy, not just locally. Adapter *authors*: adapter Vite plugins are split into separate `pre` and `post` plugins since **next.18** — publish and consume the `next`-line adapter release that matches SvelteKit, and re-verify the plugin surface.

- `adapter-cloudflare`: `platform.context` is renamed to `platform.ctx`.
- `adapter-vercel`: the `edge` runtime is no longer supported — pick a supported runtime.
- Custom adapters: `builder.createEntries` is removed from the adapter API — check the `next`-line adapter prerelease.
- Since **next.25**, an `applyReroute` helper exists for adapters supporting split serverless function deployments (#16665).

## 4. Non-mixing rules

Never generate SvelteKit 2-only APIs in SvelteKit 3 code: `invalidateAll`, `pushState`, `replaceState`, `$service-worker`, `$lib`, `kit.csrf.checkOrigin`, `kit.prerender.origin`, `adapter-node` `ORIGIN`, `noScroll`, `keepFocus`, `data-sveltekit-*-off`, `alias` config option, `$env/*`, `$app/environment`, `@sveltejs/kit/node/polyfills`, `preloadStrategy`, `handleRenderingErrors`, `base`/`assets`/`resolveRoute` from `$app/paths`, page-level `export const snapshot` (deprecated at `next.17`), `handleValidationError` (removed at `next.19`), or root-package imports of `Page`, `RequestEvent`, `Cookies`, the `Navigation*` types, and `ActionResult`/`SubmitFunction`.

Never generate SvelteKit 3-only APIs in SvelteKit 2 code: `refreshAll`, `goto({ shallow, persistState, reset })`, `$app/manifest`, `$app/service-worker`, `#lib`, `paths.origin`, `$app/env/*`, `Path`/`AssetPath` (no leading slash), `resolve()`, `asset()`, `defineParams`, `snapshot()` from `$app/navigation`, the `@sveltejs/kit/params`/`@sveltejs/kit/hooks`/`@sveltejs/kit/env`/`@sveltejs/kit/remote` subpaths, `$app/forms`/`$app/state`/`$app/server` type imports, or universal-over-server `config` precedence.

## Coverage note

Checked against every `3.0.0-next.*` entry from `next.0` through `next.25`. Version-pinned markers: `refreshAll()` deprecates `invalidateAll()` at **next.8** (#16289), `defineEnvVars` moves to `@sveltejs/kit/env` at **next.10** (#16375), remote form fields require `.as(...)` (#16331), and `event.url`/`event.params`/`event.route` become inaccessible inside query bodies (#16452), `goto()` shallow-routing/`reset` consolidates at **next.13** (#16558), Node 22.17+, `preloadCode(routeId)`, `.run()` deletion, current-URL link refresh, and `RouteId` narrowing ship at **next.14** (#16572, #16573, #16576, #16580, #16597), and the Svelte peer floor moves to **5.56.4+** and cookies require ASCII-only names with a default `path: '/'` (cookie v2) as of **next.15**. `next.16` is a single internal fix (route-resolution module generation) with no consumer-facing change. From **next.17**: enhanced cross-page form actions navigate to the action page on success and failure (#16684), and the `snapshot()` helper lands in `$app/navigation` while page-level `export const snapshot` is deprecated (#16685, #16687). **next.18** splits adapter Vite plugins into `pre` and `post` (#16711). **next.19** moves `defineParams` to `@sveltejs/kit/params` (#16716), runs all errors through `handleError` (#16664), relocates `Page`/`ReadonlyURL`/`ReadonlyURLSearchParams` to `$app/state`, the navigation types to `$app/navigation`, and `ActionResult`/`SubmitFunction` to `$app/forms` (#16694), removes `handleValidationError` in favor of `handleError` with `kind: 'validation'` (#16672), and stops treating `+`-prefixed files containing test/spec/stories as routes (#16715). **next.20** moves remote function types to `$app/server` (#16740), removes SvelteKit's own `#lib` entry from generated `paths` so the `imports` map must use explicit module extensions (#16736), and moves hooks types to `@sveltejs/kit/hooks` (#16737) and env types to `@sveltejs/kit/env` (#16739). **next.21** moves remote function types again — to `@sveltejs/kit/remote`, with `isValidationError` (#16764) — superseding the `next.20` location, and moves `RequestEvent`/`Cookies` to `$app/server` (#16751). `next.22` is a single patch (respect Vite default log level, #16767) with no consumer-facing API change. `next.23` is a single patch (only print the prerender progress newline when necessary, #16766) with no consumer-facing API change. **next.24** adds the `QUERY` HTTP method for `+server.js` (#16782) and passes the project-relative source `filename` to the font `preload` filter (#16443), plus a linear-time parse for large streamed frames (#16489) — harness-verified (build + `svelte-check` clean on next.24). **next.25** adds an `applyReroute` helper for adapters that support split serverless deployments (#16665) alongside response-logging and navigation-completion patches — changelog-verified, not yet harness-verified.

`next.15`'s own changelog reads as a full changeset consolidation — dozens of "breaking" bullets that were already shipped incrementally across `next.0`–`next.14` (Node 22.17, `$app/stores` removal, `#lib`, etc.) reappear there verbatim. Don't treat every bullet in a single `next.X` changelog entry as new to that release; cross-check against this file's existing coverage before adding a version-pinned marker for something that's actually already covered above.

This file is audited on the maintenance cadence and can lag behind upstream even within a range it claims to cover — spot-check the live `sveltejs/kit` source or changelog for a question not covered above. Behavior facts in `Writing SvelteKit 3 code` were verified against the official migration guide (https://next.svelte.dev/docs/kit/migrating-to-sveltekit-3) and the `sv migrate sveltekit-3` task source at `sv@1.0.0-next.1`.

`svelte-kit sync` — the first half of `npm run check` — may print `Found issues while validating tsconfig.json` against the generated `$app/tsconfig.json`. On `next.20`+, the old `"paths" was overwritten. Imports from "#lib" may not typecheck` warning is gone — SvelteKit stopped writing its own `#lib` entry into the generated `paths`, so `#lib` resolution happens entirely through the `package.json` `imports` map. Remaining sync warnings are validator noise about overwritten options, not `svelte-check` diagnostics — do not "fix" them by redefining `paths` in the project tsconfig. That is a different issue from `svelte-check` reporting real errors out of a stray `build/` output folder — that one *is* a genuine `tsconfig.json` exclude gap, not benign noise; see `references/cli.md` → `sv check`.
