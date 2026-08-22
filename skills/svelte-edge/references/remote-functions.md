# Remote Functions (Opt-in)

This file covers SvelteKit remote functions — an opt-in feature requiring `kit.experimental.remoteFunctions: true`.
Do not apply patterns here to projects that have not opted in.
For SvelteKit architecture (`load`, form actions, `+server`), see `references/sveltekit.md`.
This file covers the **SvelteKit 2** experimental flag only. On the SvelteKit 3 preview line (`3.0.0-next.*`), remote functions have additional breaking changes (mandatory `form.fields.foo.as(...)`, restricted `event.url`/`event.params`/`event.route` access inside queries) — read `references/sveltekit-3-preview.md` in addition to this file once a project is on that line.

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
- remote functions are declared in `.remote.ts` / `.remote.js` files

Do not rely on the build to catch a missing flag. The intended guard errors when `.remote.ts` files are used without `kit.experimental.remoteFunctions`, but harness testing on 2.70.x showed it is not dependable: a file that merely exists builds fine, and client imports can silently no-op or leak server-side code into the client bundle instead of failing loudly. The reliable hard error is `experimental_async`, raised by `compilerOptions.experimental.async` when client code `await`s a remote call. Verify both flags are configured whenever remote files are present.

Remote functions have four flavours:
- `query`, including the `query.batch` and `query.live` variants
- `form`
- `command`
- `prerender`

Do not invent other names.
Do not put remote functions in `+page.server.ts` and pretend that is the same thing.

## Version gates

All remote-function version gates live here. Do not look for them in `SKILL.md`.

- remote functions opt-in: **SvelteKit 2.27+** with `kit.experimental.remoteFunctions: true`
- `query.batch`: **SvelteKit 2.38+**
- remote form action buttons via `myForm.fields.action.as(...)`: **SvelteKit 2.50+**
- `field.as(type, value)` for pre-populated form fields: **SvelteKit 2.56+**
- hydratable remote function transport: **SvelteKit 2.56+**
- form `submit` returning a boolean validity flag: **SvelteKit 2.57+**
- `requested(query, limit)` yielding `{ arg, query }`: **SvelteKit 2.58+**
- `query.live` and `requested(...)` support for `query.batch`: **SvelteKit 2.59+**
- remote queries awaitable in any context, `.run()` removed, live queries async-iterable, form-instance `submit()` / new `enhance` callback shape: **SvelteKit 2.61+**
- remote commands accepting `File`: **SvelteKit 2.64+**
- remote queries refreshing or setting other queries: **SvelteKit 2.65+**
- exported `RemoteFormEnhanceInstance` / `RemoteFormEnhanceCallback` and retained submit-field values: **SvelteKit 2.68+**
- remote-form `submitted`: **SvelteKit 2.69+**

These gates apply to the SvelteKit 2 experimental flag only. The SvelteKit 3 preview line has its own separate version floors for remote functions — see `references/sveltekit-3-preview.md`. Do not merge a `next.*` requirement into this list or vice versa.

**Security patch floors:**
- remote forms that manipulate file inputs: require **SvelteKit 2.69.1+**
- remote-function origin checks: require **SvelteKit 2.70.0+** in builds where `NODE_ENV` isn't the literal `production` — earlier builds under those conditions compiled the check out (same root cause as the built-in form-action origin check — see the security patch floors in `SKILL.md`). The same patch also fixes prerendered remote functions being served live instead of from prerendered output under the same condition.

## `$app/server` in remote files

In `.remote.ts` / `.remote.js` files, import `query`, `form`, `command`, `prerender`, and `requested` from `$app/server` to declare remote functions.

Remote files can live anywhere under `src` except `src/lib/server`.

`getRequestEvent()` is a general server helper, not a remote-only API. This file documents its remote-function restrictions only; see `references/sveltekit.md` for ordinary server-side usage.

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

On the SvelteKit 3 preview line (**3.0.0-next.10+**, #16452), `event.url` / `event.params` / `event.route` become inaccessible from inside a `query` body entirely (not just pointing at the calling page) — this is stricter than the 2.x behavior below. Check `references/sveltekit-3-preview.md` before writing query bodies that read request context on that line.

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

On the SvelteKit 3 preview line (**3.0.0-next.10+**, #16331), every remote form field must be created with `myForm.fields.foo.as(...)` — a hand-built field object that was tolerated on 2.x fails validation there. Treat `.as(...)` as mandatory, not stylistic, once a project is on `3.0.0-next.10` or later; see `references/sveltekit-3-preview.md`.

## Remote function gotchas

These are correctness landmines documented in the official docs:

- **Query caching**: cache keys come from serialized arguments. Object/map/set members are sorted for stable keys; use an array when order must affect identity. Do not manually sort object keys.
- **Batch queries**: `query.batch` combines calls into one server request. `requested(...)` supports batch queries from **2.59+**.
- **`getRequestEvent()` inside remote functions**: headers cannot be set (except cookies inside `form` and `command` kinds), and `route` / `params` / `url` reflect the *calling page*, not the remote endpoint. Critical for auth checks — do not assume `event.route.id` points at the remote function file. On the SvelteKit 3 preview line, these three properties are inaccessible inside a `query` body rather than merely reflecting the calling page — see the note above.
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

On the SvelteKit 3 preview line, page-level `refreshAll()` (from `$app/navigation`) also refreshes active remote functions by default; pass `{ includeLoadFunctions: false }` to refresh remote functions only. This is distinct from the query-level `.refresh()` / `requested(...).refreshAll()` shorthands documented above, which exist on both lines. See `references/sveltekit-3-preview.md`.

## Hard reminders

- Never generate `query.run()` — removed in SvelteKit 2.61
- Never apply remote function patterns to a project without `kit.experimental.remoteFunctions: true` — a green build is not proof the flag is set (see the failure mode above)
- `getRequestEvent()` inside remote functions: `route`/`params`/`url` reflect the **calling page**, not the remote file (on SvelteKit 2); inaccessible entirely inside `query` bodies on the SvelteKit 3 preview line
- `enhance` callback shape changed in 2.61 — never use the old `{ form, data, submit }` shape
- `redirect(...)` works in `query`, `form`, `prerender` — **not** in `command`
- Sensitive fields: use `_` prefix to prevent them echoing back to the client
- Checkbox booleans: make them optional/defaulted — unchecked fields are absent from `FormData`
- For file inputs in remote forms: require SvelteKit **2.69.1+** (security patch)
- `requested(...)` must use a bounded `limit`; never allow unbounded client-requested refresh work
- Project on `3.0.0-next.*` -> also read `references/sveltekit-3-preview.md`; do not assume every gate/API above carries over unchanged
