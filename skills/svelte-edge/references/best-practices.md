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
3. For a new component, target current Svelte 5 syntax supported by the project.
4. For a tiny fix, preserve the file's coherent generation.
5. For an explicit migration, convert the whole component and its composition/event contracts together.

Do not leak familiar legacy syntax into a new answer merely because it still compiles. In new Svelte 5.56+ components, avoid `export let`, `$:`, `on:`, `<slot>`, `{@const}`, `createEventDispatcher`, `beforeUpdate` / `afterUpdate`, `<svelte:component>`, class-component types, and deprecated `spring`/`tweened` stores.

Enforcement differs by syntax. Mixing `export let` with runes (`legacy_export_invalid`), writing `$:` reactive statements (`legacy_reactive_statement_invalid`), and using event-attribute modifiers like `onclick|preventDefault` are hard compile errors. Legacy `on:` directives and `<slot>` in a runes component still compile, emitting the deprecation warnings `event_directive_deprecated` and `slot_element_deprecated`. Because the build does not fail on those warnings, a CI gate that must block legacy syntax should treat `svelte-check` warnings as errors (`svelte-check --fail-on-warnings`).

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

- Don't set `<svelte:options immutable={true}>` or `accessors={true}` on new components — both are Svelte 4 legacy options, deprecated overall in Svelte 5. In runes mode they have no behavioral effect, but the compiler does emit deprecation warnings (`options_deprecated_immutable`, `options_deprecated_accessors`) — a zero-warning build will still flag them. They only still do anything in legacy (non-runes) components.

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

- `<select multiple>` `value` must be an array, `null`, or `undefined` — any other value logs the runtime warning `select_multiple_invalid_value` in development and the current selection is kept as-is; it does not throw.

## Dynamic elements

Use `<svelte:element this={tag}>` only when the tag is genuinely dynamic.
If `this` is nullish, nothing renders. If `this` is a void element such as `br`, `hr`, or `img`, do not render children inside it; the children are ignored and Svelte logs the `dynamic_void_element_content` runtime warning in development — it does not throw. (A literal void tag with children is a hard compile error: `void_element_invalid_content`.)

## Browser-only and hydration-sensitive patterns

Be careful with browser-only values in SSR contexts.
This matters especially for:
- `MediaQuery`
- direct `window` / `document` access
- measurements that do not exist during SSR

If CSS can solve the problem, prefer CSS.
If component logic truly depends on browser state, explain the SSR/hydration caveat.
- To change an `<img>` `src` or `{@html}` block on hydration, stash the value, unset it during SSR, and restore it in `$effect()` after mount. This is the documented fix for `hydration_attribute_changed` / `hydration_html_changed`, not a workaround.

## Form ActionData Nullability (SvelteKit)

SvelteKit page `form` props (from Server Actions) are typed as `ActionData | null`. SvelteKit passes `null` on the initial GET request.
When passing the `form` prop to a child component that expects optional data (`ActionData | undefined`), convert `null` to `undefined` explicitly:

```svelte
<!-- +page.svelte -->
<script lang="ts">
	let { form } = $props();
</script>

<ChildComponent form={form ?? undefined} />
```

## Prerender Hash Links Rule

In shared layouts or global headers/footers, never use relative hash links (e.g. `href="#section"`).
During Static Site Generation (Prerendering), the crawler evaluates links relative to the currently crawled page path. If it crawls a subpage (e.g., `/projects/01`), it attempts to fetch `/projects/01#section`. If `/projects/01` does not have an element with `id="section"`, the build fails.

**Always use Root-relative links in shared layouts:**
```svelte
<!-- ✅ Correct -->
<a href="/#section">Section</a>

<!-- ❌ Fails prerendering on subroutes -->
<a href="#section">Section</a>
```

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
- On Svelte 5.16+, prefer `class={{...}}` object/array forms over `class:` directives in new components
- Prefer `$derived` over `$effect` for computed state
- Do not use event modifiers on modern event attributes
- Never render unsanitized user-controlled content with `{@html}`
- Keep keyed each blocks by default
- Type wrapper components with `svelte/elements`
- Do not pass children to void tags rendered through `<svelte:element>`
- Treat hydration-sensitive browser logic with care
- Explicitly map SvelteKit `form` props (`form ?? undefined`) when passing to optional component props
- Always use root-relative links (e.g. `href="/#section"`) in shared layouts to prevent prerender crawler crashes
