# Remote Functions (Opt-in)

This file covers SvelteKit remote functions — an opt-in feature requiring `kit.experimental.remoteFunctions: true`.
Do not apply patterns here to projects that have not opted in.
For stable SvelteKit architecture (`load`, form actions, `+server`), see `references/sveltekit.md`.

## What remote functions are

Remote functions are an experimental SvelteKit feature available since **2.27+**.
They are not the stable default.

Use them only when:
- the user explicitly wants remote function capabilities
- the project has opted in (`kit.experimental.remoteFunctions: true`)
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

## `$app/server` imports

Use `$app/server` for remote functions and server helpers:
- `query`, `form`, `command`, `prerender`, `requested`, `getRequestEvent`
- `read(asset)` for reading imported assets from the filesystem on the server

Keep `$app/server` imports out of client-only modules except where remote function files require them.

## Current query semantics (2.61+)

Remote queries are Promise-like and can be awaited in any context: templates, component initialization, event handlers, module scope, universal `load`, and async callbacks. Identical calls share the same active cache, so an event-handler read dedupes with a rendered consumer.

```svelte
<script lang="ts">
	import { getData } from './data.remote';
</script>

<p>{await getData()}</p>
<button onclick={async () => console.log(await getData())}>Inspect</button>
```

The short-lived `.run()` API was removed in **SvelteKit 2.61**. Never generate `query.run()` for a current project.

## Live queries

`query.live` is available from **2.59+** for real-time `AsyncIterable` data. From **2.61+**, live-query instances are themselves async-iterable.

- SSR consumes the first yielded value, then closes that iterator
- active client consumers share one connection
- instances expose `connected` and `reconnect()`
- live queries have no `refresh()` because the stream updates itself
- exclude streaming responses with `Cache-Control: no-store` from service-worker caching
- reconnect intentionally after mutations when server state or cookies require a fresh stream

Use live queries only for genuinely long-lived data, not as a fashionable replacement for ordinary reads.

## Remote forms and commands: current landmarks

- **2.50+**: action buttons use `myForm.fields.action.as(...)` style; do not use older `buttonProps` examples.
- **2.56+**: use `field.as(type, value)` for pre-populated form fields instead of manual boilerplate.
- **2.57+**: form `submit` helper returns a `boolean` validity flag; use the return value instead of tracking separately.
- **2.60+**: `submit` and `hidden` fields accept numbers/booleans; unread validation issues warn in development.
- **2.61+**: a form instance has programmatic `submit()`; `enhance` receives a form-instance copy. It no longer receives the old `{ form, data, submit }` object.
- **2.64+**: `command` accepts `File` values directly.
- **2.66+**: boolean checkbox fields should be optional/defaulted because unchecked inputs are absent from `FormData`.
- **2.68+**: `RemoteFormEnhanceInstance` and `RemoteFormEnhanceCallback` are exported; submit-field values remain in the action payload.
- **2.69+**: remote forms expose `submitted`.

For file fields, set `enctype="multipart/form-data"`. Use SvelteKit **2.69.1+** when remote forms manipulate file inputs — that patch fixes prototype pollution in file-input deletion.

## Remote function gotchas

These are correctness landmines documented in the official docs:

- **Query caching**: cache keys come from serialized arguments. Object/map/set members are sorted for stable keys; use an array when order must affect identity. Do not manually sort object keys.
- **Batch queries**: `query.batch` combines calls into one server request. `requested(...)` supports batch queries from **2.59+**.
- **`getRequestEvent()` inside remote functions**: headers cannot be set (except cookies inside `form` and `command` kinds), and `route` / `params` / `url` reflect the *calling page*, not the remote endpoint. Critical for auth checks — do not assume `event.route.id` points at the remote function file.
- **Sensitive form fields use the `_` prefix convention**: fields named with a leading underscore (e.g. `_password`) are not echoed back to the client on validation reload. Use this for any field that should not survive a round trip.
- **Unchecked booleans are absent**: make checkbox booleans optional/defaulted in the validation schema. SvelteKit 2.66+ warns about this common error.
- **Remote validation failures**: invalid remote function arguments usually mean stale clients or hostile input. Use `handleValidationError` when you need a controlled generic response; avoid `unchecked` unless you accept the security tradeoff.
- **Redirects**: `redirect(...)` works in `query`, `form`, and `prerender`, but not in `command`.

## Single-flight mutations

Remote `form` and `command` handlers can refresh or set query data in the same server round trip:

- Server-driven refresh: call `getPosts().refresh()` or `getPost(id).set(value)` in the handler; the framework awaits forwarded refreshes.
- In **2.65+**, a remote query can refresh or set another query when related cached data must update.
- Live query reconnect: call `getNotifications(userId).reconnect()` when a mutation should restart an active `query.live`.
- Client-requested refresh: client calls `.updates(...)`; server must explicitly accept with `requested(queryFn, limit)`.
- From **2.58+**, `requested(...)` yields `{ arg, query }`; `limit` is required because the client controls the requested list and unbounded work is a denial-of-service risk.
- Use `requested(queryFn, limit).refreshAll()` or live-query `reconnectAll()` shorthand when all accepted instances should update.

Hydratable transport means remote results can carry richer types than JSON; do not add custom JSON serialization without a concrete interoperability requirement.
