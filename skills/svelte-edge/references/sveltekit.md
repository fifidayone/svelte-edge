# SvelteKit Defaults, State Safety, Routing, Forms, and Edge Features

Use this file for SvelteKit architecture and behavior.

## Contents

- [Stable architecture and server safety](#stable-default-architecture)
- [Types, loading, and state](#types-and-props)
- [Forms and server routes](#form-actions)
- [Project configuration and environment variables](#project-configuration)
- [Navigation and UI state](#navigation-apis)
- [Errors and redirects](#error-and-redirect-helpers)
- [Remote functions](#remote-functions)
- [Server helpers](#getrequestevent)

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

Use generated route types.

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

### Auth and parent data

Do not assume a layout `load` protects every child route.
Layout loads do not rerun on every client navigation, and layout/page loads run concurrently unless a child explicitly awaits `parent()`.

For protected data, prefer:
- hooks for broad route families before any load runs
- route-specific `+page.server.ts` guards for page-only protection
- explicit `await parent()` only when the child truly depends on parent data

Never put authorization-critical logic in a layout load while child loads can run protected work independently.

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

## `+server` routes

Use `+server.ts` / `+server.js` for server-only handlers, APIs, webhooks, and custom HTTP behavior.

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

- New project on Kit 2.62+ -> prefer the current `sv` scaffold layout in `vite.config`
- Existing project -> preserve its coherent config location unless migration is requested
- Never split options across both files
- Kit 3 plans to require Vite-plugin configuration, but do not call `svelte.config.js` removed or invalid in Kit 2

## Server-only modules and environment variables

Use `$lib/server` for server-only code that must never enter the client bundle.

### Stable SvelteKit 2 default

Use `$env/static/private` / `$env/static/public` for build-time values and `$env/dynamic/private` / `$env/dynamic/public` for runtime values. Only use public env modules for values safe to expose to the browser; public means client-readable.

### Explicit environment variables (experimental)

SvelteKit **2.63+** offers an opt-in preview of the SvelteKit 3 model:

- enable `experimental.explicitEnvironmentVariables`
- declare variables in `src/env.ts` with `defineEnvVars` from `@sveltejs/kit/hooks`
- import declared values from `$app/env/private` or `$app/env/public`
- use `$app/env` instead of `$app/environment`
- configure validation/transforms, `public`, `static`, and descriptions per variable

This is an edge-mode recommendation only. `$env/*` and `$app/environment` remain the stable SvelteKit 2 default, but official docs say they will be removed in SvelteKit 3. Explain that migration horizon without pretending the experimental replacement is already mandatory.

Never import private env or `$lib/server` modules from client/shared code.

## Build and prerender additions

- With **SvelteKit 2.66+**, adapter precompression also covers prerendered `.md` and `.mdx` files.
- With **SvelteKit 2.67+**, use `prerender.handleInvalidUrl` to fail, warn, ignore, or custom-handle URLs the crawler cannot parse. Do not misuse `handleHttpError` for this case.

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

Remote functions are an **experimental SvelteKit feature** available since **2.27+**.
They are not the stable default.

Use them only when:
- the user explicitly wants edge capabilities
- the project has opted in
- the solution benefits from async-first component usage and type-safe client/server calls

Requirements:
- `compilerOptions.experimental.async: true`
- `kit.experimental.remoteFunctions: true`
- remote functions live in `.remote.ts` / `.remote.js` files

Remote functions have four flavours:
- `query`, including the `query.batch` and `query.live` variants
- `form`
- `command`
- `prerender`

Do not invent other names.
Do not put remote functions in `+page.server.ts` and pretend that is the same thing.

For remote form functions in newer SvelteKit, do not use older `buttonProps` examples. In **SvelteKit 2.50+**, action buttons use the newer `myForm.fields.action.as(...)` style instead.

For pre-populated form fields in **SvelteKit 2.56+**, use `field.as(type, value)` to set defaults instead of manual boilerplate.

In **SvelteKit 2.57+**, the form `submit` helper returns a `boolean` indicating submission validity for enhanced form remote functions. Use the return value instead of tracking validity separately.

### Current query semantics (2.61+)

Remote queries are Promise-like and can be awaited in any context: templates, component initialization, event handlers, module scope, universal `load`, and async callbacks. Identical calls share the same active cache, so an event-handler read dedupes with a rendered consumer.

```svelte
<script lang="ts">
	import { getData } from './data.remote';
</script>

<p>{await getData()}</p>
<button onclick={async () => console.log(await getData())}>Inspect</button>
```

The short-lived `.run()` API was removed in **SvelteKit 2.61**. Never generate `query.run()` for a current project.

### Live queries

`query.live` is available from **2.59+** for real-time `AsyncIterable` data. From **2.61+**, live-query instances are themselves async-iterable.

- SSR consumes the first yielded value, then closes that iterator
- active client consumers share one connection
- instances expose `connected` and `reconnect()`
- live queries have no `refresh()` because the stream updates itself
- exclude streaming responses with `Cache-Control: no-store` from service-worker caching
- reconnect intentionally after mutations when server state or cookies require a fresh stream

Use live queries only for genuinely long-lived data, not as a fashionable replacement for ordinary reads.

### Remote forms and commands: current landmarks

- **2.60+**: `submit` and `hidden` fields accept numbers/booleans; unread validation issues warn in development.
- **2.61+**: a form instance has programmatic `submit()`; `enhance` receives a form-instance copy. It no longer receives the old `{ form, data, submit }` object.
- **2.64+**: `command` accepts `File` values directly.
- **2.66+**: boolean checkbox fields should be optional/defaulted because unchecked inputs are absent from `FormData`.
- **2.68+**: `RemoteFormEnhanceInstance` and `RemoteFormEnhanceCallback` are exported; submit-field values remain in the action payload.
- **2.69+**: remote forms expose `submitted`.

For file fields, set `enctype="multipart/form-data"`. Use SvelteKit **2.69.1+** when remote forms manipulate file inputs because that patch fixes prototype pollution in file-input deletion.

### Remote function gotchas

These are correctness landmines documented in the official docs:

- **Query caching**: cache keys come from serialized arguments. Object/map/set members are sorted for stable keys; use an array when order must affect identity. Do not manually sort object keys.
- **Batch queries**: `query.batch` combines calls into one server request. `requested(...)` supports batch queries from **2.59+**.
- **`getRequestEvent()` inside remote functions** has restrictions: headers cannot be set (except cookies inside `form` and `command` kinds), and `route` / `params` / `url` reflect the *calling page*, not the remote endpoint. Critical for auth checks — do not assume `event.route.id` points at the remote function file.
- **Sensitive form fields use the `_` prefix convention**: fields named with a leading underscore (e.g. `_password`) are not echoed back to the client on validation reload. Use this for any field that should not survive a round trip.
- **Unchecked booleans are absent**: make checkbox booleans optional/defaulted in the validation schema. SvelteKit 2.66+ warns about this common error.
- **Remote validation failures**: invalid remote function arguments usually mean stale clients or hostile input. Use `handleValidationError` when you need a controlled generic response; avoid `unchecked` unless you accept the security tradeoff.
- **Redirects**: `redirect(...)` works in `query`, `form`, and `prerender`, but not in `command`.

### Single-flight mutations

Remote `form` and `command` handlers can refresh or set query data in the same server round trip:

- Server-driven refresh: call `getPosts().refresh()` or `getPost(id).set(value)` in the handler; the framework awaits forwarded refreshes.
- In **2.65+**, a remote query can refresh or set another query when related cached data must update.
- Live query reconnect: call `getNotifications(userId).reconnect()` when a mutation should restart an active `query.live`.
- Client-requested refresh: client calls `.updates(...)`; server must explicitly accept with `requested(queryFn, limit)`.
- From **2.58+**, `requested(...)` yields `{ arg, query }`; `limit` is required because the client controls the requested list and unbounded work is a denial-of-service risk.
- Use `requested(queryFn, limit).refreshAll()` or live-query `reconnectAll()` shorthand when all accepted instances should update.

Hydratable transport means remote results can carry richer types than JSON; do not add custom JSON serialization without a concrete interoperability requirement.

## `getRequestEvent()`

Use `getRequestEvent()` in modern SvelteKit server-side contexts when it meaningfully simplifies access to the active request event.
Remember that it is request-scoped server logic, not a general client-side primitive.

## `$app/server`

Use `$app/server` for remote functions and server helpers:
- `query`, `form`, `command`, `prerender`, `requested`, `getRequestEvent`
- `read(asset)` for reading imported assets from the filesystem on the server

Keep `$app/server` imports out of client-only modules except where remote function files require them.

## Link options

Remember that SvelteKit supports link options like preloading behavior on navigational links.
Use them when route transitions benefit from prefetching, but do not spam them everywhere by default.

## Hard reminders

- Stable first: `load`, actions, `+server`, `$app/state`
- New Svelte 5 project -> `$app/state`, never deprecated `$app/stores`
- Avoid shared mutable server state
- Keep `load` pure and side-effect-free
- Use shallow routing and snapshots when UI state/history behavior matters
- Recommend remote functions only in edge mode or opted-in projects
- Current remote query call outside render -> `await query()`, never `.run()`
- Keep stable `$env/*` defaults for Kit 2; present explicit env vars only as an opt-in Kit 3 migration preview
- Keep all configuration in one place; Vite-plugin config wins and causes `svelte.config.js` to be ignored
- For remote mutations, account for query refresh/reconnect explicitly
