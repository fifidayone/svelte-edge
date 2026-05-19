# Migration: Legacy Svelte to Modern Svelte 5

Use this file only for migration work.
Do not use it as the default source for brand-new code.

## Public migration command

```bash
npx sv migrate svelte-5
```

## Migration doctrine

When migrating, rewrite coherently.
Do not half-migrate a component into a mixed-generation hybrid.

### Good migration shapes
- migrate a whole component to runes mode
- keep a legacy file legacy if only a tiny fix is required
- modernize in coherent chunks, not one random rune at a time

## Common replacements

| Legacy | Modern target |
|---|---|
| `export let` | `$props()` |
| `$:` computed state | `$derived(...)` |
| `$:` side effects | `$effect(...)` |
| `<slot>` | snippets + `{@render}` |
| `use:action` | `{@attach ...}` or `fromAction(...)` |
| `setContext` / `getContext` | `createContext()` when available |
| `on:click` | `onclick` |
| `class:active` | object/array `class={...}` |
| `new Component({ target, props })` | `mount(Component, { target, props })` |
| `component.$destroy()` | `unmount(component)` |

## Event modifiers during migration

Modern event attributes do not support event modifiers.
When migrating from `on:submit|preventDefault`, move the behavior into the handler.

```svelte
<script lang="ts">
	function handleSubmit(event: SubmitEvent) {
		event.preventDefault();
		// ...
	}
</script>

<form onsubmit={handleSubmit}></form>
```

## Props migration

Bad half-migration:
```svelte
<script>
	export let title;
	let count = $state(0);
</script>
```

Good coherent migration:
```svelte
<script lang="ts">
	let { title }: { title: string } = $props();
	let count = $state(0);
</script>
```

## Slots to snippets

```svelte
<!-- legacy -->
<slot />
```

```svelte
<!-- modern -->
<script>
	let { children } = $props();
</script>

{@render children?.()}
```

## Actions to attachments

If the codebase is ready for modern DOM behavior and is on **Svelte 5.29+**, move toward attachments.
If you need a bridge from a legacy action, use `fromAction(...)`.

## Context migration

If the codebase is on **Svelte 5.40+**, move shared typed context helpers into a normal module with `createContext()`.
Do not put those helpers in a `.svelte` wrapper file.

## Imperative component API

In Svelte 5, components are not classes. For imperative roots, widgets, tests, and non-Svelte integration points, use the function APIs from `svelte`:

```ts
import { mount, unmount } from 'svelte';
import App from './App.svelte';

const app = mount(App, {
	target: document.querySelector('#app')!,
	props: { name: 'Ada' }
});

await unmount(app, { outro: true });
```

- Use `hydrate(Component, { target, props })` when reusing server-rendered HTML.
- Use `unmount(app, { outro: true })` when outro transitions should play before removal.
- `mount` / `hydrate` do not run effects synchronously; use `flushSync()` only when tests or imperative integration need pending effects/DOM updates immediately.
- Avoid legacy class helpers such as `asClassComponent` unless temporarily bridging old code.

## SvelteKit migration note

Do not treat remote functions as the automatic end state of every SvelteKit migration.
Stable `load`, form actions, and `+server` files are still the normal baseline.

## Migrating remote functions to SvelteKit 2.56

If the project already uses remote functions and is upgrading to **SvelteKit 2.56+**, address these breaking changes:

- Remove any manual sorting of cache keys — caching now sorts object keys internally.
- Replace `await someQuery(...)` calls that happen outside the render phase with `someQuery(...).run()`.
- Add the required `limit` parameter to all `requested(...)` calls and update destructuring to `{ arg, query }`.
- Audit client-triggered refresh logic — it now requires explicit server-side permission.
- Drop cross-query failure handling that assumed cascading failures; each refresh failure is isolated per-query.

In **SvelteKit 2.57+**, also update form submit handlers that previously discarded the return value — `submit` now yields a `boolean` validity flag.
