# Best Practices, Pitfalls, Testing, and Modern Rules

Use this file for guardrails and practical judgment.

## Contents

- [Syntax-generation choice](#choose-the-syntax-generation-before-writing)
- [Anti-mixing and effect discipline](#the-anti-mixing-rule)
- [Events, collections, and classes](#event-modifiers-in-modern-svelte-5)
- [Typed wrappers and dynamic elements](#typed-element-wrappers)
- [Hydration, testing, and HTML safety](#browser-only-and-hydration-sensitive-patterns)

## Choose the syntax generation before writing

1. Inspect `svelte`, `@sveltejs/kit`, and relevant tooling versions.
2. Inspect the touched component: runes mode, legacy mode, or already mixed.
3. For a new component, target current stable Svelte 5 syntax supported by the project.
4. For a tiny fix, preserve the file's coherent generation.
5. For an explicit migration, convert the whole component and its composition/event contracts together.

Do not leak familiar legacy syntax into a new answer merely because it still compiles. In new Svelte 5.56+ components, avoid `export let`, `$:`, `on:`, `<slot>`, `{@const}`, `createEventDispatcher`, `beforeUpdate` / `afterUpdate`, `<svelte:component>`, class-component types, and deprecated `spring`/`tweened` stores.

## The anti-mixing rule

For new components, do not mix legacy and modern Svelte generations.

Bad:
```svelte
<script>
	export let title;
	let count = $state(0);
	$: doubled = count * 2;
</script>
```

Good:
```svelte
<script lang="ts">
	let { title }: { title: string } = $props();
	let count = $state(0);
	let doubled = $derived(count * 2);
</script>
```

Also keep template syntax current: use `{const ...}` / `{let ...}` on Svelte 5.56+ instead of legacy `{@const}`. Read `references/declaration-tags.md` before writing declaration tags.

## `$effect` discipline

Treat `$effect` as an escape hatch.
Use it for:
- imperative DOM work
- subscriptions
- browser APIs
- syncing with systems outside Svelte

Do **not** use `$effect` to keep another piece of state in sync if `$derived` can express it.

Bad:
```svelte
<script lang="ts">
	let price = $state(10);
	let qty = $state(2);
	let total = $state(0);

	$effect(() => {
		total = price * qty;
	});
</script>
```

Good:
```svelte
<script lang="ts">
	let price = $state(10);
	let qty = $state(2);
	let total = $derived(price * qty);
</script>
```

## Event modifiers in modern Svelte 5

Do not write `onclick|preventDefault={...}` or other event modifiers on modern event attributes.
That syntax does not apply to event attributes.

Use normal DOM methods inside the handler:

```svelte
<script lang="ts">
	function submit(event: SubmitEvent) {
		event.preventDefault();
		// ...
	}
</script>

<form onsubmit={submit}></form>
```

During migration, helper wrappers from `svelte/legacy` can be a temporary bridge, but they are not the target style for new code.

## Keyed each blocks

Default to keyed each blocks whenever rendering collections that can change order or identity.

```svelte
{#each items as item (item.id)}
	<Row {item} />
{/each}
```

## Class objects and arrays

For **Svelte 5.16+**, prefer `class={...}` object/array forms instead of `class:` directives in new code.

```svelte
<div class={['card', { active, disabled }]}></div>
```

## CSS custom properties

Prefer CSS custom properties for highly themeable visual values instead of exploding prop lists.

## Images

For local assets and build-time image optimization in SvelteKit, `@sveltejs/enhanced-img` is often a strong choice.
Do not pretend it is the only valid image strategy.
Use normal `<img>` / `<picture>` patterns when runtime or remote-source behavior makes more sense.

## Typed element wrappers

When a component wraps a native element, type forwarded props with `svelte/elements`.
Use specific attribute interfaces when they exist, and `SvelteHTMLElements[...]` otherwise.

```svelte
<script lang="ts">
	import type { Snippet } from 'svelte';
	import type { HTMLButtonAttributes, SvelteHTMLElements } from 'svelte/elements';

	type ButtonProps = HTMLButtonAttributes & { children?: Snippet };
	type DivProps = SvelteHTMLElements['div'] & { children?: Snippet };
</script>
```

Forward rest props to the actual element. This preserves attributes, events, actions/attachments, and accessibility attributes.

## Dynamic elements

Use `<svelte:element this={tag}>` only when the tag is genuinely dynamic.
If `this` is nullish, nothing renders. If `this` is a void element such as `br`, `hr`, or `img`, do not render children inside it; Svelte throws in development.

## Browser-only and hydration-sensitive patterns

Be careful with browser-only values in SSR contexts.
This matters especially for:
- `MediaQuery`
- direct `window` / `document` access
- measurements that do not exist during SSR

If CSS can solve the problem, prefer CSS.
If component logic truly depends on browser state, explain the SSR/hydration caveat.

## Testing reminder

Detailed testing guidance now lives in `references/testing.md`.

Keep these rules in mind even when you are not reading that file first:
- prefer behavior-focused tests over implementation-detail tests
- for SvelteKit apps, default to Vitest for unit/component coverage and Playwright for end-to-end flows
- for async-first UI, test pending, success, failure, and retry behavior

## Raw HTML safety

Treat `{@html}` as a security-sensitive escape hatch.
Never render unsanitized user-controlled content with `{@html}`.

If the content is not fully trusted, sanitize it first or choose a different rendering strategy.

## Dev-only tooling

`$inspect` is for debugging and development workflow.
Do not make production behavior depend on it.

## Hard reminders

- Never mix generations in a new component
- Select syntax from the installed version before generating code
- On Svelte 5.56+, use declaration tags rather than legacy `{@const}`
- Prefer `$derived` over `$effect` for computed state
- Do not use event modifiers on modern event attributes
- Never render unsanitized user-controlled content with `{@html}`
- Keep keyed each blocks by default
- Type wrapper components with `svelte/elements`
- Do not pass children to void tags rendered through `<svelte:element>`
- Treat hydration-sensitive browser logic with care
