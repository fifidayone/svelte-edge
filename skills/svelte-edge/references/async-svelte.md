# Async Svelte: Await Expressions, Boundaries, Abort-aware Patterns

Use this file for async-first component patterns in Svelte itself.
Do not use it as the canonical source for SvelteKit remote functions; those belong in `references/sveltekit.md`.

## Direct await expressions

Available in **Svelte 5.36+** with `compilerOptions.experimental.async` enabled.

This unlocks:
- top-level `await` in component `<script>`
- `await` in markup
- async `$derived(...)`

### Top-level await

```svelte
<script lang="ts">
	let user = await fetch('/api/me').then((r) => r.json());
</script>

<p>{user.name}</p>
```

### Await in markup

```svelte
<p>{(await getProfile()).displayName}</p>
```

### Async `$derived`

```svelte
<script lang="ts">
	let id = $state('42');
	let user = $derived(await getUser(id));
</script>
```

These patterns are edge features, not default assumptions for every codebase.
Only recommend them when the project has opted in or the user explicitly wants the newest async Svelte patterns.

## `<svelte:boundary>`

Use boundaries to handle rendering errors and initial async pending states inside a subtree.

```svelte
<svelte:boundary>
	{#snippet pending()}
		<p>Loading…</p>
	{/snippet}

	{#snippet failed(error, reset)}
		<p>{error.message}</p>
		<button onclick={reset}>Try again</button>
	{/snippet}

	<UserPanel id={userId} />
</svelte:boundary>
```

Important behavior:
- `pending` only covers the first async render of that boundary subtree
- later async updates should use `$effect.pending()` when you need update-time pending state — only meaningful when `compilerOptions.experimental.async` is enabled
- boundaries catch rendering/effect errors in their subtree
- `onerror={(error, reset) => ...}` is for reporting or handling boundary errors outside the `failed` snippet
- newer SvelteKit versions also support server-side error boundaries in framework rendering flows
- they do **not** catch errors from event handlers, timers, or unrelated async work outside render/effect flow

### Server rendering and `transformError`

By default, a boundary does not save server rendering from failing. In **Svelte 5.51+**, `render(...)`, `mount(...)`, and `hydrate(...)` can receive `transformError` to sanitize an error for a boundary `failed` snippet.
The transformed value must be serializable. Do not send raw server error messages or stacks to the browser.

Frameworks such as SvelteKit own the actual `render(...)` call, so do not assume app code can set `transformError` directly unless it controls rendering.

## `getAbortSignal()`

Use `getAbortSignal()` inside async derived/effect work so requests are canceled automatically when dependencies change or the component is destroyed.

```svelte
<script lang="ts">
	import { getAbortSignal } from 'svelte';

	let query = $state('svelte');
	let results = $derived(await search(query, getAbortSignal()));
</script>
```

This is one of the best modern patterns for race-safe async UI.

## `fork(...)`

Available in **Svelte 5.42+**.
Use `fork(...)` for speculative work that may be committed later, such as preloading or optimistic preparation.
Do not present it as a default primitive for ordinary app code.

## `hydratable(...)`

Use `hydratable(key, fn)` only for low-level SSR data reuse where async server work should not be repeated and block hydration on the client.
It powers higher-level tools such as SvelteKit remote functions; most app code should prefer the framework data API.

Returned data must be serializable by Svelte's transport. Use library-prefixed keys to avoid collisions.
With CSP, `hydratable` injects data into the rendered head; dynamic SSR needs a nonce and prerendered HTML needs hashes.

## Hard reminders

- Direct await expressions require `experimental.async`
- `<svelte:boundary>` pairs naturally with async-first UI
- Use `onerror` for boundary reporting, and `transformError` only where rendering is controlled
- `getAbortSignal()` is the right answer for cancelable async derived/effect work
- `fork(...)` is advanced and should be justified, not sprayed everywhere
- `hydratable(...)` is low-level; prefer SvelteKit data APIs in app code
- Remote functions belong in `references/sveltekit.md`
