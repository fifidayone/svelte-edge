# Migration: Legacy Svelte to Modern Svelte 5

Use this file only for migration work.
Do not use it as the default source for brand-new code.

## Contents

- [Migration doctrine and replacements](#migration-doctrine)
- [Component syntax migrations](#event-modifiers-during-migration)
- [SvelteKit migration](#sveltekit-migration-note)
- [Remote-function compatibility](#remote-function-compatibility-timeline)

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
| `beforeUpdate` / `afterUpdate` | `$effect.pre(...)` / `$effect(...)` |
| `<slot>` | snippets + `{@render}` |
| `{@const value = ...}` | `{const value = ...}` on Svelte 5.56+ |
| `use:action` | `{@attach ...}` or `fromAction(...)` |
| `setContext` / `getContext` | `createContext()` when available |
| `createEventDispatcher` | typed callback props |
| `on:click` | `onclick` |
| `class:active` | object/array `class={...}` |
| `spring(...)` / `tweened(...)` stores | `Spring` / `Tween` classes |
| `<svelte:component this={Thing}>` | dynamic `<Thing />` in runes mode |
| `SvelteComponent` / `ComponentType` | `Component` |
| `ComponentEvents` | callback-prop types |
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

## `{@const}` to declaration tags

On **Svelte 5.56+**, official docs classify `{@const}` as legacy. In a component being migrated to runes mode, replace it with `{const ...}` and use `$derived` when the declared value must remain reactive:

```svelte
{const total = $derived(price * quantity)}
```

Declaration tags can live anywhere in markup and use lexical scope; they are not restricted to the immediate children accepted by `{@const}`. Do not change one tag in an otherwise untouched legacy component merely to claim a migration.

## Context migration

If the codebase is on **Svelte 5.40+**, move shared typed context helpers into a normal module with `createContext()`.
Do not put those helpers in a `.svelte` wrapper file.

## SvelteKit migration note

Do not treat remote functions as the automatic end state of every SvelteKit migration.
Stable `load`, form actions, and `+server` files are still the normal baseline.

For Svelte 5 projects on SvelteKit 2.12+, replace deprecated `$app/stores` imports with `$app/state` and update store-subscription syntax to direct reactive property reads.

### Config location and Kit 3 readiness

SvelteKit **2.62+** can place all Svelte/Kit configuration in `sveltekit({...})` inside `vite.config`, and `sv@0.16+` scaffolds new projects that way. If migrating:

- move the entire config coherently
- flatten former `kit` properties into the `sveltekit({...})` argument
- remove the old config after verifying tooling support
- never leave partial settings in both places, because plugin configuration causes `svelte.config.js` to be ignored

The old file remains supported in SvelteKit 2. Move it early only for a deliberate cleanup or Kit 3 preparation.

### Environment variables and Kit 3 readiness

SvelteKit **2.63+** offers experimental explicit environment variables. `$env/*` remains the stable Kit 2 target; do not bulk-migrate it unless the project explicitly opts into `experimental.explicitEnvironmentVariables` or is preparing for Kit 3.

An opted-in migration includes:

- declare variables in `src/env.ts` with `defineEnvVars`
- replace `$env/static|dynamic/private|public` imports with `$app/env/private|public`
- replace `$app/environment` with `$app/env`
- preserve public/private boundaries and validate transformed values

## Remote-function compatibility timeline

Remote functions are experimental and changed rapidly. Do not stop at an intermediate migration state:

- **2.56** briefly added query `.run()`, stable cache-key sorting, richer hydratable transport, and explicit server acceptance for client-requested refreshes.
- **2.57** made enhanced form `submit()` return a boolean validity result.
- **2.58** made `requested(query, limit)` require the limit and yield `{ arg, query }`.
- **2.59** added `query.live` and batch-query support in `requested(...)`.
- **2.61** removed `.run()` and made `await query()` valid in every context; it also changed `enhance` callbacks to receive a form-instance copy.

### Current migration target (2.61+)

When upgrading an older remote-functions project to a current release:

- Remove any manual sorting of cache keys — caching now sorts object keys internally.
- Replace every transitional `someQuery(...).run()` with `await someQuery(...)`.
- Add the required `limit` parameter to all `requested(...)` calls and update destructuring to `{ arg, query }`.
- Audit client-triggered refresh logic — it now requires explicit server-side permission.
- Drop cross-query failure handling that assumed cascading failures; each refresh failure is isolated per-query.
- Update old `enhance(({ form, data, submit }) => ...)` callbacks to receive the form instance and call `await form.submit()`.
- For live queries, handle shared connections, `connected`, `reconnect()`, and async iteration instead of treating them as ordinary refreshable queries.

Do not teach or preserve `.run()` as a compatibility fallback. Pinning to SvelteKit 2.56–2.60 is the only reason it should appear in historical code.
