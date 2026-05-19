# Snippets and Modern Component Composition

Use snippets for new component composition instead of legacy slots.

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
<List items={users} row={row} />

{#snippet row(user)}
	<li>{user.name}</li>
{/snippet}
```

```svelte
<!-- List.svelte -->
<script lang="ts" generics="T">
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
If the user needs dynamic component selection, consider normal component composition or `<svelte:component>`-style migration work only when required.

## Hard reminders

- For new composition patterns, use snippets rather than slots
- Prefer `children` for a single primary content region
- Use optional snippet props with guards before rendering
- Keep snippet names semantic and small
- Type snippet inputs in TypeScript
- Reach for `createRawSnippet` only for advanced programmatic rendering
- Do not mix legacy slot APIs into new components
