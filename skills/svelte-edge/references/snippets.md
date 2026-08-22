# Snippets and Modern Component Composition

Use snippets for new component composition instead of legacy slots.

## Contents

- [Basic and named snippets](#basic-snippets)
- [Passing and typing snippets](#passing-snippets-to-components)
- [Exported and programmatic snippets](#exported-snippets)
- [When not to use snippets](#when-not-to-use-snippets)

## Basic snippets

```svelte
<script lang="ts">
	let { children } = $props();
</script>

<div class="panel">
	{@render children?.()}
</div>
```

## Named snippets

```svelte
<script lang="ts">
	let { header, children, footer } = $props();
</script>

<div class="card">
	<header>{@render header?.()}</header>
	<section>{@render children?.()}</section>
	<footer>{@render footer?.()}</footer>
</div>
```

## Inline snippet declaration

```svelte
{#snippet row(user: { id: string; name: string })}
	<li>{user.name}</li>
{/snippet}

<ul>
	{@render row({ id: '1', name: 'Ada' })}
</ul>
```

## Passing snippets to components

```svelte
<!-- App.svelte -->
<script lang="ts">
	let users = $state([{ id: '1', name: 'Ada' }]);
</script>

<List items={users} row={row} />

{#snippet row(user)}
	<li>{user.name}</li>
{/snippet}
```

```svelte
<!-- List.svelte -->
<script lang="ts" generics="T extends { id?: string | number }">
	import type { Snippet } from 'svelte';

	interface Props {
		items: T[];
		row: Snippet<[T]>;
	}

	let { items, row }: Props = $props();
</script>

<ul>
	{#each items as item (item.id ?? item)}
		{@render row(item)}
	{/each}
</ul>
```

A `{#snippet}` declared as a direct child of a component becomes a **named snippet prop** on that component, not implicit `children` — this fails type-checking if the component has no matching prop (e.g. one written inside an `#each` block that wraps a component). Declare the snippet in the parent scope, outside the component tag, then pass it explicitly (`icon={icon}`).

## Implicit children

When a component needs one main content area, use `children` as the implicit snippet prop.
This is the modern replacement for the default slot mental model.

## Typing snippets

Type snippet props with `Snippet` from `'svelte'`, not as plain functions.
Use `Snippet<[T]>` for parameterized snippets — the tuple lists the argument types.
Use `<script lang="ts" generics="T">` to link a snippet parameter to another prop like `items`.

```ts
import type { Snippet } from 'svelte';

// zero-arg snippet
let { children }: { children?: Snippet } = $props();

// typed parameterized snippet — requires generics="T" on the script tag
let { items, row }: { items: T[]; row: Snippet<[T]> } = $props();
```

## Exported snippets

Exported snippets are available in modern Svelte 5 and are useful when you want reusable render fragments outside a single component file.
Use them when they improve reuse, not by default for every component.

## Programmatic snippets

Use `createRawSnippet` only for advanced library-style cases where markup must be generated from code. It requires **Svelte 5.5+**.
For ordinary app composition, write normal `{#snippet}` blocks; they are clearer and type better.

`createRawSnippet` requires getters for params and returns `{ render, setup? }`.
The `render` output is HTML, so treat it with the same care as any generated markup.

## When not to use snippets

Do not treat snippets as a replacement for all state sharing or all dynamic component logic.
In runes mode, component values are already dynamic. Render a capitalized value directly and do not generate `<svelte:component>` for new code:

```svelte
<script lang="ts">
	import type { Component } from 'svelte';

	let { selected: Selected }: { selected: Component<{ label: string }> } = $props();
</script>

<Selected label="Current" />
```

Use `Component` for component-value types. `SvelteComponent`, `ComponentType`, and `ComponentEvents` are deprecated class/event-era types. Preserve `<svelte:component>` only in coherent legacy-mode code or while migrating it.

## Hard reminders

- For new composition patterns, use snippets rather than slots
- Prefer `children` for a single primary content region
- Use optional snippet props with guards before rendering
- Keep snippet names semantic and small
- Type snippet inputs in TypeScript
- Reach for `createRawSnippet` only for advanced programmatic rendering
- Do not mix legacy slot APIs into new components
- Dynamic component in runes mode -> render `<Selected />`, not `<svelte:component>`
- A snippet declared as a direct child of a component becomes a named prop on it, not implicit `children`
